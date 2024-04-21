# 18. 급여 관리 사례 연구: 구현

```Cpp
#ifndef TRANSACTION_H
#define TRANSACTION_H

class Transaction
{
 public:
  virtual ~Transaction();
  virtual void Execute() = 0;
};

#endif
```
> Transaction

## 직원 추가 


![KakaoTalk_Photo_2024-04-20-23-46-26](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/3c0f3344-eab2-45e6-aecb-cc04e0453f12)
> 직원을 추가하는 트랜잭션의 가능한 구조. 
  
 - 트랜잭션 내부에서 직원의 임금 지급 주기가 그들의 임금 분류와 연결되어 있다.
 - 기본적인 임금 지급 방법이 급여 담당자에게 임금을 맡겨두는 방법이다.  

테스트코드를 작성해보자.

```Cpp
void PayrollTest :: TestAddSalariedEmployee() {
	int empId = 1;
	AddSalariedEmployee t(empId, "Bob", "Home", 1000.00);
	t.Exceute();

	Employee* e = GpayrollDatabase.GetEmployee(empId);
	assert("Bob" == e -> GetName());

	PaymentClassification* pc = e -> GetClassification();
	SalariedClassification* sc = dynamic_cast<SalariedClassification*>(pc);
	assert(sc);

	assertEquals(1000.00, sc -> GetSalary(), .001);
	PaymentSchedule* ps = e -> GetSchedule();
	MonthlySchedule* ms = dynamic_cast<MonthlySchedule*>(ps);
	assert(ms);
	PaymentMethod* pm = e -> GetMethod();
	HoldMethod* hm = dynamic_cast<HoldMethod*>(pm);
	assert(hm);
}
```

## 급여 관리 데이터베이스
AddEmployeeTransaction 클래스는 PayrollDatabase라는 클래스를 사용한다. 이 클래스는 퍼사드 패턴의 한 예이다.
![KakaoTalk_Photo_2024-04-21-00-00-01](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/8816124d-bdec-4dc4-b3ba-4a365be6af05)
  
아래 코드는 초기코드임. 테스트 케이스를 도우려는 의도로 만든 것이어서, 아직 memberId를 Employee 인스턴스에 연결하는 Dictionary를 포함하고 있지 않다.
```Cpp
#ifndef PAYROLLDATABASE_H
#define PAYROLLDATABASE_H

#include <map>

class Employee;

class PayrollDatabase
{
 public:
  virtual ~PayrollDatabase();
  Employee* GetEmployee(int empId);
  void AddEmployee(int empid, Employee*);
  void clear() {
  	itsEmployees.clear();
  }

 private:
  map<int, Employee*> itsEmployees;
};

#endif
```

```Cpp
#include "PayrollDatabase.h"
#include "Employee.h"

PayrollDatabase GpayrollDatabase;

PayrollDatabase::~PayrollDatabase()
{
}

Employee* PayrollDatabase::GetEmployee(int empid)
{
  return itsEmployees[empid];
}

void PayrollDatabase::AddEmployee(int empid, Employee* e)
{
  itsEmployees[empid] = e;
}
```

일반적으로 데이터베이스 구현은 구체적인 사항이므로 이에 대한 의사결정은 가능하면 미뤄두자.  

## 템플릿 메소드를 사용한 직원 추가
![KakaoTalk_Photo_2024-04-21-00-06-52](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/bcdd744a-2a22-4bb2-a157-8c30d5aaa858)
> 직원 추가를 위한 동적 모델. AddEmployeeTransaction객체가 적절한 PaymentClassification과 PaymentSchedule 객체를 얻기 위해 자신에게 메시지를 보내는것에 주목하자.  
> 이 메시지는 AddEmployeeTransaction 클래스의 파생클래스에서 구현된다. 이것은 탬플릿 메소드 패턴의 응용이다.

```Cpp
#ifndef ADDEMPLOYEETRANSACTION_H
#define ADDEMPLOYEETRANSACTION_H

#include "Transaction.h"
#include <string>

class PaymentClassification;
class PaymentSchedule;

class AddEmployeeTransaction : public Transaction
{
 public:
  virtual ~AddEmployeeTransaction();
  AddEmployeeTransaction(int empid, string name, string address);
  virtual PaymentClassification* GetClassification() const = 0;
  virtual PaymentSchedule* GetSchedule() const = 0;
  virtual void Execute();

 private:
  int itsEmpid;
  string itsName;
  string itsAddress;
};
#endif
```  
  
