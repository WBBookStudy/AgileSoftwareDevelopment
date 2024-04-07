# 17. 널 오브젝트 패턴

```Java
Employee e = DB.getEmployee("Bob");
if (e != null && e.isTimeToPay(today))
  e.pay();
```
> e가 널인지 먼저 체크한 다음 로직을 처리한다.  
> null 체크를 안해서 당한적이 많지..?  
  
null을 반환하는 대신 예외를 발생시키면 오류를 만들 위험을 감소시킬 수 있다.  
하지만 try/catch 블록은 null 검사보다 더 보기 싫을 수 있고, 기존 어플리케이션에서 수정하기가 까다롭다.
  
  
널 오브젝트 패턴을 이용하면 이 문제를 해결 할 수 있다.
![KakaoTalk_Photo_2024-04-07-19-49-39](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/55738c9f-6880-4cd4-bf2d-005bb0015f76)

NullEmployee는 Employee의 모든 메소드가 '아무 일'도 하지 않도록 구현한다.  
아래와 같이 코드가 바뀜
```Java
Employee e = DB.getEmployee("Bob");
if (e.isTimeToPay(today))
  e.pay();
```
> 깔끔함. DB.getEmployee는 항상 Employee의 인스턴스를 반환한다.  
  
```Java
public void testNull() throws Exception {
  Employee e = DB.getEmployee("Bob");
  if (e.isTimeToPay(new Date()))
    fail();
  assertEqulas(Employee.Null, e);
}
```
> isTimeToPay가 false인것을 기대하며, e는 Employee.Null임을 기대한다.  
  
```Java
public class DB {
  public static Employee getEmployee(String name) {
    return Employee.NULL;
  }
}
```
> 테스트용??
  
```Java
import java.util.Date;

public interface Employee {
  public boolean isTimeToPay(Date payDate);

  public void pay();

  public static final Employee NULL = new Employee() {
    public boolean isTimeToPay(Date payDate) {
      return false;
    }

    public void pay() {

    }
  };
}

```
> 없는 직원을 익명 내부 클래스로 만드는 것은 이것의 인스턴스가 오직 하나임을 보장하는 방법.






















