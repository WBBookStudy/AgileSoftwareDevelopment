# 13. 커맨드와 액티브 오브젝트 패턴

```Java
public interface Command {
  public void do();
}
```
> 커멘드 패턴
  
## 단순한 커맨드 적용
![KakaoTalk_Photo_2024-02-25-15-32-37](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/5b0ac1da-06fb-4823-b500-0f4bcd13a580)
> RelayOnCommand에서 do()를 호출하면 릴레이가 켜진다.  
  
이 시스템은 이벤트 주도적임. 이벤트 대부분은 센서에 의해 탐지된다.
![KakaoTalk_Photo_2024-02-25-15-35-00](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/374a41e6-36e9-43c3-9c30-dab70868dc85)
  
이 단순한 구조는 장점이 있음. Sensor는 자신이 하는 일을 모른다. 그냥 Command에서 do()를 호출할 뿐..  
커맨드 패턴은 명령의 개념을 캡슐화함으로써 연결된 장치에서 시스템의 논리적인 상호 연결을 분리해낼 수 있게 했다.  

## 트랜잭션
![KakaoTalk_Photo_2024-02-25-15-37-10](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/8f16cdac-4c00-46cf-a3f1-b480393bdc0a)

사용자가 새 직원을 추가하려고 한다면 그 사용자는 성공적으로 직원 레코드를 생성하는 데 필요한 모든 정보를 넣어줘야 한다.  
시스템은 그 정보가 문법적, 의미적으로 옳은지 검증해야 한다.  
커맨드 패턴이 이 일을 도와줄 수 있다.  

![KakaoTalk_Photo_2024-02-25-15-39-08](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/de55aada-a136-4dfd-8abb-383c06830f8c)
> AddEmployeeTransaction은 Employee가 포함하고 있는 것과 똑같은 데이터 필드를 갖고있다.    
> validate 메소드는 모든 데이터를 살펴보고 그것이 이치에 맞는지 확인한다. excute메소드는 검증된 데이터를 사용해 DB를 갱신한다.  
> PayClassification객체는 Employee 객체 내부로 옮겨지거나 복사됨

## 물리적, 시간적 분리
이렇게 사용하면 사용자에게서 데이터를 받는 코드와 그 데이터를 검증하고 그것으로 작업을 하는 코드, 그리고 업무 객체 그 자체를 극적으로 분리 할 수 있다.  
위의 프로그램이 GUI가 있는 프로그램이라면 검증과 실행코드를 따로 떼어 AddEmployeeTransaction클래스에 넣으면 GUI와 물리적으로 분리할 수 있음.

## 시간적 분리
데이터를 받으면 검증과 실행 메소드를 즉시 호출해야 할 이유는 없다. 저장부터 하고, 나중에 검증 할 수 있다.  
커멘드 패턴은 이것을 가능하게 해줌

## 되돌리기
![KakaoTalk_Photo_2024-02-25-15-48-00](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/c314aad6-dfc3-4676-9d42-a7e4a7835b4d)
> do()메소드가 수행하는 동작을 구체적으로 기억할 수 있도록 하면 Undo()도 만들 수 있음
  
## 액티브 오브젝트 패턴
다중 제어 스레드를 구현하는 패턴임.

```Java
import java.util.LinkedList;
import java.util.Iterator;

public class ActiveObjectEngine
{
  LinkedList itsCommands = new LinkedList();

  public void addCommand(Command c)
  {
    itsCommands.add(c);
  }

  public void run() throws Exception
  {
    while (!itsCommands.isEmpty())
    {
      Command c = (Command) itsCommands.getFirst();
      itsCommands.removeFirst();
      c.execute();
    }
  }
}
```

```Java
public interface Command
{
  public void execute() throws Exception;
}
```

```Java
import junit.framework.*;
import junit.swingui.TestRunner;

public class TestSleepCommand extends TestCase
{
  public static void main(String[] args)
  {
    TestRunner.main(new String[]{"TestSleepCommand"});
  }

  public TestSleepCommand(String name)
  {
    super(name);
  }

  private boolean commandExecuted = false;

  public void testSleep() throws Exception
  {
    Command wakeup = new Command()
    {
      public void execute() {commandExecuted = true;}
    };
    ActiveObjectEngine e = new ActiveObjectEngine();
    SleepCommand c = new SleepCommand(1000,e,wakeup);
    e.addCommand(c);
    long start = System.currentTimeMillis();
    e.run();
    long stop = System.currentTimeMillis();
    long sleepTime = (stop-start);
    assert("SleepTime " + sleepTime + " expected > 1000", sleepTime > 1000);
    assert("SleepTime " + sleepTime + " expected < 1100", sleepTime < 1100);
    assert("Command Executed", commandExecuted);
  }
}
```

```Java
public class SleepCommand implements Command
{
  private Command wakeupCommand = null;
  private ActiveObjectEngine engine = null;
  private long sleepTime = 0;
  private long startTime = 0;
  private boolean started = false;

  public SleepCommand(long milliseconds, ActiveObjectEngine e, Command wakeupCommand)
  {
    sleepTime = milliseconds;
    engine = e;
    this.wakeupCommand = wakeupCommand;
  }

  public void execute() throws Exception
  {
    long currentTime = System.currentTimeMillis();
    if (!started)
    {
      started = true;
      startTime = currentTime;
      engine.addCommand(this);
    }
    else if ((currentTime - startTime) < sleepTime)
    {
      engine.addCommand(this);
    }
    else
    {
      engine.addCommand(wakeupCommand);
    }
  }
}
```
SleepCommand가 일정 밀리초만큼 기다렸다가 wakeup을 실행  
  
이 프러그램은 멀티스레드 프로그램 사이에서 유사성을 비교할 수 있다.  
기다리고 있는 이벤트인 ((currentTime - startTime) < sleepTime)이 아직 일어나지 않았다면 그냥 ActiveObjectEngine에 자신을 집어 넣는다.  

RTC(run-to-completion) 테스크. 

```Java
public class DelayedTyper implements Command
{
  private long itsDelay;
  private char itsChar;
  private static ActiveObjectEngine engine = new ActiveObjectEngine();
  private static boolean stop = false;

  public static void main(String args[]) throws Exception
  {
    engine.addCommand(new DelayedTyper(100,'1'));
    engine.addCommand(new DelayedTyper(300,'3'));
    engine.addCommand(new DelayedTyper(500,'5'));
    engine.addCommand(new DelayedTyper(700,'7'));

    Command stopCommand = new Command()
    {
      public void execute() {stop=true;}
    };

    engine.addCommand(new SleepCommand(20000,engine,stopCommand));
    engine.run();
  }

  public DelayedTyper(long delay, char c)
  {
    itsDelay = delay;
    itsChar = c;
  }

  public void execute() throws Exception
  {
    System.out.print(itsChar);
    if (!stop)
      delayAndRepeat();
  }

  private void delayAndRepeat() throws CloneNotSupportedException
  {
    engine.addCommand(new SleepCommand(itsDelay,engine,this));
  }

}
```
> SleepCommand를 활용하고 멀티스레드 방식의 행위를 보여주는 프로그램 CPU클록과 실제 시간이 완벽하게 동기화 되지 않기 때문에 실행할 떄마다 달라진다. 





















