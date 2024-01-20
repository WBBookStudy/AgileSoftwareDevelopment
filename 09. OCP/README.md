# 09. 개방 폐쇄 원칙
소프트웨어 개체는 확장에 대해 열려 있어야하고, 수정에 대해서는 닫혀 있어야 한다.  

## 상세 설명
개방 폐쇄 원칙을 따르는 모듈은 다음과 같은 두 가지 중요한 속성을 갖는다.
1. 확장에 대해 열려 있다. 요구사항이 변경될 때, 변경에 맞게 새로운 행위를 추가해 모듈을 확장할 수 있다.
2. 수정에 대해 닫혀 있다. 어떤 모듈의 행위를 확장하는 것이 그 모듈의 소스 코드나 바이너리 코드의 변경을 초래하지 않는다.  
  
서로 반대입장에 있는것 같은 이 두 원칙을 어떻게 가능하게 만들까?  

## 해결책은 추상화다
![KakaoTalk_Photo_2024-01-20-16-42-40](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/38aedc00-4ec2-46b1-9626-612eefb5cdac)
> OCP를 따르지 않는 간단한 설계. 클라이언트와 서버 둘 다 구체적이고, 클라이언트 클래스는 서버 클래스를 **사용한다.** 만약 클라이언트 객체가 다른 서버 객체를 사용하게 되면 클라이언트 클래스가 새로운 서버 클래스를 지정하도록 변경해야한다.
![KakaoTalk_Photo_2024-01-20-16-42-43](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/dcab5c28-e1f8-4f5a-83ae-953580d50c95)
> OCP를 따르는 같은 설계를 보여준다. 서버가 바뀌어야 한다고 해도, 클라이언트의 코드는 바뀌지 않을것이다. 

## Shape 애플리케이션
### OCP 위반

```CPP
// shape.h

enum ShapeType { circle, square }

struct Shape {
  ShapeType itsType;
}
```
```CPP
// circle.h
struct Circle {
  ShapeType itsType;
  double itsRadius;
  Point itsCenter;
}
```
```CPP
// square.h 
struct Square {
  ShapeType itsType;
  double itsSide;
  Point itsTopLeft;
}
```
```CPP
// drawAllShapes.cc
typedef struct Shape *ShapePointer;

void DrawAllShapes(ShapePointer list[], int n) {
  int i;
  for (i = 0 ; i < n ; i++) {
    struct Shape* s = list[i];
    switch (s -> itsType) {
    case square:
      DrawSquare((struct Square*) s);
      break
    case circle:
      DrawCircle((struct Circle*) s);
      break
    }
  }
}
```
자료구조가 원인지 사각형인지 식별후 그리기  
DrawAllShapes은 새로운 도형 종류에 대해 닫혀 있을 수 없기 때문에 OCP를 따르지 않는다.  
1. 만약 도형이 추가되면 DrawAllShapes를 수정해야한다. 
2. ShapeType enum에 새 맴버를 추가해야 한다.
3. Shape 자료 구조를 사용하는 모든 모듈의 바이너리 파일을 변경해야한다. 영향이 엄청 커짐.. 

### OCP 따르기

```CPP
class Shape {
  public : virtual void Draw() const = 0;
}

class Square : public Shape {
  public : virtual void Draw() const
}

class Circle : public Shape {
  public : virtual void Draw() const
}

void DrawAllShapes(vector<Shape*> & list) {
  vector<Shape *> :: iterator i;
  for (i = list.begin() ; i != list.end() ; i++) 
    (*i) -> Draw();
}
```
이제 새로운 파생 클래스를 만들어도 DrawAllShapes를 수정 할 필요가 없다. 따라서 DrawAllShapes는 OCP를 따른다.  
실제로 Triangle같은거 추가해도 모듈에 아무런 영행을 주지 않는다.  
  
이 해결책은 **경직성**도 없다. 수정되야 하는 소스 모듈도 없을 뿐더러, 하나만 재외한다면 재빌드되어야 하는 바이너리 모듈도 없다.  
이 해결책은 **부동성**도 없다 DrawAllShapes는 Square와 Circle의 편승 없이도 다른 어플리케이션에서 재사용될 수 있다.

## 예상과 '자연스러운' 구조
모든 상황에서 자연스러운 모델은 없다.  
폐쇄는 완벽할 수 없기 떄문에 전략적이어야 한다. 설계자는 자신의 설계에서 닫혀 있는 변경의 종류를 선택해야 한다.  
또한 OCP는 비용이 많이 든다. (시간, 노력등) 따라서 개발자는 있을법한 변경정도에만 OCP를 사용하고 싶어한다.  
어떤 변경이 있을지 어떻게 알 수 있을까? 