```Cpp
#include "AddEmployeeTransaction.h"
#include "HoldMethod.h"
#include "Employee.h"
#include "PayrollDatabase.h"

class PaymentMethod;
class PaymentSchedule;
class PaymentClassification;

extern PayrollDatabase GpayrollDatabase;

AddEmployeeTransaction::~AddEmployeeTransaction()
{
}

AddEmployeeTransaction::AddEmployeeTransaction(int empid, string name, string address)
  : itsEmpid(empid)
  , itsName(name)
  , itsAddress(address)
{
}

void AddEmployeeTransaction::Execute()
{
  PaymentClassification* pc = GetClassification();
  PaymentSchedule* ps = GetSchedule();
  PaymentMethod* pm = new HoldMethod();
  Employee* e = new Employee(itsEmpid, itsName, itsAddress);
  e->SetClassification(pc);
  e->SetSchedule(ps);
  e->SetMethod(pm);
  GpayrollDatabase.AddEmployee(itsEmpid, e);
}
```
템플릿 메소드 패턴의 구현을 보여준다. 이 클래스는 파생 클래스에서 구현될 2개의 순수 가상 함수를 호출할 Exceute()메소드를 구현한다.  
이 함수는 GetSchedule()과 GetClassification()이고, 새롭게 생성된 Employee가 필요로 하는 PaymentSchedule과 PaymentClassification객체를 반환한다.  
그러면 Exceute()메소드가 이 객체들을 묶어 Employee에 넣고 이걸 PayrollDatabase에 저장한다.

```Cpp
#ifndef ADDSALARIEDEMPLOYEE_H
#define ADDSALARIEDEMPLOYEE_H

#include "AddEmployeeTransaction.h"

class AddSalariedEmployee : public AddEmployeeTransaction
{
 public:
  virtual ~AddSalariedEmployee();
  AddSalariedEmployee(int empid, string name, string address, double salary);
  PaymentClassification* GetClassification() const;
  PaymentSchedule* GetSchedule() const;

 private:
  double itsSalary;
};
#endif
```
```Cpp
#include "AddSalariedEmployee.h"
#include "SalariedClassification.h"
#include "MonthlySchedule.h"

AddSalariedEmployee::~AddSalariedEmployee()
{
}

AddSalariedEmployee::AddSalariedEmployee(int empid, string name, string address, double salary)
  : AddEmployeeTransaction(empid, name, address)
    , itsSalary(salary)
{
}

PaymentClassification* AddSalariedEmployee::GetClassification() const
{
  return new SalariedClassification(itsSalary);
}

PaymentSchedule* AddSalariedEmployee::GetSchedule() const
{
  return new MonthlySchedule();
}
```
AddSalariedEmployee의 구현. AddEmployeeTransaction에서 파생되고, Exceute()에게 줄 GetClassification(), GetSchedule()를 반환을 구현한다.

### [숙제] AddHourlyEmployee
```Cpp
void PayrollTest :: TestAddHourlyEmployee() {
	int empId = 1;
	AddHourlyEmployee t(empId, "Bob", "Home", 1000.00);
	t.Exceute();

	Employee* e = GpayrollDatabase.GetEmployee(empId);
	assert("Bob" == e -> GetName());

	PaymentClassification* pc = e -> GetClassification();
	HourlyClassification* hc = dynamic_cast<HourlyClassification*>(pc);
	assert(hc);

	assertEquals(1000.00, hc -> GetSalary(), .001);
	PaymentSchedule* ps = e -> GetSchedule();
	WeeklySchedule* ws = dynamic_cast<WeeklySchedule*>(ps);
	assert(ws);
	PaymentMethod* pm = e -> GetMethod();
	HoldMethod* hm = dynamic_cast<HoldMethod*>(pm);
	assert(hm);
}
```

```Cpp
#ifndef ADDHOURLYEMPLOYEE_H
#define ADDHOURLYEMPLOYEE_H

#include "AddEmployeeTransaction.h"

class AddHourlyEmployee : public AddEmployeeTransaction
{
 public:
  virtual ~AddHourlyEmployee();
  AddHourlyEmployee(int empid, string name, string address, double hourlyRate);
  PaymentClassification* GetClassification() const;
  PaymentSchedule* GetSchedule() const;

 private:
  double itsHourlyRate;
};
#endif
```

