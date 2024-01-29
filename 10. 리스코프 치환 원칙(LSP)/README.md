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
Square와 Circle 구조체는 Shape에서 파생되고 Draw() 함수를 갖지만, Shape에있는 함수를 오버라이드 하지 않고, Square와 Circle이 Shape를 대체할 수 없기에, DrawShape는 인자로 받는 Shape를 검사하고, 형을 결정하고 나서, 적절한 Draw 함수를 호출해야 한다.  
Square와 Circle이 Shape를 대체할 수 없다는것은 LSP 위반이며, 이 위반은 DrawShape의 OCP 위반을 유발한다. **그러므로 LSP 위반은 잠재적인 OCP 위반이다.**  

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
어느날 사용자가 직사각형뿐만 아니라 정사각형도 조작할 수 있게 해달라고 요구해왔다.  
새로운 종류의 객체가 원래 종류의 객체와 IS-A(~이다) 관계를 이룬다 말할 수 있으려면, 새 객체의 클래스는 원래 객체의 클래스에서 파생될 수 있어야 한다.  
일반적으로 정사각형은 직사각형이라고 할수 있기에, Square 클래스는 Rectangle클래스에서 파생된다고 보는게 합리적이고 다음같은 상속관계를 갖는다.  
![KakaoTalk_20240128_141048661](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/c39d4747-571d-42ab-b49a-8b1e41661921)
이 같은 IS-A관계는 객체 지향 분석의 기본적인 기법중 하나로 생각될 수 있지만,  
**이런 식의 생각은 미묘하지만 심각한 문제를 낳을 수 있다.**  
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
### 동기
한 사례로 실제 몇몇 컨테이너 클래스를 포함한 서드파티 클래스 라이브러리를 구매했다고 한다.  
이 컨테이너들은 스몰토크 언어의 Bag 및 Set 클래스와 관련이 있었다고 하는데 두종류의 Set 클래스와 비슷한 두 종류의 Bag 클래스가 있었다고 한다.  
첫번째 종류는 유계(bounded)라고 불리며 배열에 기반을 두고 있었고, 두번째 종류는 무계(unbounded)라 불리며 연결 리스트에 기반을 두고 있었다.  
BoundedSet의 생성자는 저장할 수 있는 원소의 최대 개수를 명시했고, 배열에 기반을 두고 있었기에 매우 빠르고 정상적인 동작을 하는중에는 메모리 할당작업이 이루어지지 않는다.  
UnboundedSet은 저장 가능한 원소의 개수 한도를 선언하지 않았고, 정상적인 동작의 일부로 메모리를 계속 할당하고 해제해야 하기에 속도는 느렸다.  
이 서드파티 클래스들의 인터페이스가 불편해 좀더 나은 클래스로 교체하고자 했고, 애플리케이션 코드가 이 클래스들에 의존하게 되는것을 원하지 않았다고 한다.  
![KakaoTalk_20240128_190002961](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/a44aea5c-7c88-43a6-8ce9-d61827d5375f)
위 그림과 같이 작성한 추상 인터페이스로 서드파티 컨테이너를 포장했다고 한다.  
```CPP
template <class T> class Set {
  public : virtual void Add(const T&) = 0;
    virtual void Delete(const T&) = 0;
    virtual void IsMember(const T&) const = 0;
};
```
위 코드와 같이 순수 가상 함수인 Add, Delete, IsMember를 제공하는 Set이라는 추상 클래스를 만들었고, 이 구조는 2개의 서드파티 Set 변형 클래스를 통합하고 공통의 인터페이스를 통해 그것에 접근할 수 있게 만들었다.  
클라이언트는 실제로 그것이 처리하는 Set이 bounded 종류든 unbounded 종류든 신경쓰지 않을것이고, 이 점은 큰 장점이다.  
프로그래머가 각 특정 인스턴스에서 어떤 종류의 Set이 필요한지 결정할 수 있고, 클라이언트 함수중 어느것도 그 결정에 영향을 받지 않는다는 뜻이다.  

### 문제
PersistentSet을 계층구조에 추가하려고 한다. 영속 집합(persistent set)은 어떤 스트림에도 쓰이고, 나중에 다른 애플리케이션에 의해서도 다시 읽힐 수 있는 집합이다.  
이 클래스는 추상 기반 클래스인 PersistentObject 클래스에서 파생된 객체를 받아들였고 다음과 같은 계층 구조를 만들었다.  
![KakaoTalk_20240128_212449833](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/5c18f318-97fd-4f70-8eee-770283a39262)
PersistentSet은 단순히 서드파티 영속 집합에 위임하는 역할을 하기 때문에, PersistentSet에 추가되는 원소는 PersistentObject에서 파생되어야 한다.
그럼에도 불구하고 Set의 인터페이스는 이런 제약을 갖고 있지 않다.  
어떤 클라이언트가 기반 클래스 Set에 멤버를 추가할 때, 그 클라이언트는 Set이 실제로 PersistentSet인지 아닌지 알수 없고, 따라서 PersistentObject에서 파생된것인지 알수있는 방법이 없다.  
```CPP
template<typename T>
void PersistentSet :: Add(const T& t) {
  PersistentObject& p = dynamic_cast<PersistentObject&>(t);
  itsThirdPartyPersistentSet.Add(p);
}
```
이 코드는 PersistentObject 클래스에서 파생되지 않은 객체를 PersistentSet에 추가하려는 클라이언트가 있으면, 런타임 에러가 발생할 것임을 확실하게 해준다.  
이런 함수들은 Set의 파생 클래스에서 혼동 될것이기 때문에, 이러한 계층 구조 변경은 LSP를 위반한다.  
Set의 파생 클래스를 넘겨줄 떄는 전혀 문제가 없던 함수들이 PersistentSet을 넘겨줄 때는 런타임 에러를 발생시키고, 이런 종류의 문제를 디버깅하는 일은 상대적으로 어렵다.  
이 위치를 찾아내는 것도 몹시 까다로운 일이지만, 이것을 고치는 건 더 어렵다.

