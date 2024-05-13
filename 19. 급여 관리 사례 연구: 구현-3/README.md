# 19. 급여 관리 사례 연구: 구현-3
## 직원에게 임금 지급하기
다음 그림은 PaydayTransaction 클래스의 정적 구조를 보여준다.  
![KakaoTalk_20240513_145707311](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/209505c1-ab3b-4c54-a127-1af2cd6c2e8e)

아래 그림들은 이 클래스의 동적 행위를 묘사한다.  
![KakaoTalk_20240513_145707311_01](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/86e26a47-be8a-4709-b9b5-f77ec6ec4556)
![KakaoTalk_20240513_145707311_02](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/07a6e5fd-f69b-414d-885c-347d66067d2e)
CarculatePay 메시지가 사용하는 알고리즘은 Employee 객체가 포함하는 PaymentClassification의 종류에 의존한다.  
특정날짜가 임금지급일인지 결정하는 알고리즘은 Employee 객체가 포함하는 PaymentSchedule 종류에 의존한다.  
Employee에 임금을 지급하는 알고리즘은 PaymentMethod 객체의 형에 의존한다.  
이런 고도의 추상화는 새로운 종류의 임금종류, 지급주기, 조합, 지급방법 추가에 대해 닫혀 있게 해준다.  
19-32와 19-33에 묘사된 알고리즘은 포스팅(posting)의 개념을 소개하고 있다.  
올바른 지금 금액이 계산되어 Employee에 보내진 후 임금 지급은 포스팅, 즉 임금 지급에 관련된 레코드가 갱신된다.  
그러므로 마지막으로 포스팅한 날부터 지정된 날까지의 임금을 계산하는 CalculatePay 메소드를 정의할 수 있다.  

### 경영 관련 의사결정을 해버리는 개발자를 원하는가?
이런 포스팅의 개념은 사용자 스토리나 유스케이스에는 언급되지 않았다.  
단지 알아차린 문제를 해결하는 방법으로 이것을 사용한 것이라고 한다.  
Payday 메소드가 같은 날짜나 같은 지급 주기의 날짜로 여러번 호출될 수 있는는 것을 염려하여 그 직원이 두번 이상 임금을 지급받을 수 없도록 확실히 하기를 원했고, 고객에게 물어본것은 아니라 자진해서 이렇게 하였다고 한다.  
의사결정을 하기 전에 고객이나 프로젝트 관리자에게 물어보았어야 한다. 그들은 전혀 다른 생각을 가지고 있을지도 모르기 때문이다.  
추후 고객에게 확인하는 과정에서 포스팅이란 아이디어가 고객의 의도와는 다르다는 사실을 알게되었다고 한다.  
고객은 급여 시스템을 실행하고 나서 지급 수표들을 확인할 수 있기를 원했고, 이 중 어떤것이라도 잘못되면 지급 정보를 수정하고 다시 급여 프로그램을 실행하기를 원했다.  
당시에는 포스팅이 좋은 생각처럼 보였으나, 고객이 원한것은 아니었고 그래서 계획을 버려야 했다.  

