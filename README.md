# c-plus-plus-final
c-plus-plus-final

## 함수로 객체 전달하기

객체의 주소를 매개변수로 하여 함수로 전달하면
```c++
void upgrade(Pizza *p) 
{
   p->setRadius(20);
}

int main()
{
   Pizza obj(10);
   upgrade(&obj);
```
원본 객체의 내용을 조작할 수 있다.   
이것은 장점이 되기도 하고 단점이 되기도 한다.   

## 복사 생성자가 호출되는 경우

```c++

복사 생성자가 호출되는 경우는 크게 3가지이다.

같은 종류의 객체로 초기화하는 경우
MyClass obj(obj2); // 여기서 복사생성자가 호출된다.


객체를 함수에 전달하는 경우
MyClass func(MyClass obj) { // 여기서 복사생성자가 호출된다.

}

함수가 객체를 반환하는 경우
MyClass func(MyClass obj) {
   MyClass tmp;
   
   return tmp; // 이때 복사생성자가 호출된다.
}

복사 생성자가 필요한 경우

class MyArray {
public:
   int size;
   int* data; // data 클래스 멤버 변수를 동적할당 하기위해 int형 포인터로 선언해주었다.
   
   MyArray(int size)
   {
      this->size = size;
      data = new int[size]; 
   }
   
   ~MyArray()
   {
      if(data != NULL) delete[] this->data; // data는 동적배열이기 때문에 delete[] 형으로 소멸
   }
};

int main()
{
   MyArray buffer(10);
   buffer.data[0] = 1; 원본 클래스 멤버 배열의 0번째 자리에 1을 저장하였다.
   {
      MyArray clone = buffer; // 기본 복사 생성자가 호출된다. 원래 있던 buffer객체의 내용을 clone 객체를 생성하면서 복사
   }
   buffer.data[0] = 2; // buffer는 위에 { } 구문을 빠져나오면서 객체가 소멸한다. 따라서 런타임 오류가 뜬다.
   
   return 0;
}

위의 코드에서 문제점은 MyArray clone = buffer; 구문에서 clone 객체는 블록을 빠져나오면서 소멸하게 된다.
그러면 buffer랑 clone이랑 같은 동적 메모리를 사용하고 있었을 텐데 clone이 소멸되면서 buffer는 메모리를 쓸 수가 없다.
다시 말해서 buffer랑 clone이랑 같은 집에 살고 있었는데 clone이 집을 부동산에 내놓아버렸고
buffer는 내놓은 집에 들어가서 살려고 buffer.data[0] = 2;를 하는 것과 같다.

이렇게 동일한 공간을 복사해서 사용할 때 발생하는 문제가 얕은 복사(shallow copy)라고 한다.

이 문제를 해결하기 위해서 깊은 복사(deep copy)로 기본 복사 생성자를 사용하지 않고 개발자가 직접 복사 생성자를 구현해주는 방법이 있다.

다음의 코드를 살펴보자

```c++
MyClass func(MyClass obj) {
   MyClass tmp;
   
   return tmp; // 이때 복사생성자가 호출된다.
}

복사 생성자가 필요한 경우

class MyArray {
public:
   int size;
   int* data; // data 클래스 멤버 변수를 동적할당 하기위해 int형 포인터로 선언해주었다.
   
   MyArray(int size)
   {
      this->size = size;
      data = new int[size]; 
   }
   
   MyArray(const& MyArray& other)
   {
      this->size = other.size; 
      this->data = new int[other.size]; // 새로운 공간 할당
      for(int i=0; i< size; i++) {
         this->data[i] = other.data[i]; // 데이터 복사
      }
   }
    
   ~MyArray()
   {
      if(data != nullptr) delete[] this->data;
      data = nullptr;
   }
};

