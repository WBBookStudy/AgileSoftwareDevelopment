# 12. 인터페이스 분리 원칙(ISP)
이 원칙은 응집력이 없는 인터페이스를 가지는, 일명 '비대한' 인터페이스를 가지는 클래스의 단점을 해결한다. 

## 인터페이스 오염
어떤 보안시스템을 생각해보자. 이 시스템은 잠기거나 열릴수 있는 Door 객체들이 있고, 이 객체들은 자신이 열리거나 잠겼는지 여부를 알고 있다.  
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
이 해결은 평범하지만 문제가 없는것은 아닌데, 이 중 심각한 문제는 Door 클래스가 이제 TimerClient에 의존하게 되었다는 점이다.  
Door의 모든 변형 클래스가 타이머 기능을 필요로 하는 것은 아니다. 타이머 기능을 쓰지않는 Door의 변형 클래스가 만들어진다면, 이 클래스는 TimeOut 메소드의 구현을 퇴화해야 할 것이다.  
이것은 잠재적 LSP위반이다. 그리고 이 변형 클래스를 사용하는 애플리케이션은 TimerClient 클래스를 사용하지 않더라도 이것을 import 해야 할 것이고,  
불필요한 복잡성과 중복성의 악취를 풍기며, 유지보수와 재사용성 면에서 문제를 일으킬 수 있다.

## 클라이언트 분리는 인터페이스 분리를 의미한다.
