## AWS EC2에서 nginx를 통해 서비스 띄우기 (spring - react)

### nginx란?
> 웹 서버 소프트웨어를 말하며 nginx와 함께 많이 사용되는 것이 Apache이다.  
> nginx는 트래픽이 많은 웹 사이트의 확장성을 위해 설계한 비동기 이벤트 기반 구조의 웹 서버 소프트웨어이다.


### 1. AWS EC2 인스턴스 준비
AWS EC2 인스턴스를 생성한 이후 ssh 혹은 웹페이지에서 인스턴스에 접속한다.

---

### 2. Git을 통해 프로젝트 파일 준비
AWS EC2 인스턴스에 접속이 완료된 후에는, Git을 통해 서버에 올리고자하는 리포지토리를 clone 한다.

인스턴스에 프로젝트 파일이 없었던 경우에는 밑 명령어를 통해 리포지토리를 복사할 수 있고,
```
❯ git clone 리포지토리 주소
```

인스턴스에 프로젝트 파일이 있었던 경우에는 밑 명령어를 통해 최신 상태로 업데이트할 수 있다.
```
❯ git pull origin 브랜치 이름
```

---

### 3. Spring - React에 필요한 소프트웨어 설치
Spring은 java의 프레임워크이기 때문에 java를 설치해야한다.  
최근에 진행했던 프로젝트의 경우에는 *amazon corretto 17* 를 사용했기 때문에 해당 버전을 기준으로 작성한다.

aws의 공식 document에 명령어가 친절하게 적혀있으며, 명령어를 복사하여 그대로 적용하면 대부분은 별 문제 없이 설치가 된다.  

공식 웹사이트: https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/generic-linux-install.html

amazon corretto 17 설치 명령어
```
❯ wget -O- https://apt.corretto.aws/corretto.key | sudo apt-key add -
sudo add-apt-repository 'deb https://apt.corretto.aws stable main'
```

```
❯  sudo apt-get update; sudo apt-get install -y java-17-amazon-corretto-jdk
```

이후 ***java -version*** 명령어를 작성했을 때, java의 버전이 설치한 버전이 나오면 설치가 완료된 것이다.

---

### 4. nginx 설치하기
AWS EC2에는 기본적으로 Nginx가 설치되어 있는 것 같다. (nginx 설치 명령어를 치면 이미 설치되어있다고 나옴)  

밑 명령어를 통해 nginx를 설치할 수 있다.
```
❯  sudo apt-get install nginx
```

---

### 5. nginx 설정

#### 1) nginx의 conf 파일 수정하기
ec2 인스턴스에서 사용하는 에디터는 **vi 에디터**이다. 
> vi 에디터 간단한 사용법
> 1. 내용을 수정하고자 할 때 - i
> 2. 위치 이동 - 방향키
> 3. 수정 중단 - esc
> 4. 저장하고 나가기 - :wq
> 5. 저장하지 않고 나가기 - :q

nginx.conf 파일을 열어보면 다양한 설정들이 기본적으로 작성되어 있는 것을 알 수 있다.  

이 파일에 설정을 직접 설정할 수도 있지만,  
1️⃣ etx/nginx 경로에 sites-enabled 디렉토리를 생성하고  
2️⃣ 안에 실제 서비스에 사용한 설정을 해당 디렉토리 내의 파일에 작성 후  
3️⃣ nginx.conf가 해당 파일을 참조  
하는 방법을 선호하는 것 같으므로 이 방법을 사용하려고 한다.

```
❯  sudo vi /etc/nginx/nginx.conf
```
위 명령어를 통해 nginx.conf 파일을 연 다음,  
```
include /etc/nginx/sites-enabled/*;
```
문구가 있는 곳을 찾은 후, 해당 문구는 주석 처리(#)를 해준다.  

```
include /etc/nginx/sites-enabled/*.conf; 
```
그리고 다음 줄에 위에 적혀있는 문구를 추가한다.  
만약 추가하지 않는다면 우리가 사용하고자하는 설정을 적용하지 못할 것이다.
```
        ...
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        # include /etc/nginx/sites-enabled/*;
        include /etc/nginx/sites-enabled/*.conf;
        ...
```

<br>

#### 2) sites-available, sites-enabled 설정

sites-available 디렉토리에 필요한 파일을 작성한 후, symbolic link를 통해 sites-enabled에 추가하는 방식을 많이 사용한다.
> **심볼링 링크 symbolic link란?**  
> 
> 윈도우의 바로가기 기능과 비슷한 개념이며, link를 연결하면 원본 파일을 직접 사용하는 것과 동일한 효과를 낸다.  
> 심볼릭 링크를 이용하면 삭제/수정/등록이 모두 공유된다.
> 
> symbolic link 생성  
> ❯ - ln -s [대상 원본 파일] [링크 파일]
> 
> symbolic link 제거  
> ❯ - rm [링크 파일]


```
❯ sudo mkdir /etc/nginx/sites-enabled
❯ sudo vi /etc/nginx/sites-available/이름1.conf
```
이름1에는 본인이 사용하고 싶은 이름을 사용하면 된다.

sites-enabled에 심볼링 링크를 추가할 것이기 때문에 우선 mkdir를 통해 디렉토리를 생성한다.  
vi 에디터를 통해 sites-available에 설정 파일인 conf 파일 생성 후, 밑의 내용을 작성한다.
```
server {
    listen 80;

    location /api {
        proxy_pass http://EC2 인스턴스 ip 주소:8080;
    }

    location / {
        root   /home/ubuntu/프로젝트 경로/build; # react(FE)의 빌드 경로
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
}
```
내용을 살펴보면 현재 HTTPS를 사용하지 않고 HTTP를 이용한다.  
AWS EC2 보안그룹에서 HTTP 포트(80번)을 열어주었으므로, 80 포트를 listen하고 있다고 선언하였다.

백엔드(Spring)의 경우에는 8080 포트를 통해 요청을 받기 때문에 프록시 설정을 해주어야 한다.  
/api 는 prefix로 /api가 붙으면 "http://EC2 인스턴스 ip 주소:8080;"로 프록시 처리를 하여 서버에 요청이 닿도록 한다.


root의 위치에는 프론트엔드(React)의 빌드 파일이 위치하는 경로를 인스턴스 절대 경로 기준으로 적으면 된다.

<br>

마지막으로 심볼릭 링크를 설정해주면 되는데, 밑 명령어를 통해 심볼링 링크를 지정하면 된다.
```
❯ sudo ln -s /etc/nginx/sites-available/이름1.conf /etc/nginx/sites-enabled/이름1.conf
```

---

### 6. nginx 테스트 및 재시작

#### 1) nginx 테스트
밑의 명령어를 통해 nginx 테스트를 할 수 있다. 
```
❯ sudo nginx -t
```
테스트 결과 밑과 같이 'ok', 'successful'과 같은 문구가 뜨면 설정에 사용한 문법에 문제가 없다는 뜻이다.

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

<br>

#### 2) nginx 재시작 및 권한 설정
설정이 완료되었으므로 nginx를 재시작해야 한다.  
밑 명령어를 통해 nginx를 재시작한다.
```
❯ sudo service nginx restart
```

<br>

만일 **500 Internal Server Error**가 발생했다면 프론트엔드(React)의 build 파일에 접근하는 과정에서 권한이 없어서 발생할 가능성이 있다.  
따라서 홈 디렉토리인 /home/ubuntu의 권한을 711로 설정하면 된다.

```
❯ chmod 711 /home/ubuntu
```

