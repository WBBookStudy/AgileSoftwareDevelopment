# 14. 템플릿 메소드와 스트래터지 패턴: 상속과 위임
이번 장에서는 상속과 위임의 차이를 전형적으로 보여주는 패턴 두가지를 다룬다.  
탬플릿 메소드와 스트래터지 패턴은 비슷한 문제를 해결하고, 호환되어 쓰인다.  
그러나 템플릿 메소드는 문제를 해결하기 위해 상속을 사용하는 반면, 스트래터지는 위임을 사용한다.  

## 템플릿 메소드 패턴
```JAVA
import java.io.BufferedReader;
import java.io.InputStreamReader;

public class ftocraw {
  public static void main(String[] args) throws Exception {
    InputStreamReader isr = new InputStreamReader(System.in);
    BufferedReader br = new BufferedReader(isr);
    boolean done = false;

    while(!done) {
      String fahrString = br.readLine();
      If(fahrString == null || fahrString.length() == 0)
        done = true;
      else {
        double fahr = Double.parseDouble(fahrString);
        double celcius = 5.0 /9.0 * (fahr - 32);
        System.out.println("F=" + fahr + ", C=" + celcius);
      }
    }
    System.out.println("ftoc exit");
  }
}
```
이 프로그램은 화씨온도를 섭씨온도로 변화하는 프로그램인데, 메인 루프 구조의 모든 구성 요소가 들어가 있다.  
초기화를 하고, 메인루프에서 주된 일을 하고, 정리를 한뒤 종료한다.  
템플릿 메소드 패턴을 적용하면 이 기본 구조를 ftoc 프로그램에서 분리해낼 수 있다.  
이 프로그램은 모든 일반적인 코드를 추상 기반 클래스에 구현되어 있는 메소드 하나에 집어넣는다.  
이 구현 메소드는 일반적인 알고리즘을 포함하고 있지만, 모든 구체적인 부분은 기반 클래스의 추상 메소드에 맡긴다.  
한 예로 Aplication이라는 추상 기반 클래스에 메인 루프 구조를 집어 넣을 수 있다.  
```JAVA
public abstract class Application {
  private boolean isDone = false;

  protected abstract void init();
  protected abstract void idle();
  protected abstract void cleanup();

  protected void setDone() {
    isDone = true;
  }

  public void run() {
    init();
    while(!done())
      idle();
    cleanup();
  }
}
```
이 클래스는 일반적인 메인 루프 애플리케이션을 묘사하는데, 구현된 run 함수에서 메인 루프를 볼 수 있다.  
모든 구체적인 작업이 추상메소드인 init, idle, cleanup에 맡겨진것 또한 볼 수 있다.  
init은 초기화작업을, idle은 프로그램의 주된 작업을 수행하며 setDone이 호출될때 까지 반복적으로 호출된다.  
cleanup은 종료하기 전에 필요한 일들을 수행한다.
Application을 상속해서 추상 메소드들의 내용을 채워 넣는 것만으로 ftoc 클래스를 다시 작성할 수 있다.  
```JAVA
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class ftocTemplateMethod extends Application {
  private InputStreamReader isr;
  private BufferedReader br;

  public static void main(String[] args) throw Exception {
    (new ftocTemplateMethod()).run();
  }

  protected void init() {
    isr = new InputStreamReader(System.in);
    br = new BufferedReader(isr);
  }

  protected void idle() {
    String fahrString = readLineAndReturnNullIfError();
    If(fahrString == null || fahrString.length() == 0)
        setDone();
    else {
        double fahr = Double.parseDouble(fahrString);
        double celcius = 5.0 /9.0 * (fahr - 32);
        System.out.println("F=" + fahr + ", C=" + celcius);
  }

  protected void cleanup() {
    System.out.println("ftoc exit");
  }

  private String readLineAndReturnNullIfError() {
    String s;
    try {
      s = br.readLine();
    } catch (IOException e) {
      s = null;
    }
    return s;
  }
}
```
예외 처리 때문에 코드가 좀 더 길어졌지만, 원래 ftoc 애플리케이션이 어떻게 템플릿 메소드 패턴에 맞게 변경되었는지 이해하기 쉬울 것 이다.  