### 월급을 받는 직원에게 임금 지급하기
다음 코드에는 2개의 케이스가 나와있는데, 이 테스트는 월급을 받는 직원이 올바르게 임금을 받고 있는지 확인한다.  
첫번째 테스트 케이스는 직원이 그 달의 마지막 날에 임금을 받는지 확인한다.  
두번째 테스트 케이스는 그달의 마지막 날이 아니라면 직원이 임금을 받지 않는것을 확인한다.  
```Cpp
void PayrollTest :: TestPaySingleSalariedEmployee() {
  cerr << "TestPaySingleSalariedEmployee" << endl;
  int empId = 1;
  AddSalariedEmployee t(empId, "Bob", "Home", 1000.00);
  t.Execute();
  Date payDate(11, 30,  2001);
  PaydayTransaction pt(payDate);
  pt.Execute();
  Paycheck* pc = pt.GetPaycheck(empId);
  assert(pc);
  assertEquals(pc -> GetPayDate() == payDate);
  assert("Hold" == pc -> GetField("Disposition"));
  assertEquals(0.0, pc -> GetDeductions(), .001);
  assertEquals(1000.00, pc -> GetNetPay(), .001);
}

void PayrollTest :: TestPaySingleSalariedEmployeeOnWrongDate() {
  cerr << "TestPaySingleSalariedEmployeeOnWrongDate" << endl;
  int empId = 1;
  AddSalariedEmployee t(empId, "Bob", "Home", 1000.00);
  t.Execute();
  Date payDate(11, 29,  2001);
  PaydayTransaction pt(payDate);
  pt.Execute();
  Paycheck* pc = pt.GetPaycheck(empId);
  assert(pc == 0);
}
```
다음 코드는 PaydayTransaction의 Excute() 함수를 보여준다.  
```Cpp
void PaydayTransaction :: Excute() {
  list<int> empIds;
  GpayrollDatabase.GetAllEmployeeIds(empIds);

  list<int> :: iterator i = empIds.begin();
  for(; i != empIds.end(); i++) {
    int empId = *i;
    if(Employee* e = GpayrollDatabase.GetEmployee(empId)) {
      if(e -> IsPayDate(itsPayDate)) {
        Paycheck* pc = new Paycheck(itsPayDate);
        itsPaychecks[empId] = pc;
        e -> Payday(*pc);
      }
    }
  }
}
```
이 함수는 데이터베이스에 있는 모든 Employee 객체를 순회하고, 각 직원에게 이 트랜잭션의 날짜가 그 직원의 임금 지급 날짜인지 묻는다.  
만약 그렇다면 해당 직원을 위한 새 지급 수표를 만들고 그 직원에게 수표의 필드를 채우라고 요청한다.  
다음은 MonthlySchedule.cpp의 코드중 일부를 보여준다.  
이 코드는 인자로 준 날짜가 그 달의 마지막 날인 경우에만 IsPayDate가 true를 반환하도록 구현되어있다.
```Cpp
namespace {
  bool IsLastDayOfMonth(const Date& date) {
    int m1 = date.GetMonth();
    int m2 = (date + 1).GetMonth();
    return (m1 != m2);
  }
}

bool MonthlySchedule :: IsPayDate(const Date& payDate) const {
  return IsLastDayOfMonth(payDate);
}
```
다음 코드는 Employee::PayDay()의 구현을 보여준다.  
이 함수는 모든 직원의 임금을 계산하고 지급하는 일반적인 알고리즘이다.  
```Cpp
void Employee :: PayDay(Paycheck& pc) {
  double grossPay = itsClassification -> CalculatePay(pc);
  double deductions = itsAffiliation -> CalculateDeductions(pc);
  double netPay = grossPay - deductions;
  pc.SetGrossPay(grossPay);
  pc.SetDeductions(deductions);
  pc.SetNetPay(netPay);
  itsPaymentMethod -> Pay(pc);
}
```
모든 구체적인 계산은 내부에 포함된 스트래터지 클래스인 itsClassification, itsAffiliation, itsPaymentMethod에 맡겨진다.  

### 시간제 직원에게 임금 지급하기
이것은 테스트 우선 설계에서 '점진주의'의 좋은 예다. 점진주의란 아주 평범한 테스트 케이스로부터 시작해 좀 더 복잡한 것으로 나아가는 것을 말한다.  
다음 코드는 가장 단순한 경우를 보여준다.  
한 시간제 직원을 DB에 추가하고 그에게 임금을 지급한다. 타임카드가 없기에 지급 수표금액은 0이 될것이다.  
유틸리티 격의 함수인 ValidateHourlyPaycheck는 나중에 있을 리팩토링을 표현한다.  
```Cpp
void PayrollTest :: TestPaySingleHourlyEmployeeNoTimeCards() {
  cerr << "TestPaySingleHourlyEmployeeNoTimeCards" << endl;
  int empId = 2;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
  t.Execute();
  Date payDate(11, 9, 2001);
  PaydayTransaction pt(payDate);
  pt.Execute();
  ValidateHourlyPaycheck(pt, empId, payDate, 0.0);
}

void PayrollTest :: ValidateHourlyPaycheck(PaydayTransaction& pt, int empid, const Date& payDate, double pay) {
  Paycheck* pc = pt.GetPaychek(empid);
  assert(pc);
  assert(pc -> GetPayDate() == payDate);
  assertEquals(pay, pc -> GetGrossPay(), .001);
  assert("Hold" == pc -> GetField("Disposition"));
  assertEquals(0.0, pc -> GetDeductions(), .001);
  assertEquals(pay, pc -> GetNetPay(), .001);
}
```
다음은 2개의 테스트 케이스를 보여준다.
1. 하나의 타임카드를 추가한 뒤에 직원에게 임금을 지급할 수 있는지를 테스트한다.
2. 8시간 이상이 찍혀있는 카드에 대해 초과 근무 수당을 지급할 수 있는지를 테스트 한다.

