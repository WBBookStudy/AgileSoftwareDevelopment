# 18. 급여 관리 사례 연구: 구현 2

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

![KakaoTalk_Photo_2024-04-28-15-41-20 001jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/ff716b3e-00cb-4e61-935f-b367b1c7e40a)
![KakaoTalk_Photo_2024-04-28-15-41-21 002jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/42456e48-f383-4e9f-bbbe-6ccea5f39158)
모든 트랜잭션은 EmpID를 받으므로 ChangeEmployeeTransaction이라는 최상위의 기반 클래스를 만들 수 있음.  
구조는 위와 같이 설계할 수 있다. 임금구조를 바꾸는 트랜잭션은 또 추상화를 시킬 수 있음.(Change Classification Transaction)  
![KakaoTalk_Photo_2024-04-28-15-41-21 003jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/171ac8c8-3b54-4ab0-9951-ff655be71649)
> 모든 변경 트랜잭션에 대한 동적 모델을 보여준다. 템플릿 메소드 패턴을 사용한것을 볼 수 있다.
![KakaoTalk_Photo_2024-04-28-15-41-21 004jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/0b0fb10f-5513-41ff-9a1c-a8eae0c6f200)
> 위의 추상적 행동을 구체적으로 보여준다.
```Cpp
void PayrollTest :: TestChangeNameTransaction() {
  cerr << "TestChangeNameTransaction" << endl;
  int empId = 2;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.25);
  t.Exceute();
  ChangeNameTransaction cnt(empId, "Bob");
  cnt.Execute();
  Employee* e = GpayrollDatabase.GetEmployee(empId);
  assert(e);
  assert("Bob" == e -> GetName());
}
```
> 테스트코드
```Cpp
#ifndef CHANGEEMPLOYEETRANSACTION_H
#define CHANGEEMPLOYEETRANSACTION_H

#include "Transaction.h"
#include "Employee.h"
 
class ChangeEmployeeTransaction : public Transaction
{
 public:
  ChangeEmployeeTransaction(int empid);
  virtual ~ChangeEmployeeTransaction();
  virtual void Execute();
  virtual void Change(Employee&) = 0;

 private:
  int itsEmpId;
};

#endif
```

```Cpp
#include "ChangeEmployeeTransaction.h"
#include "Employee.h"
#include "PayrollDatabase.h"

extern PayrollDatabase GpayrollDatabase;

ChangeEmployeeTransaction::~ChangeEmployeeTransaction()
{
}

ChangeEmployeeTransaction::ChangeEmployeeTransaction(int empid)
  : itsEmpId(empid)
{
}

void ChangeEmployeeTransaction::Execute()
{
  Employee* e = GpayrollDatabase.GetEmployee(itsEmpId);
  if (e != 0)
    Change(*e);
}
```
> 구현~

#### ChangeAddressTransaction
```Cpp
#ifndef CHANGEADDRESSTRANSACTION_H
#define CHANGEADDRESSTRANSACTION_H

#include "ChangeEmployeeTransaction.h"
#include <string>

class ChangeAddressTransaction : public ChangeEmployeeTransaction
{
 public:
  virtual ~ChangeAddressTransaction();
  ChangeAddressTransaction(int empid, string address);
  virtual void Change(Employee& e);

 private:
  string itsAddress;
};

#endif
```
```Cpp
#include "ChangeAddressTransaction.h"

ChangeAddressTransaction::~ChangeAddressTransaction()
{
}

ChangeAddressTransaction::ChangeAddressTransaction(int empid, string address)
  : ChangeEmployeeTransaction(empid)
    , itsAddress(address)
{
}

void ChangeAddressTransaction::Change(Employee& e)
{
  e.SetAddress(itsAddress);
}
```
> 위와 똑같이 구현해준다.

#### ChangeNameTransaction
```Cpp
#ifndef CHANGENAMETRANSACTION_H
#define CHANGENAMETRANSACTION_H

#include "ChangeEmployeeTransaction.h"
#include <string>

class ChangeNameTransaction : public ChangeEmployeeTransaction
{
 public:
  virtual ~ChangeNameTransaction();
  ChangeNameTransaction(int empid, string name);
  virtual void Change(Employee&);

 private:
  string itsName;
};

#endif
```