### 패턴 오용
이 예제는 간단하고 템플릿 메소드 패턴의 구체적인 동작을 보여주기 위한 좋은 기반을 마련해 주기 떄문에 선택했을 뿐, 실제로 ftoc를 이렇게 만들기를 권하지는 않는다.  
이것은 패턴 오용의 좋은 예가 되는데, 이런 경우 오히려 프로그램이 복잡해지고 내용만 더 늘어나기만 할 수도 있다.  
디자인 패턴은 멋지고, 많은 설계 문제를 해결할 수 있지만 디자인 패턴이 존재한다는 사실 자체가 항상 디자인 패턴을 사용해야 한다는 뜻은 아니다.  
위 예제의 경우 템플릿 메소드를 문제에 적용 할 수는 있었지만, 이 패턴을 적용하는데 드는 비용이 결과적으로 생기는 이익보다 더 컸다.  

### 버블 정렬
```JAVA
public class BubbleSorter {
  static int operations = 0;

  public static int sort(int[] array) {
    operations = 0;
    if(array.length <= 1)
      return operations;

    for(int nextToLast = array.length -2; nextToLast >= 0; nextToLast--)
      for(int index = 0; index <= nextToLast; index++)
        compareAndSwap(array, index);
    return operations;
  }

  public static void swap(int[] array, int index) {
    int temp = array[index];
    array[index] = array[index + 1];
    array[index + 1] = temp;
  }

  private static void compareAndSwap(int[] array, int index) {
    if(array[index] > array[index + 1])
      swap(array, index);
    opertations++;
  }
}
```
BubbleSorter 클래스는 버블 정렬 알고리즘을 이용하여 정수의 배열을 정렬하는 방법을 알고 있다.  
BubbleSorter의 sort 메소드는 버블 정렬을 수행하는 알고리즘을 포함한다.  
swap와 compareAndSwap 이라는 2개의 보조적인 메소드는 정수와 배열의 구체적인 부분을 다루고 정렬 알고리즘이 필요로 하는 동작을 처리한다.  
템플릿 메소드 패턴을 사용하면 버블 정렬 알고리즘을 따로 떼어 BubbleSorter라는 이름의 추상 기반 클래스에 집어 넣을 수 있다.  
```JAVA
public abstract class BubbleSorter {
  private int operations = 0;
  protected int length = 0;

  protected int doSort() {
    operations = 0;
    if(length <= 1)
      return operations;
    for(int nextToLast = array.length -2; nextToLast >= 0; nextToLast--)
      for(int index = 0; index <= nextToLast; index++) {
        if(outOfOrder(index))
          swap(index);
        operations++;
      }

    return operations;
  }

  protected abstract void swap(int index);
  protected abstract boolean outOfOrder(int index);
}
```
BubbleSorter는 outOfOrder와 swap 이라는 추상 메소드를 호출하는 sort 함수의 구현을 포함한다.  
outOfOrder는 배열에서 인접한 2개의 원소를 비교하여 그 원소의 순서가 잘못되어 잇으면 true 값을 반환하는 메소드이고, swap은 배열에서 2개의 인접 원소를 교환하는 메소드이다.  
sort 메소드는 배열에 대해 알지 못하고, 그 배열에 어떤 현의 객체가 저장되어 있는지도 신경쓰지 않는다.  
그저 배열의 여러 인덱스에 대해 outOfOrder를 호출하고 그 인덱스가 교환되어야 하는지 아닌지를 판정한다.  
이제 BubbleSorter로 다른 어떤 종류의 객체든 정렬할 수 있는 간단한 파생 클래스를 만들 수 있다.
![KakaoTalk_20240310_231731931](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/ca128929-a698-4614-a51c-b97f9e4bebdf)
템플릿 메소드 패턴은 객체 지향 프로그래밍에서 고전적인 재사용 형태 중의 하나를 보여준다.  
일반적인 알고리즘은 기반 클래스에 있고, 다른 구체적인 내용에서 사용된다.  
그러나 이 기법은 비용을 수반하는데, 상속은 아주 강한 관계여서 파생클래스는 필연적으로 기반클래스에 묶이게 된다.