```Cpp
void PayrollTest :: TestPaySingleHourlyEmployeeOneTimeCards() {
  cerr << "TestPaySingleHourlyEmployeeOneTimeCards" << endl;
  int empId = 2;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
  t.Execute();
  Date payDate(11, 9, 2001);

  TimeCardTransaction tc(payDate, 2.0, empId);
  tc.Execute();
  PaydayTransaction pt(payDate);
  pt.Execute();
  ValidateHourlyPaycheck(pt, empId, payDate, 30.5);
}

void PayrollTest :: TestPaySingleHourlyEmployeeOvertimeOneTimeCard() {
  cerr << "TestPaySingleHourlyEmployeeOvertimeOneTimeCard" << endl;
  int empId = 2;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
  t.Execute();
  Date payDate(11, 9, 2001);

  TimeCardTransaction tc(payDate, 9.0, empId);
  tc.Execute();
  PaydayTransaction pt(payDate);
  pt.Execute();
  ValidateHourlyPaycheck(pt, empId, payDate, (8 + 1.5) * 15.25);
}
```
다음 테스트 케이스는 PaydayTransaction이 금요일로 생성되지 않으면 시간제 직원에게 임금을 지급할 수 없음을 확인하고 있다.  
```Cpp
void PayrollTest :: TestPaySingleHourlyEmployeeOnWrongDate() {
  cerr << "TestPaySingleHourlyEmployeeOnWrongDate" << endl;
  int empId = 2;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
  t.Execute();
  Date payDate(11, 8, 2001);

  TimeCardTransaction tc(payDate, 9.0, empId);
  tc.Execute();
  PaydayTransaction pt(payDate);
  pt.Execute();

  Paycheck* pc = pt.GetPaycheck(empId);
  assert(pc == 0);
}
```
다음 테스트 케이스는 타임카드가 하나 이상인 직원의 임금을 계산할 수 있음을 확인하는 테스트 케이스다.  
```Cpp
void PayrollTest :: TestPaySingleHourlyEmployeeTwoTimeCards() {
  cerr << "TestPaySingleHourlyEmployeeTwoTimeCards" << endl;
  int empId = 2;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
  t.Execute();
  Date payDate(11, 9, 2001);

  TimeCardTransaction tc(payDate, 2.0, empId);
  tc.Execute();
  TimeCardTransaction tc2(Date(11, 8, 2001), 5.0, empId);
  tc2.Execute();
  PaydayTransaction pt(payDate);
  pt.Execute();
  ValidateHourlyPaycheck(pt, empId, payDate, 7 * 15.25);
}
```
마지막으로 직원이 가진 현재 지급 주기의 타임카드에 대해서만 임금을 지급한다는 사실을 증명한다. 다른 지급 주기의 타임카드는 무시된다.  
```Cpp
void PayrollTest :: TestPaySingleHourlyEmployeeWithTimeCardsSpanningTwoPayPeriods() {
  cerr << "TestPaySingleHourlyEmployeeWithTimeCardsSpanningTwoPayPeriods" << endl;
  int empId = 2;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
  t.Execute();
  Date payDate(11, 9, 2001);
  Date dateInPreviousPayPeriod(11, 2, 2001);

  TimeCardTransaction tc(payDate, 2.0, empId);
  tc.Execute();
  TimeCardTransaction tc2(dateInPreviousPayPeriod, 5.0, empId);
  tc2.Execute();
  PaydayTransaction pt(payDate);
  pt.Execute();
  ValidateHourlyPaycheck(pt, empId, payDate, 2 * 15.25);
}
```
이 모든것이 제대로 동작하는 코드는 한번에 한 테스트 케이스씩, 점진적으로 발전했다.  
다음은 제대로된 HourlyClassification.cpp의 일부분을 보여주는데, 이 코드는 단순히 타임 카드를 반복해서 확인한다.  
즉, 각 타임카드가 현재 지급 주기에 속하는지 확인해서, 그렇다면 그 타임카드가 나타내는 임금을 계산한다.  
```Cpp
double HourlyClassification :: CalculatePay(Paycheck& pc) const {
  double totalPay = 0;
  Date payPeriod = pc.GetPayDate();
  map<Date, TimeCard*> :: const_iterator i;
  for (i = itsTimeCards.begin(); i!= itsTimeCards.end(); i++) {
    TimeCard * tc = (*i).second;
    if (IsInPayPeriod(tc, payPeriod))
      totalPay += CalculatePayForTimeCard(tc);
  }
  return totalPay;
}

bool HourlyClassification :: IsInPayPeriod(TimeCard* tc, const Date& payPeriod) const {
  Date payPeriodEndDate = payPeriod;
  Date payPeriodStartDate = payPeriod - 5;
  Date timeCardDate = tc -> GetDate();
  return (timeCardDate >= payPeriodStartDate) && (timeCardDate <= payPeriodEndDate);
}

double HourlyClassification :: CalculatePayForTimeCard(TimeCard* tc) const {
  double hours = tc -> GetHours();
  double overtime = max(0.0, hours - 8.0);
  double straightTime = hours - overtime;
  return straightTime * itsRate + overtime * itsRate * 1.5;
}
```
다음은 WeeklySchedule이 금요일에만 임금을 지급한다는 것을 보여준다.  
```Cpp
bool WeeklySchedule :: IsPayDate(const Date& theDate) const {
  return theDate.GetDayOfWeek() == Date :: friday;
}
```