```Cpp
#include "ChangeNameTransaction.h"

ChangeNameTransaction::~ChangeNameTransaction()
{
}

ChangeNameTransaction::ChangeNameTransaction(int empid, string name)
  : ChangeEmployeeTransaction(empid)
  , itsName(name)
{
}

void ChangeNameTransaction::Change(Employee& e)
{
  e.SetName(itsName);
}
```

### 임금 종류 변경
![KakaoTalk_Photo_2024-04-28-16-16-26 001jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/cfd1cae0-8aea-497f-9056-42e34751ca99)
> ChangeClassificationTransaction의 동적행위. 템플릿 메소드 패턴이 또 쓰였다.
![KakaoTalk_Photo_2024-04-28-16-16-26 002jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/5a86e8d7-a556-44d6-aaa4-1e6cf06228be)
![KakaoTalk_Photo_2024-04-28-16-16-26 003jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/dc6d863a-bfde-4041-bee3-c1207e83145e)
![KakaoTalk_Photo_2024-04-28-16-16-26 004jpeg](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/2788b137-c156-47b4-aedb-1286ec0fc492)
> 각각의 구체 구현된 동적행위  
  
```Cpp
void PayrollTest :: TestChangeHourlyTransaction() {
  cerr << "TestChangeHourlyTransaction" << endl;
  int empId = 3;
  AddCommissionedEmployee t(empId, "Lance", "Home", 2500, 3.2); // 수수료를 받는 직원 생성
  t.Exceute();
  ChangeHourlyTransaction cht(empId, 27.52); // 변경트랜잭션 생성
  cht.Execute(); // 변경트랜잭션 실행
  Employee* e = GpayrollDatabase.GetEmployee(empId); // 직원 가져와서
  assert(e);
  PaymentClassification* pc = e -> GetClassification();
  assert(pc);
  HourlyClassification* hc = dynamic_cast<HourlyClassification*>(pc); // 적용이 잘 되었나? 확인할것임
  assert(hc);
  assertEquals(27.52, hc -> GetRate(), .001);
  PaymentSchedule* ps = e -> GetSchedule();
  WeeklySchedule* ws = dynamic_cast<WeeklySchedule*>(ps); // 스케쥴 적용도 잘 되었나? 확인할것임
  assert(ws);
}
```
> ChangeHourlyTransaction의 테스트 케이스

```Cpp
#ifndef CHANGECLASSIFICATIONTRANSACTION_H
#define CHANGECLASSIFICATIONTRANSACTION_H

#include "ChangeEmployeeTransaction.h"

class PaymentClassification;
class PaymentSchedule;

class ChangeClassificationTransaction : public ChangeEmployeeTransaction
{
 public:
  virtual ~ChangeClassificationTransaction();
  ChangeClassificationTransaction(int empid);
  virtual void Change(Employee&);
  virtual PaymentClassification* GetClassification() const = 0;
  virtual PaymentSchedule* GetSchedule() const = 0; 
};
#endif
```
```Cpp
#include "ChangeClassificationTransaction.h"

ChangeClassificationTransaction::~ChangeClassificationTransaction()
{
}

ChangeClassificationTransaction::ChangeClassificationTransaction(int empid)
  : ChangeEmployeeTransaction(empid)
{
}

void ChangeClassificationTransaction::Change(Employee& e)
{
  e.SetClassification(GetClassification());
  e.SetSchedule(GetSchedule());
}
```
> 가상함수들을 포함한 추상클래스. 이녀석들을 이용해 만들어보자.

```Cpp
#ifndef CHANGEHOURLYTRANSACTION_H
#define CHANGEHOURLYTRANSACTION_H

#include "ChangeClassificationTransaction.h"

class ChangeHourlyTransaction : public ChangeClassificationTransaction
{
 public:
  virtual ~ChangeHourlyTransaction();
  ChangeHourlyTransaction(int empid, double hourlyRate);
  virtual PaymentSchedule* GetSchedule() const;
  virtual PaymentClassification* GetClassification() const;

 private:
  double itsHourlyRate;
};

#endif
```

