# 23. 컴포지트 패턴
컴포지트 패턴은 아주 단순하지만 지니고 있는 의미는 크다.  
![KakaoTalk_20240616_120331518](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/4220a493-d5e4-46cf-96d5-2437cf420678)
위 그림은 컴포지트 패턴의 기본구조를 보여주는데, Shape 기반 클래스에는 Circle과 Square라는 이름의 파생 도형 2개가 있다.  
그리고 세번째 파생 클래스가 바로 컴포지트 인데, 이 CompositeShape는 여러 Shape 인스턴스들의 목록을 갖고 있다.  
CompositeShape의 draw()가 호출되면 이 클래스는 자신의 목록에 들어있는 모든 Shape 인스턴스들에게 이 메소드 수행을 위임한다.  
따라서 시스템이 보기에는 CompositeShape의 인스턴스는 그냥 일반 Shape 하나로 보인다.  
Shape를 받는 함수나 객체에게 CompositeShape를 전달할 수도 있으며, 하는 행위도 Shape와 똑같아 보이겠지만, 사실 CompositeShape의 실체는 여러 Shape 인스턴스들 집합의 프록시다.  
다음 코드는 CompositeShape 구현의 한가지 예다.  
```JAVA
public interface Shape {
  public void draw();
}
```
```JAVA
import java.awt.Shape;
import java.util.Vector;

public class CompositeShape immplements Shape {
  private Vector itsShapes = new Vector();
  public void add(Shape s) {
    itsShapes.add(s);
  }

  public void draw() {
    for (int i=0;i<itsShapes.size();i++) {
      Shape shape = (Shape)itsShapes.elementAt(i);
      shape.draw();
    }
  }
}
```

## 예제: 컴포지트 커맨드
13장에서 언급했던 Sensors와 Command 객체를 다시한번 생각해보자. Sensor는 무엇을 감지하면 Command의 do()를 호출한다.  
이전에는 Sensor가 Command를 하나 이상 실행해야 하는 경우가 종종 있다는 사실을 언급하지 못했는데, 그런 경우의 예를 들어보자.  
종이가 복사기 내부의 특정 지점에 도달하면 광학 센서가 작동된다. 그러면 이 센서는 특정 모터를 정지시키고 다른 모터를 실행한 다음 특정 클러치를 작동시킨다.  
![KakaoTalk_20240616_132323519_01](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/4ba2031a-031b-4929-bd93-a1540dcf4e71)
처음에는 모든 Sensor 클래스가 Command 객체의 목록을 유지해야 한다고 생각했지만, Sensor가 Command를 하나 이상 실행할 때, 언제나 모든 Command 객체를 동일하게 취급한다는 사실을 깨닫게된다.  
이 말은 Sensor가 그저 목록을 순회하면서 각 Command마다 단지 do()만 호출한다는 뜻이고, 이런 상황은 컴포지트 패턴을 적용하기에 적합하다.  
따라서 Sensor 클래스는 그대로 놓아두고 다음 그림처럼 CompositeCommand 클래스를 만들었다.  
![KakaoTalk_20240616_132323519_02](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/a6c91050-a6f4-4796-947a-d22c5688c224)
이제 Sensor나 Command를 변경할 필요가 없어졌다. 
우리는 두 클래스 모두 하나도 변경하지 않고도 Command에 복수성이라는 개념을 추가할 수 있었고, 이것은 OCP적용의 한 예다.  

## 다수성이냐 아니냐
Sensor와 Command의 연관은 일대일이었는데, 이것을 일대다로 바꾸는 것에 잠시 마음이 끌렸지만 일대다 관계 없이도 일대다의 행위를 할 수 있는 방법을 찾아냈다.  
일대일 관계가 일대다 관계보다 훨씬 이해하기도 쉽고 코딩 및 유지보수도 쉬우므로, 올바른 설계상의 균형처럼 보인다.  
그렇다면 우리가 지금 하고 있는 프로젝트에서 컴포지트를 사용했다면 얼마나 많은 일대다 관계를 일대일 관계로 바꿀수 있었을까??  
물론 컴포지트를 사용한다고 모든 일대다 관계를 일대일로 되돌릴 수 있는것은 아니고, 목록에 들어있는 모든 객체가 동일하게 취급받을때만 이것이 가능하다.  
그래도 충분히 컴포지트로 바꿀 수 있는 일대대 관계도 많이 있으며, 이렇게 바꾸었을 때 얻을 수 있는 이점도 상당하다.  
컴포지트를 사용하면 우리 클래스의 클라이언트마다 목록 관리와 순환 코드가 중복해서 등장하는 대신, 그 코드가 컴포지트 클래스에서만 단 한 번 나타나면 된다.
