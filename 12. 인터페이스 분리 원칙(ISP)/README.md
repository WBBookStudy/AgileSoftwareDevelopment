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
ATM의 UI는 매우 유연해야한다. 출력은 다양한 언어 및 스크린이나 점자판 음성합성기로 출력될 수 있어야 한다.  
이런 일은 인터페이스가 표시하는 모든 다양한 메시지를 위한 추상메소드를 갖는 추상기반 클래스를 만듦으로서 해낼 수 있다.  
![KakaoTalk_20240211_105631029_01](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/d5e8aa94-2c4b-41f6-b647-2dd4e423436e)
ATM이 수행할 수 있는 각각의 트랜잭션이 Transaction클래스의 파생 클래스로 캡슐화된다고 하고,  
DepositTransaction, WithdrawalTransaction, TransferTransaction 같은 클래스를 쓸 수 있다.  
각 클래스는 UI의 메소드를 호출하는데, 예를들어 입금하고 싶은 액수를 입력하라고 요청하려면 DepositTransaction객체가 UI클래스의 RequestDepositAmount 메소드를 호출한다.  
이에 대한 구조는 다음과 같다.  
![KakaoTalk_20240211_105631029_02](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/9d84243f-8d2b-40eb-9672-5e536a61835f)
각 트랜잭션은 다른 클래스에서 사용하지 않는 UI의 메소드를 사용한다.  
이는 Transaction의 파생 클래스 중 하나를 변경하는 일이 UI에서 그에 대응하는 변경을 불러올 가능성을 발생시키고,  
Transaction의 모든 파생 클래스와 UI인터페이스에 의존하는 다른 모든 클래스에 영향을 미치게 되며, **경직성**과 **취약성**을 유발하게 되며,
또한 DepositTransaction, WithdrawalTransaction, TransferTransaction 모두가 한 UI인터페이스에 의존하기에 이것들은 전부 재컴파일 되야 하는데,  
만약 트랜잭션이 개별적인 DLL이나 공유 라이브러리의 구성 요소로 배포되었다면 이런 요소들은 재배포 되어야 한다. 이는 **점착성**을 유발하게된다.  
이런 결합들은 UI인터페이스를 DepositUI, WithdrawalUI, TransferUI 같은 개별적인 인터페이스로 분리함으로서 피할 수 있고,  
이런 개별적인 인터페이스는 최종 UI 인터페이스가 다중 상속 할 수 있다.  
![KakaoTalk_20240211_105631029_03](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/92065e62-24f9-456d-81e9-ee9c53852be5)
하지만 여기에는 또다른 ISP 관련된 문제를 파악 할 수 있다.  
DepositTransaction은 DepositUI에 대해 알아야하며, 나머지도 마찬가지다.  
이는 각 트랜잭션이 자신을 위한 특정 UI 버전에 대해 알고 있어야 한다는 것이다.
```CPP
class DepositUI {
  public :
    virtual void RequestDepositAmount() = 0;
}

class DepositTransaction : public Transaction {
  public :
    DepositTransaction(DepositUI& ui) : itsDepositUI(ui) { }

    virtual void Execute() {
      ...
      itsDepositUI.RequestDepositAmount();
      ...
    }
  private :
    DepositUI& itsDepositUI;
}

class WithdrawalUI {
  public :
    virtual void RequestWithdrawalAmount() = 0;
}

class WithdrawalTransaction : public Transaction {
  public :
    WithdrawalTransaction(WithdrawalUI& ui) : itsWithdrawalUI(ui) { }

    virtual void Execute() {
      ...
      itsWithdrawalUI.RequestWithdrawalAmount();
      ...
    }
  private :
    WithdrawalUI& itsWithdrawalUI;
}

class TransferUI {
  public :
    virtual void RequestTransferAmount() = 0;
}

class TransferTransaction : public Transaction {
  public :
    TransferTransaction(WithdrawalUI& ui) : itsTransferUI(ui) { }

    virtual void Execute() {
      ...
      itsTransferUI.RequestTransferAmount();
      ...
    }
  private :
    TransferUI& itsTransferUI;
}

class UI : public DepositUI, public WithdrawalUI, public TransferUI {
  public :
    virtual void RequestDepositAmount();
    virtual void RequestWithdrawalAmount();
    virtual void RequestTransferAmount();
}
```
위 코드에서는 각 트랜잭션이 자신의 특정 UI에 대한 참조값으로 생성되게 함으로서 이 문제를 해결했다.  
이런 방식은 쓰기는 편하지만, 각 트랜잭션이 자신의 UI에 대한 참조값 멤버를 포함해야 한다.  
이 문제를 해결하는 또 다른 방법은 전역 상수 묶음을 만드는 것이다.  
전역 변수가 항상 어설픈 설계의 징후는 아니며, 이 경우에 전역변수는 쉬운 접근이라는 분명한 이점을 제공한다.  