```Cpp
#include "ChangeHourlyTransaction.h"
#include "WeeklySchedule.h"
#include "HourlyClassification.h"

ChangeHourlyTransaction::~ChangeHourlyTransaction()
{
}

ChangeHourlyTransaction::ChangeHourlyTransaction(int empid, double hourlyRate)
  : ChangeClassificationTransaction(empid)
    , itsHourlyRate(hourlyRate)
{
}

PaymentSchedule* ChangeHourlyTransaction::GetSchedule() const
{
  return new WeeklySchedule();
}

PaymentClassification* ChangeHourlyTransaction::GetClassification() const
{
  return new HourlyClassification(itsHourlyRate);
}
```
> ChangeHourlyTransaction

```Cpp
#ifndef CHANGESALARIEDTRANSACTION_H
#define CHANGESALARIEDTRANSACTION_H

#include "ChangeClassificationTransaction.h"

class ChangeSalariedTransaction : public ChangeClassificationTransaction
{
 public:
  virtual ~ChangeSalariedTransaction();
  ChangeSalariedTransaction(int empid, double salary);
  virtual PaymentSchedule* GetSchedule() const;
  virtual PaymentClassification* GetClassification() const;

 private:
  double itsSalary;
};

#endif
```
```Cpp
#include "ChangeSalariedTransaction.h"
#include "MonthlySchedule.h"
#include "SalariedClassification.h"

ChangeSalariedTransaction::~ChangeSalariedTransaction()
{
}

ChangeSalariedTransaction::ChangeSalariedTransaction(int empid, double salary)
: ChangeClassificationTransaction(empid)
, itsSalary(salary)
{
}

PaymentSchedule* ChangeSalariedTransaction::GetSchedule() const
{
  return new MonthlySchedule();
}

PaymentClassification* ChangeSalariedTransaction::GetClassification() const
{
  return new SalariedClassification(itsSalary);
}
```
> ChangeSalariedTransaction

```Cpp
#ifndef CHANGECOMMMISSIONEDTRANSACTION_H
#define CHANGECOMMISSIONEDTRANSACTION_H

#include "ChangeClassificationTransaction.h"

class ChangeCommissionedTransaction : public ChangeClassificationTransaction
{
 public:
  virtual ~ChangeCommissionedTransaction();
  ChangeCommissionedTransaction(int empid, double salary, double rate);
  virtual PaymentSchedule* GetSchedule() const;
  virtual PaymentClassification* GetClassification() const;

 private:
  double itsSalary;
  double itsRate;
};

#endif
```
```Cpp
#include "ChangeCommissionedTransaction.h"
#include "BiweeklySchedule.h"
#include "CommissionedClassification.h"

ChangeCommissionedTransaction::~ChangeCommissionedTransaction()
{
}

ChangeCommissionedTransaction::ChangeCommissionedTransaction(int empid, double salary, double rate)
: ChangeClassificationTransaction(empid)
, itsSalary(salary)
, itsRate(rate)
{
}

PaymentSchedule* ChangeCommissionedTransaction::GetSchedule() const
{
  return new BiweeklySchedule();
}

PaymentClassification* ChangeCommissionedTransaction::GetClassification() const
{
  return new CommissionedClassification(itsSalary, itsRate);
}
```
> ChangeCommissionedTransaction

### 메소드 종류 변경

![KakaoTalk_Photo_2024-04-28-16-33-50](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/97230286-4eb8-43f4-a9e4-1952a5a96f9d)

```Cpp
#ifndef CHANGEMETHODTRANSACTION_H
#define CHANGEMETHODTRANSACTION_H
#include "ChangeEmployeeTransaction.h"

class PaymentMethod;

class ChangeMethodTransaction : public ChangeEmployeeTransaction
{
 public:
  virtual ~ChangeMethodTransaction();
  ChangeMethodTransaction(int empid);

  virtual PaymentMethod* GetMethod() const = 0;
  virtual void Change(Employee&);
};

#endif
```
```Cpp
#include "ChangeMethodTransaction.h"

ChangeMethodTransaction::~ChangeMethodTransaction()
{
}

ChangeMethodTransaction::ChangeMethodTransaction(int empid)
: ChangeEmployeeTransaction(empid)
{
}

void ChangeMethodTransaction::Change(Employee& e)
{
  e.SetMethod(GetMethod());
}
```
> 추상클래스