```Cpp
#include "AddHourlyEmployee.h"
#include "HourlyClassification.h"
#include "WeeklySchedule.h"

AddHourlyEmployee::~AddHourlyEmployee()
{
}

AddHourlyEmployee::AddHourlyEmployee(int empid, string name, string address, double hourlyRate)
  : AddEmployeeTransaction(empid, name, address)
    , itsHourlyRate(hourlyRate)
{
}

PaymentClassification* AddHourlyEmployee::GetClassification() const
{
  return new HourlyClassification(itsHourlyRate);
}

PaymentSchedule* AddHourlyEmployee::GetSchedule() const
{
  return new WeeklySchedule();
}
```

### [숙제] AddCommissionedEmployee
```Cpp
void PayrollTest :: AddCommissionedEmployee() {
	int empId = 1;
	AddCommissionedEmployee t(empId, "Bob", "Home", 1000.00);
	t.Exceute();

	Employee* e = GpayrollDatabase.GetEmployee(empId);
	assert("Bob" == e -> GetName());

	PaymentClassification* pc = e -> GetClassification();
	CommissionedClassification* cc = dynamic_cast<CommissionedClassification*>(pc);
	assert(hc);

	assertEquals(1000.00, cc -> GetSalary(), .001);
	PaymentSchedule* ps = e -> GetSchedule();
	BiweeklySchedule* bs = dynamic_cast<BiweeklySchedule*>(ps);
	assert(bs);
	PaymentMethod* pm = e -> GetMethod();
	HoldMethod* hm = dynamic_cast<HoldMethod*>(pm);
	assert(hm);
}
```

```Cpp
#ifndef ADDCOMMISSIONEDEMPLOYEE_H
#define ADDCOMMISSIONEDEMPLOYEE_H

#include "AddEmployeeTransaction.h"

class AddCommissionedEmployee : public AddEmployeeTransaction
{
 public:
  virtual ~AddCommissionedEmployee();
  AddCommissionedEmployee(int empid, string name, string address, double salary, double commissionRate);
  PaymentClassification* GetClassification() const;
  PaymentSchedule* GetSchedule() const;

 private:
  double itsSalary;
  double itsCommissionRate;
};
#endif
```

```Cpp
#include "AddCommissionedEmployee.h"
#include "CommissionedClassification.h"
#include "BiweeklySchedule.h"

AddCommissionedEmployee::~AddCommissionedEmployee()
{
}

AddCommissionedEmployee::AddCommissionedEmployee(int empid, string name, string address, double salary, double commissionRate)
  : AddEmployeeTransaction(empid, name, address)
    , itsSalary(salary)
    , itsCommissionRate(commissionRate)
{
}

PaymentClassification* AddCommissionedEmployee::GetClassification() const
{
  return new CommissionedClassification(itsSalary, itsCommissionRate);
}

PaymentSchedule* AddCommissionedEmployee::GetSchedule() const
{
  return new BiweeklySchedule();
}
```

## 직원 삭제
![KakaoTalk_Photo_2024-04-21-00-28-38](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/ce1a3df6-db3e-44f4-aee4-1d28bbff3779)

```Cpp
void PayrollTest :: TestDelegateEmployee() {
	cerr << "TestDelegateEmployee" << endl;
	int empId = 3;
	AddCommissionEmployee t(empId, "Lance", "Home", 2500, 3.2);
	t.Execute();
	{
		Employee* e = GpayrollDatabase.GetEmployee(empId);
		assert(e);
	}
	DelegateEmployeeTransaction dt(empId);
	dt.Execute();
	{
		Employee* e = GpayrollDatabase.GetEmployee(empId);
		assert(e == 0);
	}
}
```

```Cpp
#ifndef DELETEEMPLOYEETRANSACTION_H
#define DELETEEMPLOYEETRANSACTION_H

#include "Transaction.h"

class DeleteEmployeeTransaction : public Transaction
{
 public:
  virtual ~DeleteEmployeeTransaction();
  DeleteEmployeeTransaction(int empid);
  virtual void Execute();
 private:
  int itsEmpid;
};
#endif
```

```Cpp
#include "DeleteEmployeeTransaction.h"
#include "PayrollDatabase.h"

extern PayrollDatabase GpayrollDatabase;
DeleteEmployeeTransaction::~DeleteEmployeeTransaction()
{
}

DeleteEmployeeTransaction::DeleteEmployeeTransaction(int empid)
  : itsEmpid(empid)
{
}

void DeleteEmployeeTransaction::Execute()
{
  GpayrollDatabase.DeleteEmployee(itsEmpid);
}
```
커맨드 패턴의 아주 전형적인 구현 형태. 생성자는 최종적으로 Execute()메소드가 사용하는 데이터를 저장한다.

