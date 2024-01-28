# 10. 리스코프 치환 원칙(LSP)

## 리스코프 치환 원칙(LSP)
LSP는 다음과 같이 설명할 수 있다.  
> 서브타입(subtype)은 그것의 기반 타입(base type)으로 치환 가능해야 한다.

## LSP 위반의 간단한 예
LSP 위반은 대개 심각하게 OCP를 위반하는 런타임 타입 정보 사용으로 이어진다.  
다음을 보자  
```CPP
struct Point {
  douvle x, y;
};

struct Shape {
  enum ShapeType {
    square, circle }
  itsType;
  Shape(ShapeType t) : itsType(t) {
  }
};

struct Circle : public Shape {
  Circle() : Shape(circle) {
  }
  void Draw() const
  Point itsCenter;
  double itsRadius;
};

struct Circle : public Shape {
  Square() : Shape(square) {
  }
  void Draw() const
  Point itsTopLeft;
  double itsSide;
};

void DrawShape(const Shape& s) {
  if(s.itsType == Shape :: square)
    static_cast<const Square&>(s).Draw();
  else if(s.itsType = Shape :: circle)
    static_cast<const Circle&>(s).Draw();
}
```
이 코드의 DrawShape 함수는 OCP를 위반하는데, 이 함수는 Shape 클래스의 모든 가능한 파생 클래스를 알아야 하고, Shape의 새로운 파생 클래스가 생길때마다 변경 되어야 한다.  
이 함수의 구조는 좋지 않은 설계인데 무엇이 프로그래머로 하여금 이런 함수를 작성하게 만드는 걸까?  
프로그래머는 객체 지향 기술을 공부하고 다형성의 부하가 지나치게 크다는 결론에 도달하는데, 그래서 아무 가상함수도 포함하지 않는 Shape 클래스를 정의했다.  
Square와 Circle 구조체는 Shape에서 파생되고 Draw() 함수를 갖지만, Shape에있는 함수를 오버라이드 하지 않는다.  
Square와 Circle이 Shape를 대체할 수 없기에, DrawShape는 인자로 받는 Shape를 검사하고, 형을 결정하고 나서, 적절한 Draw 함수를 호출해야 한다.  
Square와 Circle이 Shape를 대체할 수 없다는것은 LSP 위반이며, 이 위반은 DrawShape의 OCP 위반을 유발한다. ***그러므로 LSP 위반은 잠재적인 OCP 위반이다.***  

## 정사각형과 직사각형, 좀 더 미묘한 위반
LSP를 위반하는 훨씬 미묘한 경우가 존재한다. 다음과같은 Rectangle 클래스를 사용하는 어떤 어플리케이션을 생각해보자
```CPP
class Rectangle {
  public :
    void SetWidth(double w) {
      itsWidth = w;
    }
    void SetHeight(double h) {
      itsHeight = h;
    }
    void GetWidth() const {
      return itsWidth N;
    }
    void GetHeight() const {
      return itsHeight;
    }
  private :
    point itsTopLeft;
    double itsWidth;
    double itsHeight;
};
```
이 어플리케이션이 잘 동작하고 많은 장소에 설치되었다고 가정해보자. 어느날 사용자가 직사각형뿐만 아니라 정사각형도 조작할 수 있게 해달라고 요구해왔다.  
새로운 종류의 객체가 원래 종류의 객체와 IS-A(~이다) 관계를 이룬다 말할 수 있다면 새 객체의 클래스는 원래 객체의 클래스에서 파생될 수 있어야 한다.  
일반적으로 정사각형은 직사각형이라고 할수 있기에, Square 클래스는 Rectangle클래스에서 파생된다고 보는게 합리적이고 다음같은 상속관계를 갖는다.  
![KakaoTalk_20240128_141048661](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/c39d4747-571d-42ab-b49a-8b1e41661921)
이 같은 IS-A관계는 객체 지향 분석의 기본적인 기법중 하나로 생각될 수 있다. 그러나 이런 식의 생각은 미묘하지만 심각한 문제를 낳을 수 있다.  
잘못되었다는 첫번째 증거는 Square가 itsWidth와 itsHeight 멤버변수를 필요로 하지 않는다는 사실이다.  
그럼에도 불구하고 Square는 Rectangle에서 이 멤버변수들을 상속받는데, 분명 이것은 소모적이지만 낭비가 심각하지는 않다.  
하지만 수십만개의 Square객체를 생성해야 한다면 이런 낭비는 심각한 문제가 될 수 있다.  
Rectangle에서 Square를 파생하는것 때문에 생기는 다른 문제도 있는데, Square는 SetWidth와 SetHeight 함수를 상속하는데 정사각형의 가로와 세로 길이는 같으므로 이 함수들은 Square에서는 부적절하다.  
이것은 문제가 있다는 분명한 징후이고, 이것을 비껴갈 수 있는 방법으로, 다음처럼 오버라이드 할 수 있다.  
```CPP
void Square :: SetWidth(double w) {
  Rectangle :: SetWidth(w);
  Rectangle :: SetHeight(w);  
}

void Square :: SetHeight(double h) {
  Rectangle :: SetHeight(h);
  Rectangle :: SetWidth(h);
}
```
이제 누군가 Square의 가로를 설정한다면 세로도 똑같이 바뀔 것이다. 마찬가지로 세로를 바꿔도 가로도 맞춰 바뀐다.  
그러므로 Square의 불변식은 손상되지 않는다. 그러나 다음 함수에 대해 생각해보자.
```
void f(Rectangle & r) {
  r.SetWidth(32);  //Rectangle::SetWidth를 호출한다.
}
```
만약 어떤 Square객체에 대한 참조값을 이 함수에 넘겨준다면, 그 Square객체는 세로 값이 가로값에 맞춰 바뀌지 않기때문에 문제가 생길 것이다. 이것은 명백한 LSP 위반이다.  
이 실패의 원인은 SetWidth와 SetHeight를 virtual로 선언되지 않아서, 다형적이지 않다는 사실에 있다.  
이 문제는 쉽게 고칠수 있지만, 파생 클래스를 만드는 것이 기반 클래스의 변경으로 이어질 때, 대개는 이 설계에 결점이 있음을 의미한다.  
클래스를 고친 결과는 다음과 같을 것이다.
```CPP
class Rectangle {
  public :
    void SetWidth(double w) {
      itsWidth = w;
    }
    void SetHeight(double h) {
      itsHeight = h;
    }
    void GetWidth() const {
      return itsWidth N;
    }
    void GetHeight() const {
      return itsHeight;
    }
  private :
    point itsTopLeft;
    double itsWidth;
    double itsHeight;
};

class Square : public Rectangle {
  public :
    virtual void SetWidth(double w);
    virtual void SetHeight(double h);
};

void Square :: SetWidth(double w) {
  Rectangle :: SetWidth(w);
  Rectangle :: SetHeight(w);  
}

void Square :: SetHeight(double h) {
  Rectangle :: SetHeight(h);
  Rectangle :: SetWidth(h);
}
```