```Cpp
#ifndef CHANGEDIRECTTRANSACTION_H
#define CHANGEDIRECTTRANSACTION_H

#include "ChangeMethodTransaction.h"
#include <string>

class ChangeDirectTransaction : public ChangeMethodTransaction
{
 public:
  virtual ~ChangeDirectTransaction();
  ChangeDirectTransaction(int empid, string bank, string account);
  virtual PaymentMethod* GetMethod() const;
 private:
  string itsBank;
  string itsAccount;
};
#endif
```
```Cpp
#include "ChangeDirectTransaction.h"
#include "DirectMethod.h"

ChangeDirectTransaction::~ChangeDirectTransaction()
{
}

ChangeDirectTransaction::ChangeDirectTransaction(int empid, string bank, string account)
  : ChangeMethodTransaction(empid)
    , itsBank(bank)
    , itsAccount(account)
{
}

PaymentMethod* ChangeDirectTransaction::GetMethod() const
{
  return new DirectMethod(itsBank, itsAccount);
}
```
> ChangeDirectTransaction
```Cpp
#ifndef CHANGEMAILTRANSACTION_H
#define CHANGEMAILTRANSACTION_H

#include "ChangeMethodTransaction.h"
#include <string>

class ChangeMailTransaction : public ChangeMethodTransaction
{
 public:
  virtual ~ChangeMailTransaction();
  ChangeMailTransaction(int empid, string address);
  virtual PaymentMethod* GetMethod() const;
 private:
  string itsAddress;
};

#endif
```
```Cpp
#include "ChangeMailTransaction.h"
#include "MailMethod.h"

ChangeMailTransaction::~ChangeMailTransaction()
{
}

ChangeMailTransaction::ChangeMailTransaction(int empid, string address)
: ChangeMethodTransaction(empid)
, itsAddress(address)
{
}

PaymentMethod* ChangeMailTransaction::GetMethod() const
{
  return new MailMethod(itsAddress);
}
```
> ChangeMailTransaction
```Cpp
#ifndef CHANGEHOLDTRANSACTION_H
#define CHANGEHOLDTRANSACTION_H

#include "ChangeMethodTransaction.h"

class ChangeHoldTransaction : public ChangeMethodTransaction
{
 public:
  virtual ~ChangeHoldTransaction();
  ChangeHoldTransaction(int empid);
  virtual PaymentMethod* GetMethod() const;
};
#endif
```
```Cpp
#include "ChangeHoldTransaction.h"
#include "HoldMethod.h"

ChangeHoldTransaction::~ChangeHoldTransaction()
{
}

ChangeHoldTransaction::ChangeHoldTransaction(int empid)
: ChangeMethodTransaction(empid)
{
}

PaymentMethod* ChangeHoldTransaction::GetMethod() const
{
  return new HoldMethod();
}
```
> ChangeHoldTransaction

## 무엇을 깨달았는가?
![KakaoTalk_Photo_2024-04-28-16-39-14](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/a4ca98ba-50c4-4e3b-91a0-db4a57bb336e)
```Cpp
void PayrollTest :: TestChangeMemberTransaction() {
  cerr << "TestChangeMemeberTransaction" << endl;
  int empId = 2;
  int memberId = 7734;
  AddHourlyEmployee t(empId, "Bill", "Home", 15.25); // 시간제 직원 생성
  t.Exceute();
  TestChangeMemberTransaction cmt(empId, memberId, 99.42); // 조합이 넣기 위해 ChangeMember 생성, 실행
  cmt.Execute();
  Employee* e = GpayrollDatabase.GetEmployee(empId);
  assert(e);
  Affiliation* af = e -> GetAffiliation(); // 이하 UnionAffiliation객체를 갖고 비율등을 확인.
  assert(af);
  UnionAffiliation* uf = dynamic_cast<UnionAffiliation*>(af);
  assert(uf);
  assertEquals(99.42, uf -> GetDues(), .001);
  Employee* member = GpayrollDatabase.GetUnionMember(memberId);
  assert(member); // 이부분 주목
  assert(e == member);
}
```
> PayrollDatabase가 Bill이 조합원이라는 사실을 기록했음을 확인하지만 UML에는 없는 내용임  
> 조합원임을 기록하는 일은 ChangeMemberTransaction이 하지만, 그것을 지우는 일은 ChangeUnaffilaitedTransaction이 하자.  
> 그것을 구현하기 위해 RecordMembership(Employee*)라는 또 다른 함수를 ChangeAffiliationTransaction에 추가하는 것이다.  