int main()
{
   MyArray buffer(10);
   buffer.data[0] = 1; 원본 클래스 멤버 배열의 0번째 자리에 1을 저장하였다.
   {
      MyArray clone = buffer; // 기본 복사 생성자가 호출된다. 원래 있던 buffer객체의 내용을 clone 객체를 생성하면서 복사
   }
   buffer.data[0] = 2; // buffer는 위에 { } 구문을 빠져나오면서 객체가 소멸한다. 따라서 런타임 오류가 뜬다.
   
   return 0;
}

기본 복사 생성자와의 다른 부분 코드만 알아보자.
개발자가 생성한 복사 생성자의 형태는 다음과 같다.
MyArray(const& MyArray& other)
   {
      this->size = other.size; 
      this->data = new int[other.size]; // 새로운 공간 할당
      for(int i=0; i< size; i++){ 
         this->data[i] = other.data[i]; // 데이터 복사
      }
   }
새로운 메모리 공간을 할당해주는 모습을 볼 수 있다. 
이렇게 되면 서로 각각 다른 메모리를 사용하기 때문에 오류는 발생하지 않는다.

소멸할 때도
   ~MyArray()
   {
      if(data != nullptr) delete[] this->data;
      data = nullptr;
   }
nullptr로 멤버 data를 직접 초기화해줘야 한다.

객체에 대하여 대입 연산자 = 을 사용할 수 있을까? 답은 가능하다.이다. 같은 타입의 객체끼리는 대입 연산이 가능하다.
하지만 객체끼리 대입연산자나 인덱스 연산자, 포인터 연산자는 개발자가 연산자 중복을 사용하여 연산자를 마음대로 정의해서 사용할 수 있다.
  
```

## 정적 변수

```c++
클래스의 모든 객체가 하나의 변수를 공유하게 하려면 어떻게 해야 할까
정적 변수를 사용하는 방법이 있다. 일종의 객체 전역 변수의 개념으로 사용을 한다.
클래스 멤버변수 private 밑에 public으로 static 키워드를 붙여서 선언을 해준다.
초기화는 전역 변수이기 때문에 클래스 밖에서 초기화를 해준다.
사용 예는 다음과 같다.

class Circle {
   int x, y;
   int radius;
public:
   static int count; // 정적 변수 선언
   Circle() : x(0), y(0), radius(0) {
      count++;
   }
   Circle(int x, int y, int r) : x(x), y(y), radius(r) {
      count++;
   }
   Circle() {
      count--;
   }
};

int Circle::count = 0; // 반드시 밖에서 초기화

int main()
{
   Circle c1;
   cout << "지금까지 생성된 원의 개수 = " << Circle::count << endl; // 사용할 때도 Circle::count 로 사용
   
   Circle c2(100, 100, 30);
   cout << "지금까지 생성된 원의 개수 = " << Circle::count << endl;
   
   return 0;
}
정리하자면 정적 변수 static 변수는 클래스 밖에서 선언해주어야 하고 객체가 없으므로 접근할 때 Circle::count의 형태로 접근해야 한다.

정적 멤버 함수

멤버 함수도 정적 멤버 함수로 만들 수 있다.

정적 멤버 함수는 정적 멤버 변수와 지역 변수만을 사용할 수 있고 즉 멤버변수는 사용 불가능 
객체가 없기 때문에 접근할 때 역시 Circle::getCount()의 형태, this가 가리키는 객체가 없으므로 this 포인터를 사용할 수 없다.

정적 상수

정적으로 대부분은 상수가 많이 선언된다. 상수는 클래스의 모든 객체가 공유하기 마련이기 때문이다.
정적으로 상수를 선언하게 되면 객체 통틀어 하나만 존재하기 때문에 메모리를 절약할 수 있게 된다.
정적 상수는 선언과 동시에 초기화 가능하다.

정적 멤버 변수 빼고 함수하고 상수는 클래스 내부에서 정의가 가능한 것 같다.
```


## 객체 연산자 

##string 연산자 중복 정의
```c++
string 클래스는 연산자 중복을 사용해오고 있었다. string 클래스는 +, =, ==, != 등의 연산자들이 정의되어 있어서 편리하게 문자열 처리를 할 수 있었다.

