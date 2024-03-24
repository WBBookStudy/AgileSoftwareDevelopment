# 16. 싱글톤과 모노스테이트 패턴
이번 장에서는 단일성을 강제하는 두 패턴을 다루는데, 많은 경우 이 패턴들의 비용은 이들의 표현력이 주는 이익에 비해 상당히 적다.  

## 싱글톤 패턴
싱글톤은 아주 단순한 패턴인데, 클래스에 인스턴스가 하나만 있도록 하고 전역 액세스 지점을 제공하는 디자인 패턴이다.  
```JAVA
import java.lang.reflect.Constructor;
import junit.framwork.TestCase;

public class TestSimpleSingleton extends TestCase {
  public TestSimpleSingleton(String name) {
    super(name);
  }

  public void testCreateSingleton() {
    Singleton s = Singleton.Instance();
    Singleton s2 = Singleton.Instance();
    assertSame(s, s2);
  }

  public void testNoPublicConstructors() throws Exception {
    Class singleton = Class.forName("Singleton");
    Constructor[] constructor = singleton.getConstructor();
    assertEquals("public constructors.", 0, constructors.length);
  }
}
```
위 테스트 케이스는 어떻게 동작하는지를 잘 보여주는데,
1. public static 메서드인 Instance를 통해 Singleton 인스턴스에 접근하고, Instance 메서드는 항상 동일한 참조값을 가진 인스턴스를 반환한다.
2. Singleton 클래스는 public 생성자가 없기에 Instance메서드를 사용하지 않고서는 인스턴스를 생성할 방법이 없다.

### 싱글톤이 주는 이점
- 플랫폼 호환 : 적절한 미들웨어를 사용하면, 싱글톤은 많은 JVM과 컴퓨터에서 적용되어 확장될 수 있다.
- 어떤 클래스에도 적용 가능 : 어떤 클래스든 그냥 생성자를 전용(private)으로 만들고 적절한 정적 함수와 변수를 추가하기만 하면 싱글톤 형태로 바꿀 수 있다.
- 파생을 통해 생성 가능 : 주어진 클래스에서 싱글톤인 서브클래스를 만들 수 있다.
- 게으른 처리(lazy evaluation) : 싱글톤이 사용되지 않는다면, 생성되지도 않는다.

### 싱글톤의 비용
- 소멸(destruction)이 정의되어 있지 않음 : 싱글톤을 없애거나 사용을 중지하는 좋은 방법이 없다. null처리를 통해 필드에 할당된 인스턴스를 없애려고해도 다른 모듈에서 그 기존 인스턴스에 대한 참조값을 유지하고 있을 수 있고, 이어서 인스턴스가 2개가 존재하게 될 수도 있다.
- 상속되지 않음 : 어떤 싱글톤에서 파생된 클래스는 싱글톤이 아니다. 이것이 싱글톤이 되려면 정적 함수와 변수가 추가되어야 한다.
- 효율성 : 싱글톤의 Instance를 가져오는 메서드에서 현재 싱글톤 인스턴스가 존재하는지 null체크를 할 경우, 대부분의 경우 필요없는 if문이 계속하여 호출되게 된다.
- 비투명성 : 싱글톤의 사용자는 Instance 메소드를 실행해야 하기 때문에 자신이 싱글톤을 사용한다는 사실을 안다.

### 동작에 있어서의 싱글톤
사용자가 어떤 웹서버의 보안영역에 로그인할 수 있게 해주는 웹 기반의 시스템이 있다고 가정하자.  
이 시스템은 사용자의 이름, 패스워드, 그밖의 사용자 속성을 포함하는 데이터베이스를 갖고, 더 나아가 데이터베이스에 서드파티 API를 통해 접근한다고 가정하자.  
이것은 코드 전체에 서드파티 API 사용을 확산시키게 되고, 이는 접근이나 구조에 대한 규정을 강제할 수 있는 여지가 없게 만든다.  
좋은 해결책은 퍼사드 패턴을 사용하여 User객체를 읽고 쓰는 메소드들을 제공하는 UserDatabase 클래스를 만드는 것이다.  
UserDatabase는 User객체와 데이터베이스의 테이블과 행 사이에 변환 작업을 하고, UserDatabase 내에서 구조와 접근의 규정을 강제할 수 있다.  
```JAVA
public interface UserDatabase {
  User readUser(String userName);
  void writeUser(User user);
}
```

