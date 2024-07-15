# 24. 옵저버패턴: 패턴으로 돌아가기
문제를 해결하기 위해 코드를 리팩토링하다 보면 코드가 어떤 패턴과 비슷한 형태가 됨. 그런 일이 생기면 패턴의 이름이 들어갇록 클래스와 변수의 이름을 바꾸고 변경. 즉 **코드가 패턴으로 돌아가는 것**

## 디지털 시계
![KakaoTalk_Photo_2024-07-14-07-58-05 001jpeg](https://github.com/user-attachments/assets/b1e0965a-27a0-434b-a3b8-399e2cb1488d)
> 이 객체는 시간에 대해 알 고 있다.  
  
우리가 만들고 싶은 것은 데스크탑 컴퓨터 위에 놓여서 시간을 보여주는 디지털 시계임. 코드를 작성해보자.
```Java
public void DisplayTime {
	while (1) {
		int sec = clock.getSeconds();
		int min = clock.getMinuites();
		int hour = clock.getHours();
		showTime(hour, min, sec);
	}
}
```
> 좋은 코드가 아니다. 시각이 바뀔 때마다 표시하기 위해 가능한 모든 CPU사이클을 다 잡아먹게된다.  
  
근본적인 문제 원인은 Clock에서 DisplayTime(이하 Digital Clock)로 자료를 효율적으로 전달하는 방법임.  
Clock에서 얻은 데이터와 DigitalClock에서 사용중인 데이터가 동일한지 테스트해보자.
![KakaoTalk_Photo_2024-07-14-07-58-05 002jpeg](https://github.com/user-attachments/assets/926bf724-cdc3-43ac-af27-b96d92838cb8)
  
그러면 ClockDriver는 어떻게 동작해야 할까? 효율적이라면 TimeSource객체에서 시각이 바뀔 때 ClockDriver가 그것을 감지할 수 있어야함.  
오직 그 때만 시각을 TimeSink객체로 옮겨야한다.  
방법은? 폴링? -> CPU 너무 잡아먹음. 언제 시각이 바뀌었는지 Clock 객체가 ClockDriver에게 말해주는 것이 가장 간단한 방법이다.  
![KakaoTalk_Photo_2024-07-14-07-58-05 003jpeg](https://github.com/user-attachments/assets/96cc0ee9-bbd2-4d90-b9eb-ac9087a4d83d)
> (TimeSource에서 Clock Driver로 가는 <<parameter>> 화살표가 생김 ㅡ,.ㅡ)  
  
```Java
import junit.framework.TestCase;
clsss ClockDriverTest extends TestCase {
	public ClockDriverTest(String name) {
		super(name);
	}

	public void testTimeChange() {
		MockTimeSource source = new MockTimeSource();
		MockTimeSink sink = new MockTimeSink();
		ClockDriver driver = new ClockDriv(source, sink);
		source.setTime(3, 4, 5);
		assertEquals(3, sink.getHours());
		assertEquals(4, sink.getMinuites());
		assertEquals(5, sink.getSeconds());

		source.setTime(7, 8, 9);
		assertEquals(7, sink.getHours());
		assertEquals(8, sink.getMinuites());
		assertEquals(9, sink.getSeconds());
	}

}
```

```Java
public interface TimeSource {
	public void setDriver(ClockDriver driver);
}
```

```Java
public interface TimeSink {
	public void setTime(int hours, int minutes, int seconds);
}
```

```Java
public class ClockDriver {
	private TimeSink itsSink;

	public ClockDriver(TimeSource source, TimeSink sink) {
		source.setDriver(this);
		itsSink = sink;
	}

	public void update(int hours, int minutes, int seconds) {
		itsSink.setTime(hours, minutes, seconds);
	}
}
```

```Java
public class MockTimeSource implements TimeSource {
	private ClockDriver itsDriver;

	public void setTime(int hours, int minutes, int seconds) {
		itsDriver.update(hours, minutes, seconds);
	}

	public void setDriver(ClockDriver driver) {
		itsDriver = driver;
	}
}
```

```Java
public class MockTimeSink implements TimeSink {
	private int itsHours;
	private int itsMinutes;
	private int itsSeconds;

	public int getSeconds() {
		return itsSeconds;
	}

	public int getMinutes() {
		return itsMinutes;
	}

	public int getHours() {
		return itsHours;
	}

	public void setTime(int hours, int minutes, int seconds) {
		itsHours = hours;
		itsMinutes = minutes;
		itsSeconds = seconds;
	}

}
```

#### 정리 시작1
TimeSource인터페이스를 ClockDriver 객체들 말고도 누구든 사용할 수 있게 만들고 싶음.  
그래서 TimeSouece에서 ClockDriver로 가는 의존 관계가 맘에 안듬. 해결해보자.
![KakaoTalk_Photo_2024-07-14-07-58-06 004jpeg](https://github.com/user-attachments/assets/9891f2d9-d80f-4ee2-8263-64d579b31258)
```Java
public interface ClockObserver {
	public void update(int hours, int minutes, int seconds);
}
```

```Java
public class ClockDriver implements ClockObserver {
	private TimeSink itsSink;

	public ClockDriver(TimeSource source, TimeSink sink) {
		source.setObserver(this);
		itsSink = sink;
	}

	public void update(int hours, int minutes, int seconds) {
		itsSink.setTime(hours, minutes, seconds);
	}
}
```

```Java
public interface TimeSource {
	public void setObserver(ClockObserver observer);
}
```

```Java
public class MockTimeSource implements TimeSource {
	private ClockObserver itsObserver;

	public void setTime(int hours, int minutes, int seconds) {
		itsObserver.update(hours, minutes, seconds);
	}

	public void setObserver(ClockObserver observer) {
		itsObserver = observer;
	}
}
```

#### 정리 시작2
이제 여러 개의 TimeSink가 시각을 받을 수 있으면 좋겠다. 예를들면 하나는 디지털 시계, 하나는 약속 시간 알림 이런식..  
그래서 ClockDriver의 생성자가 TimeSouece만 받도록 바꾸고 addTimeSink라는 메소드를 추가해서 원하면 언제라도 TimeSink인스턴스들을 추가할 수 있도록 만들자.  

```Java
import junit.framework.TestCase;
clsss ClockDriverTest extends TestCase {
	public ClockDriverTest(String name) {
		super(name);
	}

	public void testTimeChange() {
		MockTimeSource source = new MockTimeSource();
		MockTimeSink sink = new MockTimeSink();
		// ClockDriver driver = new ClockDriver(source, sink);
		source.setObserver(sink);

		source.setTime(3, 4, 5);
		assertEquals(3, sink.getHours());
		assertEquals(4, sink.getMinuites());
		assertEquals(5, sink.getSeconds());

		source.setTime(7, 8, 9);
		assertEquals(7, sink.getHours());
		assertEquals(8, sink.getMinuites());
		assertEquals(9, sink.getSeconds());
	}

}
```
> TimeSink가 아니라 ClockObservers를 구현해야함.

```Java
public class MockTimeSink implements /*TimeSink*/ ClockObserver {
	private func itsHours;
	private func itsMinutes;
	private func itsSeconds;

	public int getSeconds() {
		return itsSeconds;
	}

	public int getMinutes() {
		return itsMinutes;
	}

	public int getHours() {
		return itsHours;
	}

	// public void setTime(int hours, int minutes, int seconds) {
	// 	itsHours = hours;
	// 	itsMinutes = minutes;
	// 	itsSeconds = seconds;
	// }

	public void update()(int hours, int minutes, int seconds) {
		itsHours = hours;
		itsMinutes = minutes;
		itsSeconds = seconds;
	}

}
```
![KakaoTalk_Photo_2024-07-14-07-58-06 005jpeg](https://github.com/user-attachments/assets/a7ac2dad-9f35-45ff-b5e6-8661d9fbd591)

이렇게 수정되면 테스트도 바뀌어야한다. 

```Java
import junit.framework.TestCase;
clsss ClockDriverTest extends TestCase {

	private MockTimeSource = source;
	private MockTimeSink = sink;

	public ClockDriverTest(String name) {
		super(name);
	}

	public void setUp() {
		source = new MockTimeSource();
		sink = new MockTimeSink();
		source.registerObserver(sink);
	}

	private void assertSinkEquals(MockTimeSink sink, int hours, int minutes, int seconds) {
		assertEquals(hours, sink.getHours());
		assertEquals(minutes, sink.getMinuites());
		assertEquals(seconds, sink.getSeconds());
	}

	public void testTimeChange() {
		// MockTimeSource source = new MockTimeSource();
		// MockTimeSink sink = new MockTimeSink();
		// source.setObserver(sink);

		source.setTime(3, 4, 5);
		// assertEquals(3, sink.getHours());
		// assertEquals(4, sink.getMinuites());
		// assertEquals(5, sink.getSeconds());
		assertSinkEquals(sink, 3, 4, 5);

		source.setTime(7, 8, 9);
		// assertEquals(7, sink.getHours());
		// assertEquals(8, sink.getMinuites());
		// assertEquals(9, sink.getSeconds());
		assertSinkEquals(sink, 7, 8, 9);
	}

	public void testMultipleSinks() {
		MockTimeSink sink2 = new MockTimeSink();
		source.registerObserver(sink2);

		source.setTime(12, 13, 14);
		assertSinkEquals(sink, 12, 13, 14);
		assertSinkEquals(sink2, 12, 13, 14);
	}

}
```

#### 정리 시작3

여러 TimeSink를 지원하도록 변경해보자. 등록된 모든 옵저버를 Vector에 담고 있도록 변경하고, 데이터가 변경되면 Vector를 순회해서 데이터 스트림을 날린다.

```Java
public interface TimeSource {
	// public void setObserver(ClockObserver observer);
	public void registerObserver(ClockObserver observer);
}
```

```Java
public class MockTimeSource implements TimeSource {
	// private ClockObserver itsObserver;
	private Vector itsObservers = new Vector();

	public void setTime(int hours, int minutes, int seconds) {
		// itsObserver.update(hours, minutes, seconds);
		Iterator i = itsObservers.Iterator();
		while (i.hasNext()) {
			ClockObserver observer = (ClockObserver) i.next();
			observer.update(hours, minutes, seconds);
		}
	}

	public void /*setObserver*/registerObserver(ClockObserver observer) {
		// itsObserver = observer;
		itsObservers.add(observer);
	}
}
```
![KakaoTalk_Photo_2024-07-14-07-58-06 005jpeg](https://github.com/user-attachments/assets/12e12b2d-1297-4c2e-bf55-395928ca659b)

MockTimeSource가 등록/갱신 작업을 해야 한다는 점이 마음에 안듬. 

```Java
public class /*Mock*/TimeSource /*implements TimeSource*/ {
	private Vector itsObservers = new Vector();

	/*public*/ protected void /*setTime*/notify(int hours, int minutes, int seconds) {
		Iterator i = itsObservers.Iterator();
		while (i.hasNext()) {
			ClockObserver observer = (ClockObserver) i.next();
			observer.update(hours, minutes, seconds);
		}
	}

	public void registerObserver(ClockObserver observer) {
		itsObservers.add(observer);
	}
}
```
```Java

public class MockTimeSource implements TimeSource {
	// private Vector itsObservers = new Vector();

	public void setTime(int hours, int minutes, int seconds) {
		// Iterator i = itsObservers.Iterator();
		// while (i.hasNext()) {
		// 	ClockObserver observer = (ClockObserver) i.next();
		// 	observer.update(hours, minutes, seconds);
		// }
		notify(hours, minutes, seconds);
	}

	// public void registerObserver(ClockObserver observer) {
	// 	itsObservers.add(observer);
	// }
}
```
![KakaoTalk_Photo_2024-07-14-07-58-06 007jpeg](https://github.com/user-attachments/assets/81acf675-3cd1-428a-9cc2-52d3540d74fb)
> 이제 무엇이라도 TimeSource에서 파생될 수 있다. 옵저버들은 갱신하기 위해 notify를 호출하는 것밖에 없다. 


#### 정리 시작4
MockTimeSource를 보면 TimeSource에서 직접 상속을 받고 있다. 이 말은 Clock도 TimeSource에서 파생되어야 할 것이라는 말임.  
Clock이 옵저버 등록이나 갱신에 의존해야 할 필요는 없다.. Clock은 단지 시간에 대해서만 아는 클래스임  

```Java
public interface TimeSource {
	public void registerObserver(ClockObserver observer);
}
```

```Java
public class TimeSourceImplementation {
	private Vector itsObservers = new Vector();

	public void notify(int hours, int minutes, int seconds) {
		Iterator i = itsObservers.Iterator();
		while (i.hasNext()) {
			ClockObserver observer = (ClockObserver) i.next();
			observer.update(hours, minutes, seconds);
		}
	}

	public void registerObserver(ClockObserver observer) {
		itsObservers.add(observer);
	}
}
```

```Java

public class MockTimeSource implements TimeSource {

	TimeSourceImplementation tsImp = new TimeSourceImplementation();

	public void registerObserver(ClockObserver observer) {
		tsImp.registerObserver(observer);
	}

	public void setTime(int hours, int minutes, int seconds) {
		notify(hours, minutes, seconds);
	}

}
```
![KakaoTalk_Photo_2024-07-14-07-58-06 009jpeg](https://github.com/user-attachments/assets/4dacb9c9-07f1-4dfd-a17c-8bd217a2f98f)
> MockTimeSource의 registerObserver에 대한 모든 호출이 TimeSourceImplementation로 위임됨  
> 이제 MockTimeSource가 어떤 클래스로부터도 상속받지 않는다.  
> 이러면 Clock이 등록과 갱신같은 일에 의존하는 문제를 피할 수 있다. 

![KakaoTalk_Photo_2024-07-14-07-58-07 010jpeg](https://github.com/user-attachments/assets/044380f1-44da-497b-89b1-7742b19a444c)


#### 정리 시작5
TimeSource의 클래스의 이름이 적절하지 않아졌다. 등록 및 갱신과 관계 있는 이름(옵저버 패턴에서는 Subject라고 함)으로 바꾸자.  

![KakaoTalk_Photo_2024-07-14-07-58-07 011jpeg](https://github.com/user-attachments/assets/6ad2d9fc-40d6-464b-959f-91b2e3fab7d0)

```Java
public class ObserverTest {
    private MockTimeSource source;
    private MockTimeSink sink;

    @Before
    public void setUp() throws Exception {
        source = new MockTimeSource();
        sink = new MockTimeSink(source);
        source.registerObserver(sink);
    }

    @Test
    public void testTimeChange() throws Exception {
        source.setTime(3, 4, 5);
        assertSinkEquals(sink, 3, 4, 5);

        source.setTime(7, 8, 9);
        assertSinkEquals(sink, 7, 8, 9);
    }

    private void assertSinkEquals(MockTimeSink sink, int hours, int minutes, int seconds) {
        assertThat(sink.getHours(), is(hours));
        assertThat(sink.getMinutes(), is(minutes));
        assertThat(sink.getSeconds(), is(seconds));
    }

    @Test
    public void testMultipleSinks() throws Exception {
        MockTimeSink sink2 = new MockTimeSink(source);
        source.registerObserver(sink2);

        source.setTime(3, 4, 5);
        assertSinkEquals(sink, 3, 4, 5);
        assertSinkEquals(sink2, 3, 4, 5);

    }
}
```
```Java
public interface Observer {
	public void update();
}
```

```Java
public class Subject {
	private Vector itsObservers = new Vector();

	// public void notify(int hours, int minutes, int seconds) {
	// 	Iterator i = itsObservers.Iterator();
	// 	while (i.hasNext()) {
	// 		ClockObserver observer = (ClockObserver) i.next();
	// 		observer.update(hours, minutes, seconds);
	// 	}
	// }
	protected void notifyObservers() {
		Iterator i = itsObservers.Iterator();
		while (i.hasNext()) {
			ClockObserver observer = (ClockObserver) i.next();
			observer.update();
		}
	}

	public void registerObserver(ClockObserver observer) {
		itsObservers.add(observer);
	}
}
```

```Java
public interface TimeSource {
	// public void registerObserver(ClockObserver observer);
	public int getHours();
	public int getMinutes();
	public int getSeconds();
}
```

```Java
public class MockTimeSource extends Subject implements TimeSource {
	private int itsHours;
	private int itsMinutes;
	private int itsSeconds;

	public void setTime(int hours, int minutes, int seconds) {
		itsHours = hours;
		itsMinutes = minutes;
		itsSeconds = seconds;
		notifyObservers();
	}

	public int getHours() {
		return itsHours;
	}

	public int getMinutes() {
		return itsMinutes;
	}

	public int getSeconds() {
		return itsSeconds;
	}

}
```

```Java
public class MockTimeSink implements TimeSink {
	private int itsHours;
	private int itsMinutes;
	private int itsSeconds;
	private TimeSource itsSource;

	public MockTimeSink(TimeSource source) {
		itsSource = source;
	}

	public int getSeconds() {
		return itsSeconds;
	}

	public int getMinutes() {
		return itsMinutes;
	}

	public int getHours() {
		return itsHours;
	}

	// public void setTime(int hours, int minutes, int seconds) {
	// 	itsHours = hours;
	// 	itsMinutes = minutes;
	// 	itsSeconds = seconds;
	// }

	public void update() {
		itsHours = itsSource.getHours();
		itsMinutes = itsSource.getMinuites();
		itsSeconds = itsSource.getSeconds();
	}

}
```