중복할 수 없는 연산자를 알아보면 다음과 같다.
:: 범위 지정 연산자
. 멤버 선택 연산자
.* 멤버 포인터 연산자
?: 조건 연산자

이들 연산자는 모두 상당히 중요한 역할을 하는 연산자이기에 이들 연산자의 의미를 중복 정의는 금하고 있다.

다음 장에서는 +, ==, !=, ++, =, [], *, -> 연산자의 중복 정의를 알아보고자 한다.
```

## + 연산자 중복 정의

```c++
#include <iostream>
#include <string>
using namespace std;

class MyVector {
private: 
   double x, y; // 지역변수 x, y 선언
public: 
 MyVector(double x= 0.0, double y= 0.0) : x(x), y(y) { }
 string toString() 
   return"("+to_string(x) +", "+to_string(y) +")";  // to_string은 string 클래스의 멤버함수
 MyVector operator+(const MyVector& v2);
}

MyVector MyVector::operator+(const MyVector& v2);
{
  MyVector v;
  v.x = this->x + v2.x; // this->x : v1의 지역변수, this는 v1
  v.y = this->y + v2.y; // this->y : v1의 지역변수  this는 v1
  return v;
}

int main()
{
  MyVector v1(1.0, 2.0), v2(3.0, 4.0);
  MyVector v3 = v1 + v2; // 복사 생성자 호출 (operator+ 함수에서 매개변수로 객체를 넘겨줌 (return v))
  
  // v1 + v2 <- v1. operator+(v2) // v2가 매개변수로 넘어감
}
```

## ==, != 연산자 중복
```c++
#include <iostream>
using namespace std;

class Time
{
  int hour,min,sec;
public:
  Time(int h= 0, int m= 0, int s = 0) : hour(h), min(m), sec(s) {  }
  
  bool operator== (Time &t2) {
    return (hour == t2.hour && min == t2.min && sec == t2.sec);
  }
  
  bool operator!= (Time &t2) {
    return !(*this == t2);
  }
};

int main()
{
  Time t1(1, 2, 3), t2(1, 2, 3);
  
  cout.setf(cout.boolalpha); // 출력 객체 cout의 멤버함수 setf(setformat) cout.boolalpha : 0, 1이 아니고 true false로 찍어라 
  cout << (t1 == t2) << endl;
  cout << (t1 != t2) << endl;
  return 0;
}
```

## ++ Prefix 전위 연산자 중복

```c++
#include <iostream>
using namespace std;

class Counter {
private:
  int value;
public:
  Counter(): value(0) {};
  ~Counter() { }
  int getValue() const { return value; } // 함수의 반환값이 const
  void setValue(int x) { value =x; }
  Counter& operator++() // 자기가 자기 호출해서 매개변수 없음 & 써주는 이유 : 본객체를 ++ 증가
  {
    ++value; // 지역변수
    return *this; // 복사하기 위해서 생성된 임시 객체가 아니라 본객체를 넘겨주기위해서 // ++(++c) = ++(c.operator++()) = ++(c) 
  }
};

int main()
{
  Counter c;
  cout << "카운터의 값: " << c.getValue() << endl;
  ++c;
  cout << "카운터의 값: " << c.getValue() << endl;
  return 0;
}
```

## ++ postfix 후위 연산자 중복

전위연산자와 후위연산자를 구별하기 위해서 매개변수에 int형을 추가한다.
```c++
const Counter operator++(int i) // int i 매개변수 넘어가는 것도 없고 아무 의미없고 그냥 후위연산자임을 알려주기 위해 넣어줌, const넣어주는 이유 (c++)++ 오류뜨게 하려고, profix는 반환되는 임시객체 ++ 할필요가 없으므로 & 필요없음
{
  Counter temp = {*this}; // 임시 객체 temp에 본객체를 저장해서 보관
  ++value; // 본객체 ++ 증가
  return temp; // 보관된 객체 넘겨줌
}
```

```c++
(v++)++ X
++(v++) O
```

## 대입 연산자 중복

```c++
#include <iostream>
using namespace std;
class Box
{
private:
   double length, width, height;
public:
   Box(double l= 0.0, double w = 0.0, double h = 0.0) : length(l), width(w), height(h) { }
   void display() {
      cout << "(" << length << ", " << width << ", " << height << ")" << endl;
   }
   