### 전역변수
GpayrollDatabase가 전역변수이다. 이게 안좋은가? 여기선 아님. 단 하나의 PayrollDatabase 클래스 인스턴스가 있는데, 이는 넓은 범위에 알려져야 한다.  

## 타임카드, 판매 영수증, 공제액

![KakaoTalk_Photo_2024-04-21-19-12-05](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/ccd2bb36-d0fc-4e25-93cb-5eae48c31c6b)

![KakaoTalk_Photo_2024-04-21-19-12-08](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/f3cd278f-ced5-40ea-803a-42731f954398)
> 트랜잭션이 PayrollDatabase에서 Employee객체를 받아 그 객체에 있는 PaymentClassification객체를 요청하고, TimeCard 객체를 생성하여 그 PaymentClassification에 더하는 것.

```Cpp
void PayrollTest :: TestTimeCardTransaction() {
	cer << "TestTimeCardTransaction" << endl;
	int empId = 2;
	AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
	t.Execute();
	TestTimeCardTransaction tct(20011031, 8.0, empId);
	tct.Execute();
	Employee* e = GpayrollDatabase.GetEmployee(empId);
	assert(e);
	PaymentClassification* pc = e -> GetClassification();
	HourlyClassification* hc = dynamic_cast<HourlyClassification*>(pc);
	assert(hc);
	TimeCard* tc = hc -> GetTimeCard(20011031);
	assert(tc);
	assertEquals(8.0, tc -> GetHours());
}
```
> 타임카드 객체를 시간제 직원 객체에 더할 수 있는지 검증하는 테스트 케이스

```Cpp
#ifndef TIMECARD_H
#define TIMECARD_H

#include "Date.h"

class TimeCard
{
 public:
  virtual ~TimeCard();
  TimeCard(long date, double hours);
  Date GetDate() {return itsDate;}
  double GetHours() {return itsHours;}
 private:
  long itsDate;
  double itsHours;
};
#endif
```
> 완성된건 아니고 그냥 데이터 클래스. Date클래스가 없어서 long을 사용함.

```Cpp
#ifndef TIMECARDTRANSACTION_H
#define TIMECARDTRANSACTION_H

#include "Transaction.h"

class TimeCardTransaction : public Transaction
{
 public:
  virtual ~TimeCardTransaction();
  TimeCardTransaction(long date, double hours, int empid);

  virtual void Execute();

 private:
  int itsEmpid;
  Date itsDate;
  double itsHours;
};
#endif
```

```Cpp
#include "TimeCardTransaction.h"
#include "Employee.h"
#include "PayrollDatabase.h"
#include "HourlyClassification.h"
#include "TimeCard.h"
#include "Date.h"

extern PayrollDatabase GpayrollDatabase;

TimeCardTransaction::~TimeCardTransaction()
{
}

TimeCardTransaction::TimeCardTransaction(long date, double hours, int empid)
  : itsDate(date)
  , itsHours(hours)
  , itsEmpid(empid)
{
}

void TimeCardTransaction::Execute()
{
  Employee* e = GpayrollDatabase.GetEmployee(itsEmpid);
  if (e){
    PaymentClassification* pc = e->GetClassification();
    if (HourlyClassification* hc = dynamic_cast<HourlyClassification*>(pc)) {
      hc->AddTimeCard(new TimeCard(itsDate, itsHours));
    } else
      throw("Tried to add timecard to non-hourly employee");
  } else
    throw("No such employee.");
}
```
> TimeCardTransaction 구현

### [숙제] SalesReceiptTransaction 구현
![KakaoTalk_Photo_2024-04-21-19-30-12 001jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/1e6e157e-ea54-4878-ba59-4be7e4634c5a)
![KakaoTalk_Photo_2024-04-21-19-30-13 002jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/e9895b3e-8c46-4d55-8f9c-ccd0455fb96a)
테스트코드 작성 안함 ㅎㅎ;;
```Cpp
#ifndef SALESRECEIPT_H
#define SALESRECEIPT_H

class SalesReceipt
{
 public:
  virtual ~SalesReceipt();
  SalesReceipt(long saleDate, double amount);
  Date GetSaleDate() const {return itsSaleDate;}
  double GetAmount() const {return itsAmount;}
 private:
  Date itsSaleDate;
  double itsAmount;
};
#endif
```

