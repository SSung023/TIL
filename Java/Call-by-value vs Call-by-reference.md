## Call-by-value vs Call-by-reference

### Parameter vs Argument 
> Parameter와 Argument는 비슷한 늬앙스를 띄고 있지만, 실제로는 다른 의미를 가지고 있다.  
> 하지만 다른 의미를 가지고 있기 때문에 구분해서 사용하는 것이 좋다.

#### Parameter
Parameter는 메서드 정의에서 나열되는 변수명을 말한다.
``` java
public void modify(int num1, int num2){ ... }
```
위에서 "num1"과 "num2"가 Parameter에 해당한다. **"매개변수"** 라고 말하며 함수/메서드의 **"변수명"** 을 이야기 한다.

<br>

#### Argument
Argument는 메서드/함수를 호출할 때 실제로 전달되는 값을 말한다.
``` java
...
modify(10, 23);
...
```
위에서 10, 23이 Argument에 해당한다. **"전달인자, 인자"** 로 이야한다.
***
<br>

### Call-by-value? Call-by-reference?
> 메서드에서 Parameter를 전달할 때 두 가지 방법이 있는데, 이것이 Call-by-value와 Call-by-reference이다.

#### Call-by-value
먼저 Call-by-value는 Parameter를 전달할 때 값을 복사하여 전달해주기 때문에 Pass-by-value라고도 한다.  
메서드를 호출하는 부분(Caller)과 호출되는 부분(Callee)에서의 값은 서로 다른 값이다.  
``` java
public void test(){
    int num1 = 10;
    int num2 = 20;
    
    swap(num1, num2);
    
    System.out.println(num1); // 10 출력
    System.out.println(num2); // 20 출력
}

// num1에는 10의 값이, num2에는 20의 값이 복사되어 전달
void swap(int num1, int num2){
    // 밑과 같은 과정을 진행하더라도 메서드 내에서만 반영이 된다
    int tmp = num1;
    num1 = num2;
    num2 = tmp;
}
```
위 코드를 예시로 들어보면,   
test 메서드 내에서 swap을 호출할 때에 인자로 전달할 때 num1, num2의 값을 복사해서 swap() 메서드에 전달하기 때문에  
swap 메서드 내의 num1, num2는 test 메서드 내의 num1, num2와는 다른 값이다. 

<br>

#### Call-by-reference
Call-by-reference는 Parameter를 전달할 때, 변수의 "주소(참조) 값"을 전달하기 때문에 Pass-by-reference라고도 한다.  
참조를 전달하기 때문에 Caller과 Callee의 파라미터의 값은 서로 동일하다.  
따라서 전달받은 인자의 값을 수정하면 실제 값도 변경된다.

``` java
void test(){
    int num1 = 10;
    int num2 = 20;
    
    swap(num1, num2);
    
    System.out.println(num1); // 20 출력
    System.out.println(num2); // 10 출력
}

// num1과 num2의 주소값이 전달
void swap(int &num1, int &num2){
    int tmp = num1;
    num1 = num2;
    num2 = tmp;
}
```
위는 C++ 코드 예시인데, C++에서는 & 연산자를 통해 참조를 전달할 수 있다. (Java는 Call-by-reference를 사용하지 않는다)  
따라서 위 코드에서는 test()에서의 num1, num2가 swap()에서의 num1, num2와 같다.

***

#### 다음 글에서는 Java가 Call-by-reference를 사용하지 않고, Call-by-value만을 사용하는 방법에 대해 알아보자!