### 본질적인 문제
Square와 Rectangle은 이제 제대로 동작하는 것처럼 보이고, 이 설계는 자체 모순이 없고 올바르다는 결론을 내릴 수 있다. 하지만 이 결론은 잘못된 것이다.  
자체 모순이 없는 설계라고 해서 반드시 모든 사용자에 대해 모순이 없는 것은 아니다. 다음을 보자
```CPP
void g(Rectangle& r) {
  r.SetWidth(5);
  r.SetHeight(4);
  assert(r.Area() == 20);
}
```
이 함수는 Rectangle에 대해서는 올바르게 동작하겠지만, Square를 넘겨받는다면 단정 에러(Assertion error)를 선언한다.  
본질적인 문제는 **'g의 작성자는 Rectangle의 가로 길이를 바꾸는 것이 세로 길이를 바꾸지는 않을 것이라고 생각한다'** 는데 있다.  
함수 g는 Square/Rectangle 계층 구조에 대해 취약하다. 이런 함수에서는 Square가 Rectangle과 치환가능하지 않고, LSP를 위반하게 된다.  

### 유효성은 본래 갖추어진 것이 아니다
LSP는 **'모델만 별개로 보고, 그 모델의 유효성을 충분히 검증할 수 없다'** 라는 아주 중요한 결론을 내린다.  
특정 설계가 적절한지 아닌지를 판단할 때는 단순히 별개로 봐선 해답을 찾을수 없고, 그 설계를 사용자가 택한 합리적인 가정의 관점을 봐야한다.  
대부분 이러한 가정들은 예상하기가 쉽지 않고, 이것들을 전부 예상하려 한다면, 시스템을 **불필요한 복잡성**의 악취로 찌들게 하는 결과를 낳게 될 것이다.  
그러므로 다른 모든 원칙과 마찬가지로, 관련된 취약성의 악취를 맡을 때까지 가장 명백한 LSP 위반을 제외한 나머지의 처리는 연기하는게 최선이다.  

### 'IS-A'는 행위에 대한 것이다
함수 g의 관점에서 볼때 Square객체는 절대로 Rectangle 객체가 아니다. Square 객체의 행위가 g가 기대하는 Rectangle 객체의 행위와 일치하지 않기 때문이다.  
행위 측면에서 Square는 Rectangle이 아니고, 행위야말로 소프트웨어의 모든 것이다.  
LSP는 객체지향설계 에서 IS-A 관계는 합리적으로 가정할 수 있고 클라이언트가 의존하는 행위와 관련이 있다는 점을 분명히 한다.  

### 계약에 의한 설계
많은 개발자가 '합리적 추정'이라는 개념에 불편함을 느끼기도 하는데, 이런 합리적인 추정을 명시적으로 만들어 LSP를 강제하는 테크닉이 있다.  
이를 계약에 의한 설계(DBC: design by contract)라고 한다.  
DBC를 사용하면, 어떤 클래스의 작성자는 그 클래스의 계약사항을 명시적으로 정한다. 이 계약은 모든 고객 코드의 작성자가 신뢰할 수 있는 행위에 대해 알려준다.  
또한 이 계약은 각 메소드의 사전조건과 사후조건을 선언하는 것으로 구체화된다.  
메소드를 실행하기 위해서는 사전조건이 참이 되어야 하고, 메소드가 완료되고 나면 사후조건이 참이 됨을 보장한다.  
기반 클래스의 인터페이스를 통해 어떤 객체를 사용할 때, 사용자는 그 기반 클래스의 사전조건과 사후조건만 알 수 있고, 파생된 객체는 기반 클래스가 받아들일 수 있는 것은 모두 받아들여야 한다.  
또한 파생클래스는 기반 클래스의 모든 사후조건을 따라야 한다. 즉 그 행위와 출력은 기반 클래스의 제약을 위반해서는 안된다.  

### 단위테스트에서의 계약사항 구체화하기
계약은 또한 단위 테스트를 작성함으로서 구체화될 수 있다.  
단위 테스트는 어떤 클래스의 행위를 철저하게 테스트 함으로서, 그 클래스의 행위를 좀 더 분명하게 만들어준다.  

## 실제 예
