---
title: "Java Memory Structure"
excerpt: "memory management in java"

date: 2022-12-11
last_modified_at: 2022-12-11

categories:
  - study

tags:
  - Java
  - theoretical

comments: true

toc: true
toc_sticky: true
---

## Intro

![java_giphy.gif](https://media.giphy.com/media/Zei8AMzoPUTQGSPK02/giphy.gif){: .align-center}

↪ 지난번에는 메모리에 대해 다루었습니다. 이번에는 메모리를 Java에서 어떻게 manage하는지에 대해 간단하게 살펴보려고 합니다. 이번 포스트를 작성하면서 잘못 알고 있던 부분도 몇가지 발견해 개인적으로도 의미있는 시간이었습니다.

---

## Memory Structure in Java
### JVM

![jvm_01.jpg](/assets/images/posts/2022-12-11-java-memory-structure/jvm_01.jpg){: .align-center}

↪ 우리가 작성한 .java 파일은 javac(Java 컴파일러)에 의해 .class 파일로 컴파일됩니다. 이렇게 만들어진 Class 파일은 Class Loader에 의해 Runtime Data Area(메모리) 상에 적재됩니다. 그리고 이를 Execution Engine이 바이트 코드로 변환해 수행합니다.

이번에 알아볼 부분은 특히 이 중에서 <span style="color:red">**Runtime Data Area**</span>, 즉 메모리 영역에 관한 내용입니다. Runitme Data Area는 그림과 같이 Thread마다 할당되는 부분과 모든 Thread가 공유해 사용하는 영역이 존재하는데, 이 <span style="color:red">**Shared Area**</span>가 GC(Garbage Collector)의 타겟이 됩니다.

이 포스트에서 자세히 설명하는 내용에서 제외되는 부분인 PC Register와 Native Method Stack에 대해서는 간략히 적고 넘어가도록 하겠습니다.

- <span style="color:red">**PC Register**</span>: Thread에서 실행중인 메서드 중  실행중인 instruction 주소입니다. instruction이란 Class가 변환된 바이트 코드의 각 라인을 의미합니다.
- <span style="color:red">**Native Method Stack**</span>: Java가 아닌 C/C++와 같은 native programming language로 작성한 메소드를 호출할 때 사용되는 영역입니다.

### Shared Area

![shared_area_01.jpg](/assets/images/posts/2022-12-11-java-memory-structure/shared_area_01.jpg){: .align-center}

↪ 앞선 JVM의 전체 구조 중 공유 메모리 영역에 대한 그림입니다. 크게는 <span style="color:red">**Heap**</span> 영역과 <span style="color:red">**Metaspace**</span> 영역으로 볼 수 있는데, Java 8 버전 이전에는 Metaspace라는 영역은 없었습니다. 기존에 있던 것은 Permanent Area인데, Class의 Meta 정보나 Method의 Meta 정보, Static 변수와 상수 정보들이 저장되었던 공간입니다.

Java 8  버전부터는 이 Permanent Area(PermGen)가 Metaspace로 대체되고, Static 변수와 상수 정보들은 Heap 영역에 저장하도록 변경되었습니다. 고정된 크기가 할당되는 PermGen에 개발자가 Static 변수와 상수들을 저장하면서, OOM(Out of Memory) 문제가 종종 발생하곤 하는 구조적 문제를 해결하기 위해 이러한 변경이 일어나게 되었습니다. 따라서 동적인 새롭게 생겨난 Metaspace는 JVM이 관리하는 Heap 영역이 아닌, OS 레벨에서 관리하는 Native Memory Area로 취급하게 됩니다.

### Heap & GC

![gc_01.jpg](/assets/images/posts/2022-12-11-java-memory-structure/gc_01.jpg){: .align-center}

↪ Shared Memory Area 중 우리가 코드가 작동하면서 메모리에 할당되는 내용을 이해하기 위해 볼 영역은 <span style="color:red">**Heap**</span> 영역입니다. Heap 영역에 동적 할당된 인스턴스가 저장되고, Java에서는 동적으로 할당된 메모리의 반납을 <span style="color:red">**GC**</span>(Garbage Collector)가 담당하기 때문에, GC의 동작과 관련해 이 Heap 영역도 나누어져 있습니다.

1. 처음 객체가 생성되면, <span style="color:red">Eden</span> 영역에 저장됩니다.
2. <span style="color:red">Minor GC</span>가 발생합니다.
   1. Eden 영역에서 살아남은 객체는 <span style="color:red">Survivor</span> 영역으로 이동합니다. (S0, S1)
   2. Survivor 영역에서 살아남은 객체는 Minor GC에서 살아남은 기록인 age bit가 <span style="color:red">MaxTenuringThreshold</span>라는 설정값(-XX:MaxTenuringThreshold)을 초과하기 전까지 S0과 S1을 번갈아 이동합니다.
   3. MaxTenuringThreshold 값을 age bit가 초과한 경우, <span style="color:red">Tenured</span> 영역으로 이동합니다.
3. Major GC가 발생하면, Tenured 영역에서 GC가 일어나게 됩니다.

GC는 대략적으로 위와 같은 과정으로 일어나게 되는데, GC가 일어날 때는 <span style="color:red">**STW(STOP-THE-WORLD)**</span>라는 이벤트가  발생합니다. 이름에서 알 수 있듯이 GC를 실행하는 thread를 제외한 다른 thread들은 모두 작업을 정지하는데, Minor GC보다는 Major GC에 걸리는 시간이 더 길기 때문에 적절한 빈도로 GC가 실행되는 것이 유리합니다.

---

## Heap & Stack
### String

![string_giphy.gif](https://media.giphy.com/media/iFU36VwXUd2O43gdcr/giphy.gif){: .align-center}

↪ 이제 잠시 예시 코드와 함께 Heap과 Stack에서 할당이 어떻게 이루어지는지를 살펴보기 전에 Java의 <span style="color:red">**String**</span>에 대해 짧게 이야기하고 넘어가려고 합니다. 예시에 String type도 나올 예정이기 때문이죠.

Java에서 String 객체를 생성하는 두 가지 방법은 다음과 같습니다.

1. String literal (double quotation)을 사용하는 방법
2. `new` 연산자를 사용하는 방법

String도 동적으로 할당되는 객체이기 때문에 당연히 new 연산자를 통해 Heap 영역에 메모리를 할당 받아 인스턴스를 만들 수 있을 겁니다. 그러면 String literal을 사용하는 방법은 뭐가 다른 걸까요?

Heap 영역 내부에는 <span style="color:red">**Constant Pool**</span>이 존재합니다. String literal, 그러니까 큰 따옴표를 사용해 String 객체를 만들게 되면 이 곳에 저장된 문자열을 참조하게 됩니다. 그리고 여기에 생성된 문자열과 동일한 문자열을 가진 또 다른 String 객체를 만들게 되면, 그 객체도 이 문자열을 참조하게 됩니다.

다시 말하자면, String literal이 아니라 `new` 연산자를 통해 String 인스턴스를 생성했다면 같은 문자열 값을 가진 String 객체라도 서로 다른 참조값을 가졌겠지만, String literal을 통해 같은 문자열 값을 가진 String 인스턴스를 여러개 만들었다면 모두 같은 문자열을 참조하게 됩니다. (`new` 연산자를 사용하더라도, <span style="color:red">**intern**</span> 메소드를 통해 Constant Pool을 바라보게 할 수는 있습니다.)

덧붙이자면, Constant Pool 내부에 저장된 문자열은 수정이 불가능합니다. 따라서 `String str = "앞" + "뒤";`와 같은 연산이 이루어지게 되면, `"앞", "뒤", "앞뒤"`라는 세 가지 String이 Constant Pool 내부에 만들어지게 됩니다. 당연히 이후에 `"앞"`과 `"뒤"`가 쓰일 일이 없다면, GC의 타겟이 될 겁니다. 이런 이유 때문에 문자열 수정이 빈번하게 일어나는 경우 <span style="color:red">**StringBuilder**</span>를 사용해 문자열을 다루어야 성능에 문제가 생기지 않습니다.

Java 5 버전 이후로 \+ 연산이 내부적으로 StringBuilder를 사용하긴 하지만, 매번 새 인스턴스를 생성해 append()를 호출하고 String을 반환하기 때문입니다.

### Example Code & Visualization

![wtf_giphy.gif](https://media.giphy.com/media/xL7PDV9frcudO/giphy.gif){: .align-center}

↪ 아마 지금까지 다룬 내용에 대해 딱히 신경을 쓰고 있지 않던 분들은 여기까지 읽고 이런 표정일 겁니다.

그래서 좀 더 직관적으로 이 내용들을 받아들일 수 있게 간단한 예시 코드와 함께 코드가 한 라인 실행될 때 메모리 할당이 어떻게 이루어지는지 살펴볼 겁니다.

### Code

```java
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<Comics> myComicsList = new ArrayList<>();

        myComicsList.add(new Comics("Walking Dead", 1));
        myComicsList.add(new Comics("Walking Dead", 2));
        myComicsList.add(new Comics(new String("Walking Dead"), 3));

        printMyComics(myComicsList);
    }

    public static void printMyComics(ArrayList<Comics> _comicsList) {
        _comicsList.forEach(_rec -> {
            _rec.setTitle(_rec.getTitle() + " " + _rec.getEpisode());
            System.out.println(_rec.getTitle());
        });
    }
}

class Comics {
    private String title;
    private int episode;

    // constructor
    public Comics(String _title, int _episode) {
        this.setTitle(_title);
        this.setEpisode(_episode);
    }

    // getter & setter
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public int getEpisode() { return episode; }
    public void setEpisode(int episode) { this.episode = episode; }
}
```

↪ 위와 같은 코드가 실행될 때의 메모리를 살펴보겠습니다.

### Visualization

![stack_and_heap_1.jpg](/assets/images/posts/2022-12-11-java-memory-structure/stack_and_heap_1.jpg){: .align-center}

↪ 엔트리 포인트인 main함수의 parameter 변수를 stack에 저장하면서 프로세스가 시작됩니다. 한 단계씩 그림을 나누어 표현하면 지나치게 글이 길어질 것 같아 몇 단계씩 묶어 표현하겠습니다.

![stack_and_heap_2.jpg](/assets/images/posts/2022-12-11-java-memory-structure/stack_and_heap_2.jpg){: .align-center}

**1\.** 지역 변수인 myComicsList는 Comics type의 ArrayList 인스턴스를 생성해 주소값을 저장합니다.

**2\.** Comics의 새로운 인스턴스가 생성되고, "Walking Dead"라는 문자열이 <span style="color:red">**Constant Pool**</span>에 없기 때문에 새롭게 할당되어 Comics의 property인 title이 이를 바라봅니다.  
ArrayList에 첫번째 요소가 이번에 생성된 Comics 인스턴스를 바라봅니다.

**3\.** Comics의 새로운 인스턴스가 생성되고, "Walking Dead"라는 문자열이 <span style="color:red">**Constant Pool**</span>에 있기 때문에 이번에 생성된 Comics의 property인 title이 이를 바라봅니다.  
ArrayList에 첫번째 요소가 이번에 생성된 Comics 인스턴스를 바라봅니다.

![stack_and_heap_3.jpg](/assets/images/posts/2022-12-11-java-memory-structure/stack_and_heap_3.jpg){: .align-center}

**4\.** `new` 키워드를 통해 <span style="color:red">**String 인스턴스**</span>가 생성됩니다. Comics의 새로운 인스턴스가 생성되어 property인 title이 이 String 인스턴스를 바라봅니다.

![stack_and_heap_4.jpg](/assets/images/posts/2022-12-11-java-memory-structure/stack_and_heap_4.jpg){: .align-center}

**5\.** printMyComics 메소드를 호출하면서 parameter _comicsList가 Stack에 push 되고, myComicsList가 바라보던 인스턴스를 바라봅니다.

**6\.** ArrayList의 forEach 메소드 내부에서 화살표 함수로 정의한 익명함수의 parameter _rec가 _comicsList가 바라보는 ArrayList의 첫 번째 원소를 바라봅니다.  
"Walking Dead 1"이라는 문자열이 <span style="color:red">**Constant Pool**</span>에 없기 때문에 새롭게 할당되어 Comics의 property인 title이 이를 바라봅니다.

**7\.** ArrayList의 forEach 메소드 내부에서 화살표 함수로 정의한 익명함수의 parameter _rec가 _comicsList가 바라보는 ArrayList의 두 번째 원소를 바라봅니다.  
"Walking Dead 2"이라는 문자열이 <span style="color:red">**Constant Pool**</span>에 없기 때문에 새롭게 할당되어 Comics의 property인 title이 이를 바라봅니다.

**8\.** ArrayList의 forEach 메소드 내부에서 화살표 함수로 정의한 익명함수의 parameter _rec가 _comicsList가 바라보는 ArrayList의 세 번째 원소를 바라봅니다.  
<span style="color:black;background-color:#ff6666">String의 더하기 연산에 쓰이고 있는 피연산자가 모두 **literal**이 아니기 때문에, **Constant Pool**에는 변화가 없습니다.</span>  
Comics의 property인 title이 바라보던 String 인스턴스의 값을 "Walking Dead 3"로 저장합니다.

---
