# 22. 급여 관리 사례 연구(2부)
프로그램이 성장하면 작업하는 사람들의 수도 늘어남. 작업하는 사람이 서로 방해하지 않으며 부드럽게 개발을 진행 할 수 있도록 해야한다.  

## 패키지 구조와 표기법
![KakaoTalk_Photo_2024-06-09-15-25-27 001jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/fbfb87b6-cc21-4f5c-83c5-fb737e55fab2)
> 예제  
  
이 클래스들을 패키지로 나눌 때 사용한 기준은? 임의로 나눈것임.. 좋은 생각은 아니었음  
상위 패키지가 수정되면 하위패키지를 테스트, 컴파일 해야 함.  
우선 Transactions패키지에 들어 있는 클래스들은 동일한 변화에 동일하게 폐쇄되어있지 않다.  

## 공통 폐쇄 원칙(CCP)적용하기
![KakaoTalk_Photo_2024-06-09-15-25-28 002jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/c13c959f-1d80-4182-af91-04311832899e)
이 다이어그램에서 급여 관리 애플리케이션의 클래스들은 어떤 변화에 공통적으로 폐쇄되어 있는 것끼리 묶여있다.  

예를들면 PayrollApplication패키지에 PayrollApplication과 TransactionSource 클래스는 PayrollDomain패키지의 Transaction추상클래스에 의존한다.  
DIP를 이런식으로 구현함..  
  
PayrollDomain패키지는 아무것에도 의존하지 않음. 이 패키지가 담고 있는 대부분의 클래스가 추상클래스임...  
일반성과 독립성  
  
PaymentClassification의 파생 클래스 3개를 담고 있는 Classifications패키지에는 파생클래스가 아주 많음. 하지만 이 많은 클래스의 변화는 패키지 내에서만으로 한정되어 있음  
  
의존하는 패키지가 없거나 매우 적은 패키지에는 실행코드가 들어있다. 이 패키지들은 **책임이 없는 패키지**임.  
많은 패키지들이 의존하고 있는 패키지는 **책임이 있는 패키지**이고, 아무에게도 의존하지 않으므로 **독릭적인 패키지**임.  
22-1의 그림에서 아래쪽 세부적인 패키지들이 독립적이면서도 책임도 많다는것을 주목하자. 안좋음.... SAP위반 아키텍처가 세부사항을 제약하는 편이 낫다.

## 재사용 릴리즈 등가 원칙(REP) 적용하기
위의 설계는 재사용할 수 있는 부분이 있나? Classifications, Methods, Schedules, Affiliations는 재사용 할 수 없음  
어떤 패키지에 들어 있는 클래스 하나만 재사용 되는 경우는 극히 드뭄. 어떤 패키지에 들어있는 클래스들은 응집력이 있어야 하기 떄문임.  
재사용 최소 단위는 패키지이다. REP를 지킬 수 있도록 함께 재사용 할 수도 있어야 한다.  
  
22-1은 CRP를 위반한다. 그래서 패키지에서 재사용 가능한 부분만 조각내서 가져와 재사용한다면 관리상의 어려운 문제에 부딛힐것임..  
22-2는 그런 문제가 생기지 않는다.  
22-3처럼 조금 바꿔보자.
![KakaoTalk_Photo_2024-06-09-15-25-28 003jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/6919ffbe-d733-4e14-8da0-7da15d48cef3)
이제 트랜잭션과 트랜잭션이 다루는 요소가 분리된다. 예를들면 MethodTransactions패키지의 클래스들은 Methods 패키지의 클래스들을 다룬다.  
이제 재사용성과 유지보수성은 더욱 개선됨. 하지만 패키지가 5개 늘어났고, 의존 관계 아키텍처가 더 복잡해졌다.  

## 결합과 캡술화
![KakaoTalk_Photo_2024-06-09-15-25-28 004jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/92b92f2e-126a-43d3-88d7-3ae3d87679f4)
> +, -로 클래스가 익스포트 될 수 있는지를 보여준다.  
  
패키지에서 특정한 클래스들을 숨기는 이유는 그 패키지로 들어오는 결헙을 막기 위해서임  
TimeCard와 SalesReceipt를 전용 클래스로 하는 것은 좋은 선택임. 세부 클래스이기때문에 외부에서 의존하지 못하게 만들어야 하기 떄문.
![KakaoTalk_Photo_2024-06-09-15-25-28 005jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/4f625a3b-6a0a-4c9e-bea2-0234f28c9f2d)