```JAVA
public class UserDatabaseSource implements UserDatabase {
  private static UserDatabase theInstance = new UserDatabaseSource();

  public static UserDatabase instance() {
    return theInstanse;
  }

  private UserDatabaseSource() {
  }

  public User readUser(String userName) {
    //구현부분
    return null;  //그저 컴파일되게 하기 위해
  }

  public void writeUser(User user) {  //구현 부분
  }
}
```
위 코드들은 싱글톤식의 해결책을 보여준다.  
싱글톤 클래스의 이름은 UserDatabaseSource이고 이 클래스는 UserDatabase 인터페이스를 구현한다.  
이것은 싱글톤 패턴의 아주 흔한 사용 형태로서, 모든 데이터베이스 접근이 UserDatabaseSource의 단일 인스턴스를 통해 이루어짐을 보장해준다.  
이는 검사, 카운터, 락 등을 UserDatabaseSource에 넣어 접근과 구조에 대한 규정을 강제하기 쉬워진다.  

## 모노스테이트 패턴
모노스테이트 패턴은 단일성을 이루기 위한 또 다른 방법으로, 싱글톤 패턴과는 완전히 다른 매커니즘이다.  
간단하게 정적 데이터 멤버를 사용하여 클래스의 모든 인스턴스의 상태를 보유하는 디자인 패턴이다.
```JAVA
import junit.framwork.TestCase;

public class TestMonoState extends TestCase {
  public TestMonoState(String name) {
    super(name);
  }

  public void testInstance() {
    MonoState m = new MonoState();
    for(int x = 0;x < 10;x++) {
      m.setX(x);
      assertSame(x, m.getX());
    }    
  }

  public void testInstancesBehaveAsOne() {
    MonoState m1 = new MonoState();
    MonoState m2 = new MonoState();
    for(int x = 0;x < 10;x++) {
      m1.setX(x);
      assertSame(x, m2.getX());
    }
  }
}
```
첫번째 테스트 함수는 단순히 자신의 x변수가 설정되거나 검색될 수 있는 어떤 객체를 나타낸다.  
그러나 두번째 테스트 케이스는 같은 클래스의 두 인스턴스가 하나인 것처럼 동작하는 모습을 보여준다.  
만약 Singleton 클래스를 이 테스트 케이스에 접목하고 모든 new Monostate부분을 Singleton.Instance에 대한 호출로 바꾸더라도, 이 테스트케이스를 통과해야 한다.  
따라서, 이 테스트 케이스는 단일 인스턴스라는 제약을 강제하지 않은 Singleton의 행위를 나타낸다.  
어떻게 2개의 인스턴스가 하나의 객체인것처럼 동작할수 있을까? 당연하게도 이것은 2개의 객체가 같은 변수를 공유함을 의미하고, 이는 모든 변수를 정적으로 만듦으로서 쉽게 이룰 수 있다.  
싱글톤 패턴은 단일성 구조를 강제하고, 둘 이상의 인스턴스가 생성되는 것을 막는다.  
반면 모노스테이트 패턴은 구조적인 제약을 부여하지 않고도 단일성이 있는 행위를 강제한다.  
이 차이를 분명히 보려면 모노스테이트 테스트 케이스가 Singleton클래스에 대해서도 유효하지만, 싱글톤 테스트 케이스는 Monostate 클래스에 유효하지 않다.


### 모노스테이트가 주는 이점
- 투명성 : 사용자는 이 객체가 모노스테이트임을 알 필요가 없다.(일반 객체와 동일하게 사용)
- 파생 가능성 : 모노스테이트의 파생클래스는 같은 모노스테이트의 일부가 된다. 이들은 모두 같은 정적 변수를 공유한다.
- 다형성 : 모노스테이트의 메소드는 정적이 아니기 때문에, 파생 클래스에서 오버라이드 될 수 있다.
- 잘 정의된 생성과 소멸 : 정적인 모노스테이트의 변수는 생성과 소멸 시기가 잘 정의되어 있다.

### 모노스테이트의 비용
- 변환 불가 : 보통 클래스는 파생을 통해 모노스테이트로 변환될 수 없다.
- 효율성 : 하나의 모노스테이트느 실제 객체이기 떄문에 많은 생성과 소멸을 겪을 수 있다.
- 실재함 : 모노스테이트의 변수는 이 모노스테이트가 사용되지 않더라도 공간을 차지한다.
- 플랫폼 한정 : 한 모노스테이트가 여러개의 JVM 인스턴스나 여러개의 플랫폼에서 동작하게 만들 수 없다.