   Box& operator=(const Box& b2)
   {
      this->length = b2.length;
      this->width = b2.width;
      this->height = b2.height;
      return this;
   }
};

int main()
{
   Box b1(30.0, 30.0, 60.0), b2;
   b1.display();
   
   b2 = b1; // b2.operator=(b1)
   b2.display():;
   
   return 0;
}

대입 연산자는 참조자를 반환해야 한다.   
그 이유는 대입 연산자는 연속하여 적용될 수 있기 때문이다.   
대입 연산자와 복사 생성자는 많이 혼동되는 문제이다.   

```c++
MyArray first; // 기본 생성자 호출
MyArray second(first); // 복사 생성자 호출
MyArray third = first; // 복사 생성자 호출
MyArray third {first}; // 복사 생성자 호출

second = third; // 대입 연산자 호출
```

## 인덱스 연산자 [ ]의 중복

```c++
int A[10];
A[-1] = 0; // 오류
```

인덱스 연산자의 중복 -> 인덱스의 범위를 체크

```c++
int &operator[] (int i) { // 형식이 아니라 인덱스 번호, &를 넣어주는 이유 : 자기 자신 객체 A[3] 이게 없으면 복사되서 
   if(i >= SIZE) {
      cout << "잘못된 인덱스: ";
      return a[0];
   }
   else if(i < 0) // 0보다 작은 음수라면
   {
      cout << "잘못된 인덱스: ";
      return a[0];
   }
   return a[i];
}

int main() {
   MyArray A;
   
   A[3] = 9; // 인덱스 연산자 함수 호출 A[3].operator[](3) = 9; return a[3] = 9
   cout << "A[3]=" << A[3] << endl;
   cout << "A[16]=" << A[16] << endl;
   
   return 0;
}
```

## 포인터 연산자의 중복

## 상속(inheritance)

class Car
{
   int speed; // 속성값
};

class SportsCar: public Car
{
   bool turbo; // 속성값
};

SportsCar 클래스는 Car 클래스를 상속한다.

Class Diagram (UML) 그림으로 표현   
- : private member
+ : public member

```c++
#include <iostream>
#include <string>
using namespace std;

class Car {
private:
   int speed; // 속도
public:
   void setSpeed(int s) { speed = s; }
   int getSpeed() { return speed; }
};

class SportsCar : public Car {
private:
   bool turbo;
public:
   void setTurbo(bool newValue) { turbo = newValue; }
   bool getTurbo() { return turbo; }
};

int main()
{
   SportsCar c; // 부모클래스 메모리 먼저 잡히고, 자식클래스 메모리 잡힌다.
   
   c.setSpeed(60); // 부모클래스 Car 함수 호출
   c.setTurbo(true); // 자기자신 자식클래스 함수 호출
   c.setSpeed(100);
   c.setTurbo(false);
   return 0;
}

자식클래스는 부모클래스의 속성값을 사용할 수 있다. (public)

상속은 왜 필요한가?   
-> 공통되는 속성을 뽑아서 상위 클래스(부모 클래스)를 만드는 일반화(generalization)을 거친다.(상속개념)   
<-> 반대로 공통된 클래스로 하위클래스(자식 클래스)를 만드는 것(specialization)   
상속은 여러 단계로 이루어질 수 있다.   
상속은 is-a kind of 관계이다.   
ex) 자동차는 탈 것의 종류 중 한 개이다.(일반화)   
has-a (composed of) ~로 구성되어 있다. (집합 관계)는 상속으로 모델링을 하면 안 된다.   
library has a book.   

parent=base=super class 상위      
child=derived=sub class 하위   

## 상속과 생성자/소멸자

객체가 생성될 때 자식 클래스 생성자 뿐만 아니라 부모클래스의 생성자도 호출된다.   
장유유서 개념 -> 부모 클래스 생성자 호출 먼저 되고 자식 클래스 생성자가 호출된다.   
소멸될 때는 자식클래스가 먼저 소멸되고 그 다음 부모 클래스가 소멸된다.   

```c++
#include <iostream>
#include <string>
using namespace std;