### LSP를 따르지 않는 해결책
몇년전의 필자는 이것을 규정(convention)에 따라 해결했다 하는데, 소스코드에서 해결하지 않았다는 의미라고 한다.  
PersistentSet과 PersistentObject가 전부 애플리케이션에 알려지지는 않게 한다는 규정을 세웠다고 하는데, 이들은 단 하나의 특정한 모듈에만 알려진다.  
이 모듈은 영속 저장소에서 모든 컨테이너를 읽고쓰는 책임을 졌고, 어떤 컨테이너에 써야 할 때, 그 내용은 적절한 PersistentObject의 파생 객체에 복사되고 PersistentSet에 추가되고, 이 PersistentSet는 스트림에 저장된다.  
컨테이너가 스트림에서 읽고자 할때는 과정이 반대로 일어나게되고, PersistentSet은 스트림에서 읽히며, PersistentObject는 PersistentSet에서 지워지고 일반 객체에 복사되며, 이 객체는 일반 Set에 추가된다.  
이 해결책이 지나치게 제한적인것처럼 보일 수도 있지만, 이때 당시 필자가 생각하는 유일한 방법이었다고 한다.  
이 해결책은 제대로 된 효과를 보지 못했다 하는데, 이 규정의 필요성을 이해하지 못한 개발자들이 애플리케이션의 일부분에서 위반했다고 한다.  
이것이 바로 규정과 관련해 생기는 문제이고, 끊임없이 각 개발자에게 알려야 한다. 그렇지 않으면 규정은 위반되게 되고, 한군데의 위반은 전체 구조를 손상시킬 수 있다.  

### LSP를 따르는 해결책
지금이라면 PersistentSet이 Set과 IS-A 관계에 있지 않다는 사실과, 이 클래스가 Set의 적절한 파생 클래스가 아님을 인정하고, 계층 구조를 분리하지만 완전히 분리하지는 않을 것이다.  
사실 LSP 관점에서 문제가 되는 것은 Add메소드뿐이다. 결과적으로 멤버 여부 테스트, 순환 등을 허용하는 추상 인터페이스 아래 Set과 PersistentSet 모두를 형제 관계로 묶는 계층 구조를 만들 것이다.  
![KakaoTalk_20240128_223924956](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/0e49a5b2-490e-491b-bd19-bfaaf885eef8)
하지만 PersistentObject에서 파생되지 않은 객체를 PersistentSet에 추가할 수 있게 만들 수는 없다.  

## 파생 대신 공통 인자 추출하기




## 휴리스틱과 규정
LSP 위반의 단서를 보여주는 간단한 휴리스틱이 있다. 이것은 기반 클래스에서 어떻게든 기능성을 제거한 파생 클래스에 대해 적용해야 한다.  
기반 클래스보다 덜한 동작을 하는 파생 클래스는 보통 그 기반 클래스와 치환이 불가능하므로 LSP를 위반한다.  

### 파생 클래스에서의 퇴화 함수
다음 코드를 보자
```JAVA
public class Base {
  public void f() {/**일부 코드**/}
}

public class Derived extends Base {
  public void f() {}
}
```
Base에 함수 f가 구현되어 있지만 Derived에서 이것은 퇴화한다.  
유감스럽게도 Base의 사용자는 f를 호출하면 안된다는 사실을 모르기 때문에, 여기서는 치환 위반이 생긴다.  
파생 클래스에서 퇴화 함수가 존재한다고 해서 무조건 LSP위반을 나타낸다고 할 수는 없다. 하지만 이것이 일어났을 때 위반 여부를 살펴볼 만한 가치는 있다.  

### 파생 클래스에서의 예외 발생
또 다른 위반 형태는 그 기반 클래스가 발생시키지 않는 예외를 파생 클래스의 메소드에 추가하는 것이다.  
기반 클래스의 사용자가 예외를 기대하지 않는다면, 파생 클래스의 메소드에 예외를 추가했을 때 이들은 치환 가능하지 않다.  
사용자의 기대가 변하든지, 아니면 파생 클래스가 그 예외를 발생시키지 않아야 한다.  

## 결론
OCP는 객체지향설계를 위해 논의된 수많은 의견중에서도 핵심이고, LSP는 OCP를 가능하게 하는 주요 요인 중 하나다. 
이것은 기반타입으로 표현된 모듈을 수정 없이도 확장 가능하게 만드는, 서브타입의 치환 가능성을 말한다.  
이 치환 가능성은 암암리에 의존할 수 있는 그 어떤것이 되어야 하고, 기반타입의 계약사항은 명시적으로 강제되지 않은 경우 코드에서 분명하고 뚜렷해야 한다.  
서브타입의 진실된 정의는 '치환 가능성'이다. 여기서 치환 가능성은 명시적 또는 암묵적 계약에 의해 정의된다.