### 지급 주기 : 설계 문제
이제 조합비와 공제액을 구현할 때가 되었다. 월급을 받는 직원을 추가하고, 그 직원을 조합원으로 바꾸고,  
그 직원에 임금을 지급한 후 조합비가 임금에서 빠져나갔는지 확인하는 테스트 케이슬을 확인한 후 다음과 같이 코드로 만들었다.  
```Cpp
void PayrollTest :: TestSalariedUnionMemberDues() {
  cerr << "TestSalariedUnionMemberDues" << endl;
  int empId = 1;
  AddSalariedEmployee t(empId, "Bob", "Home", 1000.00);
  t.Execute();
  int memberId = 7734;
  ChangeMemberTransaction cmt(empId, memberId, 9.42);
  cmt.Execute();
  Date payDate(11, 30, 2001);
  PaydayTransaction pt(payDate);
  pt.Execute();
  ValidatePaycheck(pt, empId, payDate, 1000.0 - ???);
}
```
이 테스트 케이스의 마지막 라인의 ???를 주목하자. 무엇이 들어가야 할까??  
사용자 스토리에 따르면 조합비가 주별로 나오지만 월급을 받는 직원은 달별로 임금을 받는다.  
각달에 몇개의 주가 있을까? 이것은 그다지 정확하지 않다. 따라서 고객에게 무엇을 원하는지 물어야 한다.  
다음은 PaydayTransaction::Execute()에 사한 변경을 보여준다. Paycheck이 만들어질때 그 주기의 시작 날짜와 마지막 날짜 모두를 넘겨받는 것에 주목하자.  
```Cpp
void PaydayTransaction :: Execute() {
  list<int> empIds;
  GpayrollDatabase.GetAllEmployeeIds(empIds);

  list<int> :: iterator i = empIds.begin();
  for (; i != empIds.end(); i++) {
    int empId = *i;
    if(Employee* e = GpayrollDatabase.GetEmployee(empId)) {
      if(e -> IsPayDate(itsPayDate)) {
        Paycheck* pc = new Paycheck(e -> GetPayPeriodStartDate(itsPayDate), itsPayDate);
        itsPaychecks[empId] = pc;
        e -> Payday(*pc);
      }
    }
  }
}
```
HourlyClassification과 CommissionedClassification에서 TimeCard와 SalesReceipt가 그 지급 주기 내의 것인지 확인하는 두 함수는 합쳐져서 기반 클래스인 PaymentClassification으로 옮겨졌다.  
```Cpp
bool PaymentClassification :: IsInPayPeriod(const Date& theDate, const Paycheck& pc) const {
  Date payPeriodEndDate = pc.GetPayPeriodEndDate();
  Date payPeriodStartDate = pc.GetPayPeriodStartDate();
  return (theDate >= payPeriodStartDate) && (theDate <= payPeriodEndDate);
}
```
이제 UnionAffilliation::CalculateDeductions에서 직원의 조합비를 계산할 준비가 되었다.  
지급 주기를 정의하는 두 날짜는 급료 지급 수표에서 추출되어 그 사이의 금요일 횟수를 세는 유틸리티 격의 함수에 넘겨진다.  
주당 조합비 비율을 이 숫자에 곱해서 그 지급 주기의 조합비를 계산하게 된다.  
```Cpp
namespace {
  int NumberOfFridaysInPayPeriod(constn Date& payPeriodStart, const Date& payPeriodEnd) {
    int fridays = 0;
    for(Date day = payPeriodStart; day <=payPeriodEnd; day++) {
      if(day.GetDayOfWeek() == Date :: friday)
        fridays++;
    }
    return fridays;
  }
}

double UnionAffiliation :: CalculateDeductions(Paycheck& pc) const {
  double totalDues = 0;

  int fridays = NumberOfFridayInPayPeriod(pc.GetPayPeriodStartDate(), pc.GetPayPeriodEndDate());
  totalDues = itsDues * fridays;
  return totalDues;
}
```
마지막 두 테스트 케이스는 조합 공제액과 관련된 것이어야 한다.  
첫번째 테스트는 공제액이 올바르게 공제되는지를 확인한다.
```Cpp
void PayrollTest :: TestHourlyUnionMemberServiceCharge() {
  cerr << "TestHourlyUnionMemberServiceCharge" << endl;
  int empId = 1;
  AddSalariedEmployee t(empId, "Bill", "Home", 15.24);
  t.Execute();
  int memberId = 7734;
  ChangeMemberTransaction cmt(empId, memberId, 9.42);
  cmt.Execute();
  Date payDate(11, 9, 2001);
  ServiceChargeTransaction sct(memberId, payDate, 19.42);
  sct.Execute();
  TimeCardTransaction tct(payDate, 8.0, empId);
  tct.Execute();
  PaydayTransaction pt(payDate);
  pt.Execute();
  Paycheck* pc = pt.GetPaycheck(empId);
  assert(pc);
  assert(pc -> GetPayPeriodEndDate() == payDate);
  assertEquals(8 * 15.24, pc -> GetGrossPay(), .001);
  assert("Hold" == pc -> GetField("Disposition"));
  assertEquals(9.42 + 19.42, pc -> GetDeductions(), .001);
  assertEquals((8 * 15.24) - (9.42 + 19.42), pc -> GetNetPay(), .001);
}
```
두번째 케이스는 현재 지급 주기 밖의 공제액은 공제되지 않음을 확인한다.  
```Cpp
void PayrollTest :: TestServiceChargesSpanningMultiplePayPeriods() {
  cerr << "TestServiceChargesSpanningMultiplePayPeriods" << endl;
  int empId = 1;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.24);
  t.Execute();
  int memberId = 7734;
  ChangeMemberTransaction cmt(empId, memberId, 9.42);
  cmt.Execute();
  Date earlyDate(11, 2, 2001);  //지난주 금요일
  Date payDate(11, 9, 2001);
  Date lateDate(11, 16, 2001);  //다음주 금요일
  ServiceChargeTransaction sct(memberId, payDate, 19.42);
  sct.Execute();
  ServiceChargeTransaction sctEarly(memberId, earlyDate, 100.00);
  sctEarly.Execute();
  ServiceChargeTransaction sctLate(memberId, lateDate, 200.00);
  sctLate.Execute();
  TimeCardTransaction tct(payDate, 8.0, empId);
  tct.Execute();
  PaydayTransaction pt(payDate);
  pt.Execute();
  Paycheck* pc = pt.GetPaycheck(empId);
  assert(pc);
  assert(pc -> GetPayPeriodEndDate() == payDate);
  assertEquals(8 * 15.24, pc -> GetGrossPay(), .001);
  assert("Hold" == pc -> GetField("Disposition"));
  assertEquals(9.42 + 19.42, pc -> GetDeductions(), .001);
  assertEquals((8 * 15.24) - (9.42 + 19.42), pc -> GetNetPay(), .001);
}
```
이것을 구현하기 위해 UnionAffiliation::CalculateDeductions가 IsInPayPeriod를 호출하게 하고 싶었지만,  
IsInPayPeriod를 PaymentClassification 클래스에 넣어버렸다.  
이 클래스가 이 함수를 호출해야 하는 PaymentClassification의 파생 클래스인 한, 이 함수를 그곳에 넣어두는 것이 편했다.  
하지만 이제 다른 클래스들도 이 함수를 필요로 하게 되어, 이 함수를 Date 클래스로 옮겨, 주어진 날짜가 다른 두 주어진 날짜 사이에 있는지 확인만 한다.  
```Cpp
static bool IsBetween(const Date& theDate, const Date& startDate, const Date& endDate) {
  return (theDate >= startDate) && (theDate <= endDate);
}
```

