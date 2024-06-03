# 21. 팩토리패턴
의존 관계 역전원칙에 따르면, 구체 클래스에 의존하는 것은 피하고 추상 클래스에 의존하는 것을 선호해야 한다.  
구체 클래스가 쉽게 변경되는 종류일 경우 특히 그렇다.  
구체 클래스가 변경될 가능성이 크면 클수록 그 클래스에 문제가 생길 가능성도 커지지만, 구체 클래스가 쉽게 변경되는 종류의 클래스가 아니라면, 그 클래스에 의존하는 것이 그렇게 걱정거리가 되지는 않는다.  
예를들어 String의 인스턴스를 만드는 것은 String이 조만간 변경될 가능성은 거의 없기에 String에 의존하는 것은 매우 의존하다.  
반면 애플리케이션을 한창 개발하는 중이라면, 매우 변경되기 쉬운 구체 클래스들이 많이 생긴다.  
이 구체 클래스들에 의존하는것은 문제가 있으며, 대부분의 변경에 영향을 받지 않도록 추상 인터페이스에 의존하는 편이 낫다.  
팩토리 패턴을 사용하면 추상 인터페이스에만 의존하면서도 구체적 객체(conccrete objcet)들의 인스턴스를 만들 수 있으므로, 한창 개발하느라 생성할 구체 클래스의 변경이 잦을 때 이 패턴이 큰 도움이 된다.  
![KakaoTalk_20240602_145529958](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/e1d6616b-360e-49c0-a2f8-6ed2b7ab2a56)
위 시나리오를 보자. Shape인터페이스에 의존하는 SomeApp라는 클래스가 있는데, 이 SomeApp은 오직 Shape 인터페이스를 통해서면 여러 Shape 인스턴스들을 사용한다.  
불행하게도, SomeApp은 오직 Square와 Circle의 인스턴스를 직접 생성하기 때문에, Square와 Circle에게 의존하게 된다.  
![KakaoTalk_20240602_150241954](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/578ba9af-6575-4ebf-856b-897b7909a877)
위 그림처럼 팩토리 패턴을 적용하면 이 문제점들을 고칠수 있다.  
ShapeFactory 인터페이스가 등장하는데, 이 인터페이스에는 makeSquare와 makeCircle 두 메소드가 있다.  
makeSquare 메소드는 Square의 인스턴스를 생성하고 makeCircle은 Circle의 인스턴스를 생성하는데, 두 메소드가 생성한 인스턴스를 반환할떄는 모두 Shape 타입으로 해서 반환한다.  
```JAVA
public interface ShapeFactory {
  public Shape makeCircle();
  public Shape makeSquare();
}
```
```JAVA
public class ShapeFactoryImplementation implements ShapeFactory {
  publci Shape makeCircle() {
    return new Circle();
  }

  public Shape makeSquare() {
    return new Square();
  }
}
```
이렇게 하면 구체 클래스에 의존하는 문제점이 완전하게 해결됨을 알수 있다.  
애플리케이션 코드는 더이상 Square와 Circle에 의존하지 않으면서도 이 두 클래스의 인스턴스는 계속 생성할 수 있다.  
애플리케이션은 이 인스턴스들을 Shape 인터페이스를 통해서만 사용하며 Square나 Circle 한곳에만 있는 메소드는 전혀 호출하지 않는다.  
따라서 이제 구체클래스에 의존한느 문제점은 사라진다.  
누군가는 ShapeFactoryImplementation을 구현해야 하겠지만 나머지 사람들은 더이상 Square나 Circle를 생성할 필요가 없다.  
ShapeFactoryImplementation은 main 함수에서 생성되거나 main 함수에 딸린 초기화 함수에서 생성될 것이다.  