```Cpp
#ifndef SALESRECEIPTTRANSACTION_H
#define SALESRECEIPTTRANSACTION_H

#include "Transaction.h"

class SalesReceiptTransaction : public Transaction
{
 public:
  virtual ~SalesReceiptTransaction();
  SalesReceiptTransaction(long saleDate, double amount, int empid);

  virtual void Execute();

 private:
  int itsEmpid;
  Date itsSaleDate;
  double itsAmount;
};
#endif
```

```Cpp
#include "SalesReceiptTransaction.h"
#include "Employee.h"
#include "PayrollDatabase.h"
#include "CommissionedClassification.h"
#include "SalesReceipt.h"

extern PayrollDatabase GpayrollDatabase;

SalesReceiptTransaction::~SalesReceiptTransaction()
{
}

SalesReceiptTransaction::SalesReceiptTransaction(long saleDate, double amount, int empid)
: itsSaleDate(saleDate)
, itsAmount(amount)
, itsEmpid(empid)
{
}

void SalesReceiptTransaction::Execute()
{
  Employee* e = GpayrollDatabase.GetEmployee(itsEmpid);
  if (e){
    PaymentClassification* pc = e->GetClassification();
    if (CommissionedClassification* cc = dynamic_cast<CommissionedClassification*>(pc)) {
      cc->AddReceipt(new SalesReceipt(itsSaleDate, itsAmount));
    } else
      throw("Tried to add sales receipt to non-commissioned employee");
  } else
    throw("No such employee.");
}
```

### 조합원의 공제액
설계가 좀 안맞을 수 있음. 핵심 Employee 객체들은 서로 다른 여러 단체에 가입되어 있을 수도 있지만, 트랜잭션 모델은 모든 소속 단체가 조합이어야 함을 가정한다.  
이 딜레마를 해결하는 동적 모델은 Employee객체가 포함하는 Affiliation 객체들의 집합을 검색해서 UnionAffiliation객체를 찾는 것. 그리고 그 UnionAffiliation에 ServiceCharge 객체를 더한다.  
![KakaoTalk_Photo_2024-04-21-19-37-46 001jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/27c3a3cd-b034-46ce-841a-51f2e026e999)
![KakaoTalk_Photo_2024-04-21-19-37-47 002jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/1bc73e7f-c07f-4eea-9b9c-75b7fa97ae5d)
```Cpp
void PayrollTest :: TestAddServiceCharge() {
	cer << "TestAddServiceCharge" << endl;
	int empId = 2;
	AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
	t.Exceute();
	Employee* e = GpayrollDatabase.GetEmployee(empId);
	assert(e);
	UnionAffiliation* af = new UnionAffiliation(12.5);
	e -> SetAffiliation(af);
	int memberId = 86; // Maxwell Smart
	GpayrollDatabase.AddUnionMember(memberId, e);
	ServiceChargeTransaction sct(memberId, 20011101, 12.95);
	sct.Execute();
	ServiceCharge* sc = af -> GetServiceCharge(20011101);
	assert(sc);
	assertEquals(12.95, sc -> GetAmount(), .001);
}
```

```Cpp
#ifndef SERVICECHARGETRANSACTION_H
#define SERVICECHARGETRANSACTION_H

#include "Transaction.h"

class ServiceChargeTransaction : public Transaction
{
 public:
  virtual ~ServiceChargeTransaction();
  ServiceChargeTransaction(int memberId, long date, double charge);
  virtual void Execute();

 private:
  int itsMemberId;
  Date itsDate;
  double itsCharge;
};
#endif
```

```Cpp
#include "ServiceChargeTransaction.h"
#include "Employee.h"
#include "ServiceCharge.h"
#include "PayrollDatabase.h"
#include "UnionAffiliation.h"

extern PayrollDatabase GpayrollDatabase;

ServiceChargeTransaction::~ServiceChargeTransaction()
{
}

ServiceChargeTransaction::ServiceChargeTransaction(int memberId, long date, double charge)
:itsMemberId(memberId)
, itsDate(date)
, itsCharge(charge)
{
}

void ServiceChargeTransaction::Execute()
{
  Employee* e = GpayrollDatabase.GetUnionMember(itsMemberId);
  Affiliation* af = e->GetAffiliation();
  if (UnionAffiliation* uaf = dynamic_cast<UnionAffiliation*>(af)) {
    uaf->AddServiceCharge(itsDate, itsCharge);
  }
}
```


## 직원 변경
은.... 다음주에... ㅎㅎㅎㅎ