아래의 두 코드는 Employee 클래스의 구현을 보여준다.
```cpp
#ifndef EMPLOYEE_H
#define EMPLOYEE_H
#include <string>

class PaymentSchedule;
class PaymentClassification;
class paymentMethod;
class Affiliation;
class Paycheck;
class Date;

class Employee {
  public :
    virtual ~Employee();
    Employee(int empid, string name, string address);
    void SetName(string name);
    void SetAddress(string address);
    void SetClassification(PaymentClassification*);
    void SetMethod(PaymentMethod*);
    void SetSchedule(PaymentSchedule*);
    void SetAffiliation(Affiliation*);

    int GetEmpid() const {
      return itsEmpid;
    }
    string GetName() const {
      return itsName;
    }
    string GetAddress() const {
      return itsAddress;
    }
    PaymentMethod* GetMethod() {
      return itsPaymentMethod;
    }
    PaymentClassification* GetClassification() {
      return itsClassification;
    }
    PaymentSchedule* GetSchedule() {
      return itsSchedule;
    }
    Affiliation* GetAffiliation() {
      return itsAffiliation;
    }

    void Payday(paycheck&);
    bool IsPayDate(const Date& payDate) const;
    Date GetPayPeriodStartDate(const Date& payPeriodEndDate) const;

  private:
    int ItsEmpid;
    string itsName;
    string itsAddress;
    PaymentClassification* itsClassification;
    PaymentSchedule* itsSchedule;
    PaymentMethod* itsPaymentMethod;
    Affiliation* itsAffiliation;
};
#endif
```
```Cpp
#include "Employee.h"
#include "NoAffiliation.h"
#include "PaymentClassification.h"
#include "PaymentSchedule.h"
#include "PaymentMethod.h"
#include "Paycheck.h"

Employee :: ~Employee() {
  delete itsClassification;
  delete itsSchedule;
  delete itsPaymentMethod;
}

Employee :: Employee(int empid, string name, string address) : itsEmpid(empid), itsName(name),
  itsAddress(address),
  itsAffiliation(new NoAffiliation()),
  itsClassification(0),
  itsSchedule(0),
  itsPaymentMethod(0) {
}

void Employee :: SetName(string name) {
  itsName = name;
}

void Employee :: SetAddress(string address) {
  itsAddress = address;
}

void Employee :: SetClassification(PaymentClassification* pc) {
  delete itsClassification;
  itsClassification = pc;
}

void Employee :: SetSchedule(PaymentSchedule* ps) {
  delete itsSchedule;
  itsSchedule = ps;
}

void Employee :: SetMethod(PaymentMethod* pm) {
  delete itsPaymentMethod;
  itsPaymentMethod = pm;
}

void Employee :: SetAffiliation(Affiliation* af) {
  delete itsAffiliation;
  itsAffiliation = af;
}

bool Employee :: IsPayDate(const Date& payDate) const {
  return itsSchedule -> IspayDate(payDate);
}

Date Employee :: GetPayPeriodStartDate(const Date& payPeriodEndDate) const {
  return itsSchedule -> GetPayPeriodStartDate(payPeriodEndDiate);
}

void Employee :: Payday(Paycheck& pc) {
  Date payDate = pc.GetPayPeriodEndDate();
  double grossPay = itsClassification -> CalculatePay(pc);
  double deductions = itsAffiliation -> CalculateDeductions(pc);
  double netPay = grossPay deductions;
  pc.SetGrossPay(grossPay);
  pc.SetDeductions(deductions);
  pc.SetNetPay(netPay);
  itsPaymentMethod -> Pay(pc);
}  
```