## 의존 관계 순환
눈치 빠른 사람은 이 형태의 팩토리 패턴에 문제가 있음을 알아챘을 것이다.  
ShapeFactory 클래스는 Shape의 파생형마다 메소드가 하나씩 있다. 이럴 경우 Shape에 새로운 파생형을 추가하는 일을 어렵게 만들지도 모르는 의존 관계 순환이 생길 수 있다.  
새로운 Shape 파생형을 추가할 때마다 ShapeFactory에 새로운 메소드를 추가해야 하는데, 대부분의 경우 이것은 모든 ShapeFactory 사용자의 ShapeFactory클래스를 재컴파일 해야 하고 재배포해야 한다는 의미가 되어버린다.  
타입 안정성을 조금 희생하면 이 의존 관계 순환을 막을 수 있다.  
ShapeFactory에 Shape 파생형마다 메소드를 하나씩 만드는 대신, String을 받는 make 함수 하나만 만들면 된다.  
```JAVA
public void testCreateCircle() throws Exception {
  Shape s = factory.make("Circle");
  assert(s instanceof Circle);
}
```
이 기법을 사용하려면 ShapeFactoryImplementation이 들어오는 인자를 가지고 어떤 Shape 파생형을 인스턴스화해야 할지 결정하기 위해 연쇄적으로 if/else 문을 사용해야 한다.  
그 예가 아래 코드에 나와있다.  
```JAVA
public interface ShapeFactory {
  publci Shape make(String shapeName) throws Exception;
}
```
```JAVA
publci class ShapeFactoryImplementation implements ShapeFactory {
  public Shape make(String shapeName) throws Exception {
    if(shapeName.equals("Circle"))
      return new Circle();
    else if (shapeName.equals("Square"))
      return new Square();
    else
      throw new Exception("ShapeFactory cannot create" + shapeName);
  }
}
```
Shape 파생형의 이름을 잘못 쓴 호출자가 컴파일 에러 대신 런타임 에러를 받게 되기 떄문에, 위험하다고 볼수도 있을 것이다.  
틀린말은 아니다. 하지만 적절한 수의 단위테스트가 있고, 테스트 주도적 개발을 적용한다면 런타임 에러가 문제가 되기 전에 미리 잡을 수 있을 것이다.  

## 대체할 수 있는 팩토리
팩토리를 사용해서 생기는 가장 큰 장점중 하나는, 어떤 팩토리의 구현을 다른 구현으로 대체할 수 있다는 점이다.  
다양한 데이터베이스 구현에 모두 잘 적응해야 하는 애플리케이션을 하나 예로 들어보자.  
이 애플리케이션에서 사용자가 일반 파일을 사용할 수도 있고 Oracle™ 어댑터를 구매할 수도 있다 가정해보자.  
프록시 패턴을 써서 애플리케이션과 데이터베이스 구현을 분리할 수도 있을 것이다.  
그리고 이 프록시들을 인스턴스화하기 위해 팩토리들을 사용할 수도 있을 것이다.  
![KakaoTalk_20240602_153050515](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/141068b1-4c8f-4080-bd1d-9d8de1ab31c4)
EmployeeFactory에 두가지 구현이 있음을 주목하자.  
하나는 일반 파일을 대상으로 작업하는 프록시들을 만들고, 다른 하나는 Oracle™을 대상으로 작업하는 프록시들을 만든다.  
애플리케이션 자체는 어떤 것이 사용되는지 모르거나, 알더라도 상관하지 않는다는 점에도 주목하자.  

