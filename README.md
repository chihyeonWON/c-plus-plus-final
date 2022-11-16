# c-plus-plus-final
c-plus-plus-final

# 객체 연산자 

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
  Counter& operator++() // 자기가 자기 호출해서 매개변수 없음
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