## 메인 프로그램
급여의 메인 프로그램은 이제 입력 소스에서 트랜잭션을 파싱하고 난 다음 그것을 실행하는 루프로 표현될 수 있다.  
다음 두 그림은 메인프로그램의 정적 모델과 동적 모델을 묘사하고 있다.  
![KakaoTalk_20240513_203132410](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/3d125873-361a-4556-a460-99434e195fec)
PayrollApplication이 루프에 들어가 번갈아 TransactionSource로부터 트랜잭션을 요청하고, 이 Transaction 객체의 Execute를 실행한다.  
TransactionSource는 여러 방식으로 구현할 수 있는 추상 클래스다.  
이 정적 다이어그램은 TextParserTransactionSource라는 파생 클래스를 보여주는데, 이것은 들어오는 텍스트 스트림을 읽어 유스케이스에 설명된 트랜잭션으로 파싱한다.  
그리고 적절한 Transaction 객체를 생성하여 PayrollApplication에 전송한다.  
TransactionSource의 구현에서 인터페이스를 분리하면 트랜잭션의 소스가 추상화될 수 있다.  
예를들어 쉽게 PayrollApplication의 인터페이스를 GUITransactionSource나 RemoteTransactionSource로 바꿀 수 있다.  

## 데이터베이스
이제 이 반복은 분석, 설계, 구현되었으므로 데이터베이스의 역할을 생각해볼수 있다.  
애플리케이션에 관한 한, 데이터베이스는 단순히 저장 공간을 관리하는 메커니즘일 뿐이다.  
보통 데이터베이스는 설계와 구현에 있어 중심 요소로 고려되어서는 안된다.  
개발자는 설계에 기반을 두어 필요한 데이터베이스를 자유롭게 선택할 수 있고, 필요할 때 특정 데이터베이스 상품을 바꾸거나 교체할 수 있다.  

## 급여 관리 설계의 요약
이 설계는 많은 양의 추상화와 다형성을 사용했으며, 그로 인해 이 설계의 많은 부분이 급여 관리 정책의 변경에 닫혀 있게 되었다.  
이 애플리케이션은 기본 기본 월급에 기반해 분기별로 임금을 받는 직원과 보너스 지급 주기에 맞게 변경될 수 있다.  
이 변경은 설계의 추가를 필요로 하겠지만, 기존의 설계와 코드에서 아주 일부분만이 변경될 것이다.  
전체 과정을 진행하는동안 명확성과 폐쇄의 문제에 집중했고 그 결과로 급여 관리 애플리케이션의 좋은 초기 설계를 얻게 되었고, 전체저으로 문제 영역과 밀접한 관계가 있는 클래스들의 핵심을 얻게 되었다.  