## 올가미 놓기
변경으로부터 우리 자신을 어떻게 보호할 수 있을까? 일어날 수 있다고 생각되는 변경에 대해 '올가미를 놓는'것.  
그러나 올가미를 놓으면 그것이 사용되지 않음에도 불구하고 유지보수되어야 하는 **불필요한 복잡성**의 악취를 풍겼다.  
첫 번째 총알은 그냥 맞고, 그 다음에 추상화를 시키는 방법도 있다.  

### 변경 촉진하기
첫 번째 총알을 맞기로 결정했다면 총알이 빨리, 자주 날라오면 유리하다. 우리는 개발이 더 진행되기 전에 어떤 종류의 변경이 일어날지 알고싶어진다. 그 방법은...  
 - 테스트를 먼저 작성한다. 시스템이 테스트 가능하도록 만들고, 테스트가 가능하도록 만들어달라는 변경은 우리를 놀라게만들지 않는다.
 - 아주 짧은 주기(주보다는 일)로 개발한다.
 - 기반구조보다 기능 요소를 먼저 개발하고, 자주 이 기능 요소를 이해당사자에게 보여준다.
 - 가장 중요한 기능 요소를 먼저 개발한다.
 - 소프트웨어를 빨리, 그리고 자주 릴리즈한다. 가능 한 빠르게, 가능한 한 자주 고객과 사용자 앞에서 그것을 시연한다.

## 명시적인 폐쇄를 위해 추상화 사용하기
폐쇄는 추상화에 기반을 둔다.  
```CPP
class Shape {
  public : virtual void Draw() const = 0;
    virtual bool Precedes(const Shape& s) {
      return Precedes(s);
    }
}

template <typename P> class Lessp { // 포인터의 컨테이너를 정렬하기 위한 유틸리티
  public : bool operator() (const P p, const P q) {
    return (*p) < (*q);
  }
}

void DrawAllShapes(vector<Shape*>& list) {
  vector<Shape *> orderedList = list;
  sort(orderedList.begin(), orderedList.end(), Lessp<Shape *>());
  vector<Shape *> :: const_iterator i;
  for (i = orderedList.begin() ; i != orderedList.end() ; i++)
    (*i) -> Draw();
}

bool Circle :: Precedes(const Shape& s) const {
  if (dynamic_cast<Square*>(s))
    return true
  else 
    return false
}

```
> 사용자는 모든 Circle이 Square 앞에 그려지도록 요청했다. 그 요구사항에 대한 해결.. 하지만 Precedes함수는 OCP를 따르지 않는다. Shape의 새로운 파생 클래스에 대해 닫을 수 있지 않다.

## 폐쇄를 위해 '데이터 주도적' 접근 방식 사용하기
```CPP

class Shape {
  public : virtual void Draw() const = 0;
    bool Precedes(const Shape&) const

    bool operator<(const Shape& s)> const {
      return Precedes(s);
    }
  private:
    static const char* typeOrderTable[];
}

const char* Shape :: typeOrderTable[] = {
  typeid(Circle).name(), typeid(Square).name(), 0
}

// 이 함수는 테이블에서 클래스 이름을 찾는다
// 테이블은 그려질 도형의 순서를 정의한다.
// 발견되지 않은 도형은 언제나 발견된 도형에 우선한다. 

bool Shape :: Precedes(const Shape& s) const {
  const char* thisType = typeid(*this).name();
  const char* argType = typeid(s).name();
  bool done = false
  int thisOrd = -1;
  int argOrd = -1;
  for (int i = 0 ; !done ; i++) {
    const char* tableEntry = typeOrderTable[i];
    if (tableEntry != 0) {
      if (strcmp(tableEntry, thisType) == 0)
        thisOrd = i;
      if (strcmp(tableEntry, argType) == 0)
        argOrd = i;
      if ((argOrd > 0) && (thisOrd >= 0))
        done = true
    } else // 테이블 항복 == 0
      done = true 
  }
  return thisOrd < argOrd;
}

```
> 요렇게 해결  
  
다양한 Shape의 순서에 대해 닫히지 않은 유일한 항목은 테이블 자체뿐이다. 이 테이블은 다른 모든 모듈에서 분리되어 고유한 모듈에 위치할 수 있으므로, 이것에 대한 변경은 다른 모든 모듈에 아무런 영향을 주지 않느다.

## 결론
OCP는 유연성 재사용성 유지보수성등을 향상시킨다.  
하지만 어설픈 추상화를 피하는 일은 추상화 자체만큼이나 중요하니 주의해야함.


