```Cpp
#ifndef CHANGEAFFILIATIONTRANSACTION_H
#define CHANGEAFFILIATIONTRANSACTION_H

#include "ChangeEmployeeTransaction.h"

class ChangeAffiliationTransaction : public ChangeEmployeeTransaction
{
 public:
  virtual ~ChangeAffiliationTransaction();
  ChangeAffiliationTransaction(int empid);
  virtual Affiliation* GetAffiliation() const = 0;
  virtual void RecordMembership(Employee*) = 0;
  virtual void Change(Employee&);
};

#endif
```
```Cpp
#include "ChangeAffiliationTransaction.h"

ChangeAffiliationTransaction::~ChangeAffiliationTransaction()
{
}

ChangeAffiliationTransaction::ChangeAffiliationTransaction(int empid)
: ChangeEmployeeTransaction(empid)
{
}

void ChangeAffiliationTransaction::Change(Employee& e)
{
  RecordMembership(&e);
  e.SetAffiliation(GetAffiliation());
}
```
> 뭐.. 똑같음..

```Cpp
#ifndef CHANGEMEMBERTRANSACTION_H
#define CHANGEMEMBERTRANSACTION_H

#include "ChangeAffiliationTransaction.h"

class ChangeMemberTransaction : public ChangeAffiliationTransaction
{
 public:
  virtual ~ChangeMemberTransaction();
  ChangeMemberTransaction(int empid, int memberid, double dues);
  virtual Affiliation* GetAffiliation() const;
  virtual void RecordMembership(Employee*);
 private:
  int itsMemberId;
  double itsDues;
};
#endif
```

```Cpp
#include "ChangeMemberTransaction.h"
#include "UnionAffiliation.h"
#include "PayrollDatabase.h"

extern PayrollDatabase GpayrollDatabase;

ChangeMemberTransaction::~ChangeMemberTransaction()
{
}

ChangeMemberTransaction::ChangeMemberTransaction(int empid, int memberid, double dues)
: ChangeAffiliationTransaction(empid)
, itsMemberId(memberid)
, itsDues(dues)
{
}

Affiliation* ChangeMemberTransaction::GetAffiliation() const
{
  return new UnionAffiliation(itsMemberId, itsDues);
}

void ChangeMemberTransaction::RecordMembership(Employee* e)
{
  GpayrollDatabase.AddUnionMember(itsMemberId, e);
}
```
> ChangeMemberTransaction

```Cpp
#ifndef CHANGEUNAFFILIATEDTRANSACTION_H
#define CHANGEUNAFFILIATEDTRANSACTION_H

#include "ChangeAffiliationTransaction.h"

class ChangeUnaffiliatedTransaction : public ChangeAffiliationTransaction
{
 public:
  virtual ~ChangeUnaffiliatedTransaction();
  ChangeUnaffiliatedTransaction(int empId);
  virtual Affiliation* GetAffiliation() const;
  virtual void RecordMembership(Employee*);
};
#endif
```

```Cpp
#include "ChangeUnaffiliatedTransaction.h"
#include "NoAffiliation.h"
#include "UnionAffiliation.h"
#include "PayrollDatabase.h"

extern PayrollDatabase GpayrollDatabase;

ChangeUnaffiliatedTransaction::~ChangeUnaffiliatedTransaction()
{
}

ChangeUnaffiliatedTransaction::ChangeUnaffiliatedTransaction(int empId)
: ChangeAffiliationTransaction(empId)
{
}

Affiliation* ChangeUnaffiliatedTransaction::GetAffiliation() const
{
  return new NoAffiliation();
}

void ChangeUnaffiliatedTransaction::RecordMembership(Employee* e)
{
  Affiliation* af = e->GetAffiliation();
  if (UnionAffiliation* uf = dynamic_cast<UnionAffiliation*>(af)) {
    int memberId = uf->GetMemberId();
    GpayrollDatabase.RemoveUnionMember(memberId);
  }
}
```
> RecordMembership함수는 현재 직원지 조합원인지 아닌지를 결정해야한다. 만약 조합원이라면 UnionAffiliation에서 memberId를 받아 조합원임을 나타내는 레코드를 지운다.



















