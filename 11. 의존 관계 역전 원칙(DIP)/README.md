# 11. 의존 관계 역전 원칙(DIP)
 - 상위 수준의 모듈은 하위 수준의 모둘에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다.
 - 추상화는 구체적인 사항에 의존해서는 안 된다. 구체적인 사항은 추상화에 의존해야 한다.

## 레이어 나누기
![KakaoTalk_Photo_2024-02-03-22-00-23](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/1bb51821-38a4-4aef-a94e-1cdda3a0e57d)
> Policy 레이어가 아래 Utility레이어의 모든 변화에 민감하다. 
![KakaoTalk_Photo_2024-02-03-22-00-23](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/22633c44-d559-44ef-bc54-d128db158bde)
> 좀 더 적절한 모델. 하위 수준의 레이어는 상위 수준의 인터페이스에 의존한다. Policy 레이어의 Utility레이어 의존성이 없어진다. 

## 소유권의 역전 
하위 수준의 모듈은 상위 수준의 모듈 안에 선언되어 호출되는 인터페이스의 구현을 제공한다.  
예제에서는 PolicyLayer는 MechanismLayer나 UtilityLayer의 어떤 변경에도 영향을 받지 않는다.  
더욱이 Policylayer는 PolicyServiceInterface에 맞는 하위 수준 모듈을 정의하는 어떤 문맥에서든 재사용 될 수 있다.  
-> 좀 더 유연하고, 튼튼하고, 이동이 쉬운 구조를 만들어냈다.

## 추상화에 의존하자
구체 클래스에 의존해서는 안되고 추상클래스나 인터페이스에 의존하자.

 - 어떤 변수도 구체 클래스에 대한 포인터나 참조값을 가져선 안 된다.
 - 어떤 클래스도 구체 클래스에서 파생되어서는 안 된다.
 - 어떤 메소드도 그 기반 클래스에서 구현된 메소드를 오버라이드해서는 안 된다.
  
-> 하지만 어느 것인가는 구체 클래스의 인스턴스를 생성해야 하고, 그런 일을 하는 모듈은 이 클래스에 의존해야 한다.  
구체 클래스가 너무 많이 변경되지 않고, 다른 비슷한 파생 클래스가 만들어지지 않는다면 이것에 의존하는 것은 그리 큰 해가 되지 않는다. ex) String Class  

## 간단한 예
![KakaoTalk_Photo_2024-02-03-22-11-30](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/f7beea8c-523e-427e-a606-12d2f861f1a5)
> 왜 미숙할까?
```Java
public class Button {
  private Lamp itsLamp;
  public void poll() {
    if (/* 어떤 조건*/) 
      itsLamp.turnOn();
  }
}
```
> DIP를 위반한다. 상위 수준 정책은 하위 수준 구현에서 분리되어있지 않다.

## 내재하는 추상화를 찾아서
무엇이 상위 수준 정책인가? 애플리케이션에 내재하는 추상화이자, 구체적인 것이 변경되더라도 바뀌지 않는 진실. 메타포  
![KakaoTalk_Photo_2024-02-03-22-15-09](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/3aa0f72a-f304-4821-9ce6-093b09cd2be3)
> ButtonServer는 Button이 어떤 것을 켜거나 끄기 위해 사용할 수 있는 추상 메소드를 제공하고, Lamp는 ButtonServer인터페이스를 구현한다. Lamp는 의존을 당하는게 아니라 의존을 하게된다. 
  
이제 Button이 ButtonServer 인터페이스를 구현하는 어떤 장치든 제어할 수 있다.  

## 용광로 사례
```CPP
#define THERMOMETER 0x86
#define FURNACE 0x87
#define ENGAGE 1
#define DISENGAGE 0

void regulate(double minTemp, double maxTemp) {
  for (;;) {
    while (in(THERMOMETER) > minTemp)
      wait(1);
    out(FURNACE, ENGAGE);

    while (in(THERMOMETER) < maxTemp) 
      wait(1);
    out(FURNACE, DISENGAGE);
  }
}

```
> 많은 하위 수준의 구체적인 내용으로 어지럽혀져있다.

![KakaoTalk_Photo_2024-02-03-22-49-53](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/6df79e0c-b2f3-4c44-8d9b-f520f946ce73)
```CPP
void Regulate(Thermometer& t, Heater& h, double minTemp, double maxTemp) {
  for (;;) {
    while (t.Read() > minTemp)
      wait(1);
    h.Engage();

    while (t.Read() < maxTemp) 
      wait(1);
    h.Disengage();
  }
}
```
> DIP를 이용해 의존성을 역전시킴. 이 알고리즘은 제대로 재사용 할 수 있다.

## 동적 다형성과 정적 다형성
C++ 얘기라 생략



  