## 측정법
통제하지 못하면 관리할 수 없다. 측정하지 못하면 통제할 수 없다.  
측정값을 게산하는 간단한 쉘 스크립트나 파이썬 스크립트를 만들어보자.  
 - 관계 응집도(H): 패키지 응집도의 여러 측면 가운데 하나를 그 패키지 내부 클래스들이 서로 맺는 관계 수의 평균값으로 나타낼 수 있다. 패키지 내부의 클래스 관계의 수를 R, 패키지 안에 들어있는 클래스의 수를 N, 이 측정값은 패키지와 그 패키지 안에 들어있는 클래스들 사이의 관계를 나타냄  
![KakaoTalk_Photo_2024-06-09-15-25-28 006jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/f3213d1c-7241-4821-86e0-e81aa828d799)
 - 들어오는 결합(Ca): 측정 대상인 패키지에 들어있는 클래스 가운데 다른 패키지가 의존하는 클래스의 수를 세어서 계산
 - 나가는 결합(Ce): 측정 대상인 패키지에 들어 있는 클래스가 의존하는 다른 패키지에 속한 클래스의 수를 세어서 계산
 - 추상도(A) 또는 일반성의 정도: 패키지 안에 들어 있는 추상 클래스(또는 인터페이스)의 수와 전체 클래스 수의 비율로 계산. 측정값은 0~1  
![KakaoTalk_Photo_2024-06-09-15-25-28 007jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/f00a10fa-c1fb-4a91-8b70-3acb3ab511f7)
 - 불안정도(I): 나가는 결합의 수와 전체 결합의 수의 비율로 계산한다. 측정값은 0~1  
![KakaoTalk_Photo_2024-06-09-15-25-28 008jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/58991b87-593f-4e04-9156-15a0838ef9d8)
 - 주계열로부터 거리(D): 주계열은 A + I = 1공식으로 그릴 수 있는 이상적인 직선임. 어떤 패키지와 주계열 사이의 거리를 계산하면 됨. 측정값의 범위는 ~.7부터 0까지이며 0에 가까울수록 좋다.  
![KakaoTalk_Photo_2024-06-09-15-25-28 009jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/23f049d6-6ecf-4c1e-833c-5a6daa4fe337)
 - 정규화된 주계열로부터의 거리(D'): 측정값을 [0,1]범위로 정규화한것임. 0은 주계열 위에 위치한 패키지를 나타내고 1은 주계열로부터 가장 먼 패키지를 나타냄  
![KakaoTalk_Photo_2024-06-09-15-25-28 010jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/d20e1b63-b60b-4b00-b4c8-adbd34a1bb9f)

## 측정값을 급여 관리 애플리케이션에 적용하기
는 읽어보시길..

## 객체 팩토리
Classifications와 ClassificationTransactions가 의존도가 높은 이유는 패키지 안에 있는 클래스들이 인스턴스화 되어야하기 때문임.  
하지만 이 패키지 안에서 이렇게 생성된 객체들은 거의 대부분 추상 인터페이스를 통해서만 사용된다. 그러므로 각각의 구체적 객체를 만들 필요를 없애면 결합도가 낮아질것이다.  
팩토리 패턴을 사용하자.  

### TransactionImplementation을 위한 객체 팩토리
![KakaoTalk_Photo_2024-06-09-16-57-13 001jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/f80f2edb-d997-49dd-b005-c3d636c47ad2)
이런 팩토리 객체(인터페이스) 하나 만들어서 생성하면 됨...

### 팩토리 초기화
객체 팩토리를 사용해서 객체를 생상하려면 적절한 구체 팩토리를 가리키도록 초기화가 되어있어야 함.  
메인 프로그램은 이 일을 하기 최적의 장소임.  
결국 메인프로그램은 모든 구체적 패키지에 의존성이 걸려있을것이고, 바뀔때마다 릴리즈를 할것이다.  
![KakaoTalk_Photo_2024-06-09-16-57-13 002jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/35c3cd09-1b3b-4f19-85cc-c3f3d88a153e)

## 응집도의 경계를 다시 고려해보기
![KakaoTalk_Photo_2024-06-09-15-25-29 011jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/850dd51e-5f0a-4d3d-a04e-6f4d43b01561)
너무 복잡해서 이렇게 바꾸기로...  






