### 동작에 있어서의 모노스테이트
지하철 개찰구를 위한 간단한 유한 상태 기계를 구현하는 시스템이 있다고 가정하자.  
![KakaoTalk_20240324_150528512](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/999b7040-efdf-4c03-9b5a-c7ed69bc66b3)
이 개찰구는 Locked상태에서 생명주기를 시작하는데, 동전 하나가 들어오면, Unlocked 상태로 이전하고 출입구를 열고, 울리고 있는 중일수도 있는 경보 상태를 리셋하고, 동전을 요금통에 넣는다.  
어떤 사용자가 이 시점에 출입구를 지나가면, 개찰구는 Locked 상태로 다시 돌아가고 출입구를 닫는다.  
여기에 두가지의 비정상적인 상황이 있는데,  
1. 사용자가 출입구를 지나가기전에 2개 이상의 동전을 넣으면 그 동전들은 환불되고 출입구는 열린 상태로 있게된다.
2. 사용자가 돈을 지불하지 않고 출입구를 지나가면, 경보가 울리고 출입구는 잠긴 상태로 있게된다.

이런 일들을 나타내는 테스트 프로그램은 다음과 같다.
```JAVA
import junit.framwork.TestCase;

public class TestTurnstile extends TestCase {
  public TestTurnstile(String name) {
    super(name);
  }

  public void setUp() {
    Turnstile t = new Turnstile();
    t.reset();
  }

  public void testInit() {
    Turnstile t = new Turnstile();
    assertTrue(t.locked());
    assertTrue(!t.alarm());
  }

  public void testCoin() {
    Turnstile t = new Turnstile();
    t.coin();
    Turnstile t1 = new Turnstile();
    assertTrue(!t1.locked());
    assertTrue(!t1.alarm());
    assertEquals(1, t1.coins());
  }

  ... //기타 구현...
}
```
이 테스트케이스의 기반 Turnstile클래스는 2개의 이벤트함수(coin과 pass)를 유한 상태 기꼐의 상태를 표현하는 Turnstile의 두 파생클래스(Locked와 Unlocked)에 위임한다.  
```JAVA
public class Turnstile  {
  private static boolean isLocked = true;
  private static booldean isAlarming = false;
  private static int itsCoins = 0;
  private static int itsRefunds = 0;
  protected final static Turnstile LOCKED = new Locked();
  protected final static Turnstile UNLOCKED = new Unlocked();
  protected static Turnstile itsState = LOCKED;

  public void reset() {
    lock(true);
    alarm(false);
    itsCoins = 0;
    itsRefunds = 0;
    itsState = LOCKED;
  }
  // 기타 구현...
}

class Locked extends Turnstile {
	public void coin() {
    state = UNLOCKED;
    lock(false);
    alarm(false);
    deposit();
  }
    
  public void pass() {
    alarm(true);
  }
}

class Unlocked extends Turnstile {
	public void coin() {
    refund();
  }
    
  public void pass() {
    lock(true);
    itsState = LOCKED;
  }
}
```
이 예제는 모노스테이트 패턴의 유용한 기능 몇가지를 보여주는데, 모노스테이트 파생 클래스가 다형적이 된다는것과,  
이 모노스테이트 파생클래스 자신들도 모노스테이트가 된다는 사실을 이용했다.  
또한 이 예제는 어떤 모노스테이트를 일반 클래스로 바꾸는일이 바꾸는일이 얼마나 어려운지 보여주는데, 이 프로그램의 구조는 Turnstile의 모노스테이트적 본질에 강하게 의존하고 있다.  
이 유한상태기계로 2개 이상의 개찰구를 제어하고자 한다면, 이 코드에는 많은 리팩토링이 필요할 것이다.  
이 예제에서 Unlocked과 Locked가 Turnstile로 부터 파생되게 하는것이 '~~ 원칙 위반'처럼 보이겠지만 Turnstile은 모노스테이트이기 때문에, 별도의 인스턴스는 존재하지 않는다.  
따라서 Unlocked과 Locked는 실제로 별도의 객체로 보기보다는 Turnstile 추상화의 일부라고 볼 수 있다.  

## 결론
특정 객체는 단일 인스턴스만 생성해야 한다는 제약을 강제해야 할때가 종종 있다.  
싱글톤은 인스턴스 생성을 제어하고 제한하기 위해 private 생성자, 1개 정적 변수, 1개 정적 함수를 사용한다.  
모노스테이트는 그저 객체의 모든 변수를 static으로 만든다.  
싱글톤은 파생을 통해 제어하고 싶은 이미 존재하는 클래스가 있는 경우, 그리고 접근하기 위해 모두가 instance()를 호출해도 상관이 없을 때 좋은 선택이다.  
모노스테이트는 단일 객체의 파생 객체가 다형적이 되게 하고 싶을 때 좋은 선택이다.
