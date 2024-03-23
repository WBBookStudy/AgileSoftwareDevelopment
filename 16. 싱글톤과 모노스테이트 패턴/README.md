# 16. 싱글톤과 모노스테이트 패턴
이번 장에서는 단일성을 강제하는 두 패턴을 다루는데, 많은 경우 이 패턴들의 비용은 이들의 표현력이 주는 이익에 비해 상당히 적다.  

## 싱글톤 패턴
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
싱글톤은 아주 단순한 패턴이다.  

### 싱글톤이 주는 이점
### 싱글톤의 비용
### 동작에 있어서의 싱글톤

## 모노스테이트 패턴
### 모노스테이트가 주는 이점
### 모노스테이트의 비용
### 동작에 있어서의 모노스테이트
## 결론
