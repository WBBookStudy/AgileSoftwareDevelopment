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

### 모노스테이트가 주는 이점
### 모노스테이트의 비용
### 동작에 있어서의 모노스테이트
## 결론