class Shape {
private:
   int x, y;
public:
   Shape() { cout << "SHAPE 생성자" << endl; }
   ~Shape() { cout << "SHAPE 소멸자" << endl; }
};

class Rectangle : public Shape {
private:
   int width, height;
public:
   Rectangle() { cout << "RECTANGLE 생성자" << endl; }
   ~Rectangle() { cout << "RECTANGLE 소멸자" << endl; }
};

int main()
{
   Rectangle r; // Rectangle 객체 r 생성
   return 0; // 생성된 객체가 소멸되어야 한다.
}

```
출력결과
```c++
SHAPE 생성자   
RECTANGLE 생성자
RECTANGLE 소멸자
SHAPE 소멸자
```

## 부모 클래스의 생성자를 지정하는 방법

부모 클래스의 생성자가 default 생성자, 매개변수 초기화하는 생성자 등 여러 개가 있을 수 있다.  
이때 부모 클래스의 생성자를 지정할 수 있는 방법이 있다.   

```c++
자식 클래스의 생성자() : 부모클래스의 생성자()
{
}
```
예제
```c++
Rectangle (int x=0, int y=0, int w=0, int h=0) : Shape(x, y)
{
   width = w;
   height = h;
}
뒤에 오는 부모클래스의 생성자는 값을 호출하는 형태로 지정해야 한다. Shape(int x, int y) X

## 접근 지정자

protected 지정자는 상속받는 클래스에서 사용한다.   
외부에서는 사용x(전용멤버)이지만 자식 클래스는 protected 사용 가능

부모 클래스에 protected를 지정해주면 자식 클래스에서만 사용 가능, 외부에서는 사용X

## 멤버 함수 재정의

polymophism 함수 다형성   

1. 함수 중복정의(overloading)   
2. 함수 재정의(overriding) 상속에서 사용   

부모클래스의 정의되어 있는 함수를 자식클래스에 맞게끔 자식클래스에서 정의하는 것을 함수 재정의(overriding)이라고 한다.   

같은 클래스 내에서 함수 정의 -> 중복정의   
상속 관계 클래스에서 함수 정의 -> 재정의   

## 부모클래스의 함수를 호출하고 싶다.

```c++
void print() { // print함수 재정의를 했지만
   ParentClass::print(); 부모 클래스의 print를 호출하고 싶을 때 다음과 같이 사용한다.
```   

## 상속 받는 경우

private로 상속 -> 부모클래스 속성 접근 x      
public 로 상속 -> public으로 상속      
protected로 상속 -> protected으로 상속      

일반적으로 public으로 쓰지만 좀 더 수준이 높은 보안이 필요할 때는 protected를 사용한다.   
상속 받을 때 접근 지정자를 줄 수 있다.
```c++
class Derived : private Base // public, protected 속성값들을 상속을 받지만 자식 클래스에서 private가 된다.
```

## 다중 상속 (multiple inheritance)

```
#include <iostream>
using namespace std;

class PassangerCar {
private:
   int seats;
   void se_seats(int n) { seats = n; }
};

class Truck {
public:
   int payload;
   void set_payload(int load) { payload = load; }
};

class Pickup : public PassangerCar, public Truck { // PassangerCar, Truck, Pickup 순으로 메모리가 잡힌다.
public: 
   int tow_capability;
   void set_tow(int capa) { tow_capability = caps; }
};

int main()
{
   Pickup my_car;
   my_car.set_seats(4);
   my_car.set_payload(10000);
   my_car.set_tow(30000);
   return 0;
}
```

   
