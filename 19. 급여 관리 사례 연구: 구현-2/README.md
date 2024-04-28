# 18. 급여 관리 사례 연구: 구현 2

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



