## 스트래터지 패턴
스트래터지 패턴은 일반적인 알고리즘과 구체적인 구현 사이의 의존성 반전 문제를 완전히 다른 방식으로 풀어낸다.  
위에서 패턴을 오용한 Application 예제를 다시 생각해보자.  
일반적인 알고리즘을 추상 기반 클래스에 넣는 대신, ApplicationRunner라는 이름의 구체 클래스에 넣는다.  
Application이란 이름의 인터페이스 안에서 일반적 알고리즘이 호출해야할 추상 메소드를 정의한다.  
이 인터페이스에서 ftoc를 파생시켜 ApplicationRunner에 넘겨주면 ApplicationRunner는 이 인터페이스에 위임한다.  
![KakaoTalk_20240310_233554369](https://github.com/jhkman/AgileSoftwareDevelopment/assets/50142323/eedb82bf-cc3f-4d04-8609-f93165f00055)

이 구조는 이익과 비용면에서 템플릿 메소드 구조에 비해 더 낫다는 사실이 명백한데, 스트래터지에는 템플릿 메소드보다 더 많은 전체 클래스 개수와 더 많은 간접 지정(indirection)이 있다.  
ApplicationRunner 내부의 위임 포인터는 실행 시간과 데이터 공간 면에서 상속의 경우보다 좀 더 많은 비용을 초래한다.  
반면 서로 다른 많은 애플리케이션을 실행한다면, ApplicationRunner 인스턴스를 재사용하여 Application의 다른 많은 구현에 이것을 넘겨줄 수 있을테고,  
그럼으로서 일반적인 알고리즘과 그것이 제어하는 구체적인 부분 사이의 결합 정도를 감소시킬 수 있다.  

### 다시 정렬하기
스트래티지 패턴을 사용한 버블 정렬 구현을 생각해보자.  
```JAVA
public class BubbleSorter {
  private int operations = 0;
  private int length = 0;
  private SortHandle itsSortHandle = null;

  public BubbleSorter(SortHandle handle) {
    itsSortHandle = handle;
  }

  public int sort(Object array) {
    itsSortHandle.setArray(array);
    length = itsSortHandle.length();
    operations = 0;
    if(length <= 1)
      return operations;

    for(int nextToLast = length - 2; nextToLast >= 0; nextToLast--)
      for(int index = 0; index <= nextToLast; index++) {
        if(itsSortHandle.outOfOrder(index))
          itsSortHandle.swap(index);
        operations++;
      }

    return operations;
  }
}
```
```JAVA
public class SortHandle {
  public void swap(int index);
  public boolean outOfOrder(int index);
  public int length();
  public void setArray(Object array);
}
```
```JAVA
public class IntSortHandle implements SortHandle {
  private int[] array = null;

  public void swap(int index) {
    int temp = array[index];
    array[index] = array[index + 1];
    array[index + 1] = temp;
  }

  public void setArray(Object array) {
    this.array = (int[]) array;
  }

  public int length() {
    return array.length;
  }

  public boolean outOfOrder(int index) {
    return (array[index] > array[index + 1]);
  }
}
```
IntSortHandle클래스가 BubbleSorter에 대해 아무것도 모른다는 점에 주목하자.  
이 클래스는 버블 정렬 구현부에 어떤 의존성도 갖고 있지 않은데, 이는 템플릿 메소드에서는 없었던 일이다.  
템플릿 메소드를 사용한 접근은 swap와 outOfOrder 메소드가 버블 정렬 알고리즘에 직접 의존하도록 구현함으로서 부분적으로 DIP를 위반한다.  
스트래터지를 사용한 접근은 이런 의존성을 포함하고 있지 않다. 따라서 IntSortHandle은 BubbleSorter가 아니라 다른 Sorter 구현과 함께 사용할 수 있다.  
이렇듯 스트래터지 패턴은 템플릿 메소드 패턴에 비해 한 가지 특별한 이점을 제공한다.  
템플릿 메소드 패턴이 일반적인 알고리즘으로 많은 구체적인 구현을 조작할 수 있게 해주는 반면,  
DIP를 완전히 따르는 스트래터지 패턴은 각각의 구체적인 구현이 다른 많은 일반적인 알고리즘에 의해 조작될 수 있게 해준다.

## 결론
템플릿 메소드와 스트래터지 패턴 모두 상위 단계의 알고리즘을 하위 단계의 구체적인 부분으로부터 분리해주는 역할을 하고, 상위단게의 알고리즘이 구체적인 부분과 독립적으로 재사용될 수 있게 해준다.  
약간의 복잡성과 메모리, 실행 시간을 더 감내하면 스트래터지는 구체적인 부분이 상위 단계 알고리즘으로부터 독립적으로 재사용될 수 있게까지 해준다.  