## 테스트 픽스처를 위해 팩토리 사용하기
단위 테스트를 작성할때, 어떤 모듈이 사용하는 다른 모듈들과 분리된 상태에서 테스트하고 싶은 경우가 종종 있다.  
예를들어 데이터베이스를 사용하는 Payroll 애플리케이션이 있다고 가정하자.  
![KakaoTalk_20240602_153657233](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/7924bc2b-465e-4e60-bf10-dbefd332793f)
여기서 DB를 전혀 사용하지 않고 Payroll 모듈의 기능을 테스트해보고 싶을 수 있다.  
이것은 DB의 추상 인터페이스를 사용해서 이룰 수 있다. 추상 인터페이스의 구현 하나는 실제 DB를 사용하고, 다른 하나는 DB의 행위를 흉내낸다.  
그리고 데이터베이스로 들어오는 호출이 올바른지 검사하기 위해 테스트 코드를 작성하면 된다.  
![KakaoTalk_20240602_154106232](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/d5d8bb6f-3933-4421-ac87-0e1597b7877d)
위 구조와 같이 PayrollTest 모듈은 Payroll 모듈에게 호출을 보냄으로써 이 모듈을 테스트한다.  
그리고 PayrollTest는 Payroll이 DB에 보내는 호출을 잡기 위해 Database 인터페이스도 구현하는데, 이러면Payroll이 올바르게 동작하는지 PayrollTest가 보증할 수 있게 된다.  
이렇게 하면 다른 방법으로는 생성하기 어려운 여러 종류의 데이터베이스 에러나 문제도 PayrollTest가 흉내 내볼 수 있다.  
이것은 스푸핑(spoofing)이라는 이름으로도 알려져 있는 기법이다.  
Payroll은 어떻게 자기가 Database로서 사용할 PayrollTest의 인스턴스를 받을 수 있을까?  
Payroll이 직접 PayrollTest를 생성하지는 않을것이다. 하지만 마찬가지로 분명히 Payroll은 어떻게든 자신이 사용할 Database 구현에 대한 참조를 받아야 한다.  
자연스럽게 PayrollTest가 Database 참조를 Payroll에게 전달할 수 있는 경우도 있다.  
그리고 PayrollTest가 Database를 참조하는 전역 변수를 설정해야만 하는 경우도 있다.  
하지만 Payroll이 Database 인스턴스를 꼭 스스로 생성해야 하는 경우도 있다.  
마지막의 경우라면 Payroll에게 다른 팩토리를 넘겨주는 방법으로 Payroll을 속여서 Database의 테스트 버전을 생성하도록 팩토리를 사용할 수 있다.  
![KakaoTalk_20240602_155507873](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/53f88ac6-3cf9-405e-a605-1c7ac5ba6995)
Payroll 모듈은 GdatabaseFactory라는 이름의 전역변수를 통해 팩토리를 얻는다.  
PayrollTest 모듈은 DatabaseFactory를 구현하고 자기 자신의 참조를 GdatabaseFactory에 넣는다.  
Payroll이 Database를 생성하기 위해 팩토리를 사용한다면, PayrollTest가 이 호출을 받아서 자기 자신의 참조를 반환한다.  
이렇게 하면 자기가 PayrollDatabase를 생성했다고 Payroll이 믿게 만들면서도 PayrollTest 모듈이 Payroll 모듈을 완전하게 스푸핑해서 모든 DB호출을 가로챌 수 있다.  

## 팩토리 사용이 얼마나 중요한가?
DIP를 엄격하게 적용하면, 시스템에 들어있는 쉽게 변경되는 종류의 모든 클래스마다 팩토리를 사용해야 한다.  
그뿐 아니라 팩토리 패턴이 지닌 힘은 마음을 끌리게 하는데, 이런 두가지 요인은 때대로 개발자가 기본적으로 팩토리를 사용하도록 부추길수 있다.  
하지만 이건 너무 극단적이기 때문에 그다지 권장하지 않는다.  
보통 팩토리를 사용하지 않고 시작하여, 팩토리의 필요성이 충분히 커지면 그제야 시스템에 도입해도 충분하다.  
팩토리는 피하려면 피할 수 있는 복잡함이다. 특히 지금 진화하고 있는 설계의 초기 단계의 경우라면 더욱 그러하다.  
언제나 팩토리를 사용한다면 설계를 확장하기가 급격히 어려워진다.  

## 결론
팩토리는 강력한 도구이다. DIP를 지키려고 할때 큰 도움이 되기도 하고, 높은 차원의 정책 모듈이 클래스들의 구체적인 구현에 의존하지 않고도 그 클래스들의 인스턴스를 생성하게 해주기도 한다.  
그리고 어떤 클래스 무리를 완전히 다른 클래스 무리로 교체하는 일도 가능하게 만든다.  
하지만 팩토리는 피하려면 피할 수도 있는 복잡함이다. 기본으로 팩토리를 사용하는 것이 최선의 방법인 경우는 드물다.
