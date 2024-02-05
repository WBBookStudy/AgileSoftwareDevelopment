# 12. 인터페이스 분리 원칙(ISP)
이 원칙은 응집력이 없는 인터페이스를 가지는, 일명 '비대한' 인터페이스를 가지는 클래스의 단점을 해결한다. 

## 인터페이스 오염
어떤 보안시스템을 생각해보자.  
이 시스템은 잠기거나 열릴수 있는 Door 객체들이 있고, 이 객체들은 자신이 열리거나 잠겼는지 여부를 알고 있다.  
```CPP
class Door {
  public :
    virtual void Lock() = 0;
    virtual void Unlock() = 0;
    virtual bool IsDoorOpen() = 0;
}
```
이 클래스는 추상 클래스이기에 클라이언트는 Door의 특정한 구현에 의존하지 않고도 Door 인터페이스를 따르는 객체를 사용할 수 있다.  
TimedDoor에 대해 생각해보자. 이건 문이 열린채로 너무 오랜 시간이 지나면 알람을 울려야하고, 이를 위해 Timer라는 다른 객체와 통신한다.  
```CPP
class Timer {
  public : void Register(int timeout, TimeClient* client);
}

class TimerClient {
  public : virtual void TimeOut() = 0;
}
```
제한시간초과 여부에 대한 정보를 받고 싶은 객체는 Timer의 Register 함수를 호출한다.  
이 함수의 인자는 제한시간과, 시간이 초과되었을 때 호출되는 TimeOut함수를 포함하는 TimeClient 객체에 대한 포인터가 된다.  
어떻게 TimeClient 클래스가 TimedDoor클래스와 통신하여 TimedDoord의 코드에서 제한시간 초과 여부를 통지받게 할 수 있을까?  
![KakaoTalk_20240205_200206807](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/e9d566c7-b43c-474b-b355-cb11426f1762)
위 계층 구조에서는 Door와 TimeDoor가 TimerClient를 상속받게 만든다. 이는 TimerClient가 Timer를 통해 자신을 등록하고 TimeOut메시지를 받을 수 있음을 확실하게 해준다.  
이 해결은 평범하지만 다소의 문제가 있는데, 이 중 심각한 문제는 Door 클래스가 이제 TimerClient에 의존하게 되었다는 점이다.  
Door의 모든 변형 클래스가 타이머 기능을 필요로 하는 것은 아니다.  
타이머 기능을 쓰지않는 Door의 변형 클래스가 만들어진다면, 이 클래스는 TimeOut 메소드의 구현을 퇴화해야 할 것이고, 이것은 잠재적 LSP위반이다.  
그리고 이 변형 클래스를 사용하는 애플리케이션은 TimerClient 클래스를 사용하지 않더라도 이것을 import 해야 할 것이고, 불필요한 복잡성과 중복성의 악취를 풍기며, 유지보수와 재사용성 면에서 문제를 일으킬 수 있다.

## 클라이언트 분리는 인터페이스 분리를 의미한다.
Door와 TimerClient는 완전히 다른 클라이언트가 사용하는 인터페이스를 의미한다.  
Timer는 TimerClient를 사용하고, 문을 조작하는 클래스는 Door를 사용한다. 클라이언트가 분리되어 있기에, 인터페이스도 분리된 상태로 있어야 한다.  
클라이언트가 자신이 사용하는 인터페이스에 영향을 끼치기 때문이다.  

## 인터페이스 분리 원칙(ISP)
> 클라이언트가 자신이 사용하지 않는 메소드에 의존하도록 강제되어서는 안 된다.

클라이언트가 자신이 사용하지 않는 메소드에 의존하도록 강제될 때, 이 클라이언트는 이런 메소드의 변경에 취약하다.  
이것은 모든 클라이언트 간의 의도하지 않은 결합을 불러 일으킨다. 우리는 가능하다면 이런 결합을 막고싶고, 따랏거 인터페이스를 분리하기를 원한다.  

## 클래스 인터페이스와 객체 인터페이스
TimeDoor를 다시 살펴보면, 2개의 개별적인 클라이언트 Timer와 Door의 사용자가 사용하는 2개의 개별적인 인터페이스를 가지는 객체가 있다.  
이 두 인터페이스의 구현은 같은 데이터를 조작하기 떄문에 같은 객체에서 구현되어야 한다. 그러면 ISP를 어떻게 만족시킬 수 있을까?  
객체의 클라이언트는 그 객체의 인터페이스를 통해 객체에 접근할 필요가 없고, 위임이나 그 객체의 기반 클래스를 통해 접근할 수 있다.  

### 위임을 통한 분리
한 해결책은 TimerClient에서 파생된 객체를 생성하고 그것의 일을 TimedDoor에 위임하는 것이다.  
![KakaoTalk_20240205_223739796](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/bdd7362f-c5be-491f-9255-4b6f42155c35)
TimedDoor는 Timer를 사용해서 타이머 사용자 등록을 하려고 할때, DoorTimerAdapter를 생성하고 Timer를 써서 이것을 등록한다.  
Timer가 TimeOut 메시지를 DoorTimerAdapter에 전송하면, DoorTimerAdapter는 그 메시지를 TimedDoor에 보내 위임한다.  
이 방법은 ISP를 따름과 동시에 Door 클라이언트의 Timer에 대한 결합을 방지한다.  
만약 Timer의 변경이 있더라도, Door의 사용자는 아무런 영향을 받지 않을 것이고, TimeDoor는 TimerClient와 똑같은 인터페이스를 가질 필요가 없다.  
```CPP
class TimedDoor : public Door {
  public :
    virtual void DOorTimeOut(int TtimeOutId);
}

class DoorTimerAdapter : public TimerClient {
  public :
    DoorTimerAdapter(TimedDoor& theDoor) : itsTimedDoor(theDoor) { }

    virtual void TimeOut(int timeOutId) {
      itsTimedDoor.DoorTimeOut(timeOutId);
    }

  private :
    TimedDoor& itsTimeDoor;
}
```
DoorTimerAdaptger는 TimerClient 인터페이스를 TimeDoor 인터페이스로 변환할수 있고, 이는 굉장히 범용적인 해결책이다.  
그러나 이 방법도 다소 세련되지 못한데, 타이머 사용자 등록을 하려 할때마다 새 객체를 생성하는 일이 수반된다.  
게다가 위임과정은 아주 작긴 하지만 실행시간과 메모리를 필요로 한다.  

### 다중 상속을 통한 분리
![KakaoTalk_20240205_225646131](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/cc88e0ed-6fc8-4cf2-b3a3-6cf5916478e7)
```CPP
class TimeDoor : public Door, public TimerClient {
  public : virtual void TimeOut(int timeOutId);
}
```
위 계층 구조와 코드는 ISP를 만족시키기 위해 다중 상속이 어떻게 사용될 수 있는지를 보여준다.  
이 모델에서 TimeDoor는 Door와 TimerClient에서 모두 상속을 받는다.  
두 기반 클래스의 클라이언트는 TimeDoor를 사용할 수는 있지만, 둘 다 실제로 TimedDoor클래스에 의존하지는 않기 때문에, 분리된 인터페이스를 통해 같은 객체를 사용하게 된다.  


## ATM 사용자 인터페이스 예