### 복합체와 단일체
함수 g가 DeopsitUI와 TransferUI 둘 모두에 접근하고 이 UI를 함수에 넘겨주고 싶다고 하자. 함수의 prototype을 어떻게 써야할까?
```CPP
void g(DepositUI&, TransferUI&);    //1
void g(UI&);                        //2
```
우리는 대체로 2번의 형태(단일)로 쓰고 싶을 것이다.  
1번(복합)의 형태에서 두 인자는 **같은 객체**를 참조하게 될것이고, 복합 형태를 사용한다면 그 호출은 다음과 같은 모양이 될것이다.  
```CPP
g(ui, ui);
```
아무래도 이상해 보이지만, 대개 복합 형태가 단일 형태보다는 바람직하다.  
단일형태는 g가 UI에 include되어 있는 모든 인터페이스에 의존하도록 만들고, WithdrawUI가 바뀔 때 g와 g의 모든 클라이언트가 영향을 받을 수 있다.  
그리고 g의 두 인자가 항상 같은 객체를 참조할 것이라 단정할 수는 없고, 나중에 어떤 이유로 인해 인터페이스 객체가 분리될 수도 있다. 
모든 인터페이스가 하나의 객체로 결합되어 있다는 사실은 g가 알 필요가 없는 정보고, 그러므로 이런 함수에서는 복합 형태가 더 선호된다. 

#### 클라이언트 그룹 만들기
클라이언트는 종종 이들이 호출하는 서비스 메소드에 의해 그룹으로 묶일 수 있다.  
이런 그룹 만들기를 통해 각 클라이언트에 대해서가 아니라 각 그룹에 대해 분리된 인터페이스를 만들 수 있다.  
그러면 각 서비스가 구현해야 하는 인터페이스의 수가 줄어들 뿐만 아니라 그 서비스가 각 클라이언트의 형에 의존하게 되는 일을 방지할 수 있다.  

#### 인터페이스 변경
객체 지향 애플리케이션을 유지보수할 때는 기존 클래스와 컴포넌트의 인터페이스가 종종 변경되는데,  
이런 변경이 큰 영향을 미쳐서 시스템의 재컴파일과 재배포가 필요해질 때가 있다.  
이것을 완화하려면 기존 인터페이스를 변경하는 것이 아니라 기존 객체에 새로운 인터페이스를 추가하면 된다.  
새로운 인터페이스의 메소드에 접근하려는 원래 인터페이스의 클라이언트는 그 인터페이스에 대해 객체에 질의할 수 있다.  

## 결론
비대한 클래스는 클라이언트들 간에 기이하고 해가되는 결합도를 유발한다.  
한 클라이언트가 이 클래스에 변경을 가하면 모든 나머지 클래스가 영향을 받게 되므로, 클라이언트는 자신이 실제 호출하는 메소드에만 의존해야 한다.  
그러려면 이 비대한 클래스의 인터페이스를 클라이언트 고유의 인터페이스 여러개로 분리하고, 이 인터페이스는 자신의 특정한 클라이언트나 클라이언트 그룹이 호출하는 함수만 선언한다.  
그러면 비대한 클래스가 모든 클라이언트 고유의 인터페이스를 상속하고 그것을 구현할 수 있게 된다.  
이렇게 하면 호출하지 않는 메소드에 대한 클라이언트의 의존성을 끊고, 클라이언트가 서로에 대해 독립적이게 만들 수 있다.  
