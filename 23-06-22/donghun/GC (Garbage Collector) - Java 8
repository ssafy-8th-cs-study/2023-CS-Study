## Java Garbage Collection (GC)

Java의 Garbage Collection은 자동 메모리 관리를 위한 기능으로,
malloc과 free등으로 수동 지시를 하지 않더라도 더 이상 참조되지 않는 객체들의 메모리 점유를 해제하는 역할을 한다.

GC는 Java Virtual Machine (JVM)에서 관리되며, Heap 영역의 메모리를 감시한다.

<hr>

먼저 JVM의 동작 흐름을 보자.

1. 소스 파일은 자바 컴파일러(javac)에 의해 바이트 코드화 된다.
1-1. 바이트 코드는 어셈블리어와 다르다. JVM이 이해할 수 있는 전용 저급 언어와 같다.

2. Class Loader가 Class 파일을 읽어들여 메모리에 로드한 뒤 Execution Engine에 의해 실행된다.
2-1. Execution Engine은 로드된 Class 파일의 바이트 코드를 해석하고 실행한다.
2-2. 실행 중에 Runtime Date Area에 실행에 필요한 메모리, 객체, 스레드, 스택 등이 저장된다.

3. Runtime Date Area의 구성은 다음과 같다.
   3-1. Method Area(Deprecated), Heap Area, Stack Area, PC register, Native Method Stack(Metaspace 포함)
      3-1-1. GC는 Heap Area에서 동작한다.
 
 
4. Heap Area는 Young Generation, Old Generation으로 구분된다.
   4-1. Java 7부터 G1GC를 지원하나, Java 8의 Default GC는 ParallelGC이다.
   4-2. Java 9부터 Default GC는 G1GC로 확정되었다. 그러므로, Java 11과 Java 17의 Default는 G1GC이다.
   
   4-3. G1GC 이전의 GC는
      4-3-1. ParallelGC는 이름처럼 Minor GC가 실행되는 동안 Mark, Copy, Sweep이 병렬 스레드 상에서 일어난다.
      4-3-2. 복사 알고리즘 이전에 쓰이던 레거시 방식으로 Bump the Pointer 방식이 있었다.
      4-3-3. 순환 참조 문제로 인해 ref count대신 mark & sweep을 사용한다.
      
   4-4. G1GC는
      4-4-1. Region 단위로 구획이 정해진다.
      4-4-2. 별다른 공간적 지역성 고려 없이 Available한 Region에 Eden, Survivor, Old, Humongous Region이 지정된다.
         4-4-2-1. Humongous Region이란 메모리 영역을 많이 차지하는, 크기가 큰 객체를 할당하기 위해 배정된 Region이다. 전통적인 GC에서는 Humongous 객체를 Young Generation을 거치지 않고 Old Generation에 직접적으로 할당해왔다.

<hr>

## ParallelGC (Java8 Default)

>Minor GC(Young Generation)에서는 병렬 스레드 처리가 진행되지만,<br>Major GC(Old Generation)에서는 병렬 처리로 진행되지 않는다.

5. Minor GC(Young Generation)
   5-1. Young Generation은 Eden, Survivor0, Survivor1로 이루어져 있다.
      - Survivor0 혹은 Survivor1 중 하나는 Copy & Sweep를 위해, 또한 과정 상 반드시 비워져있다. 그렇지 않다면 GC가 정상적으로 동작하지 않고 있는 것이다.
      - Heap 메모리에 처음 적재되는 객체는 Eden 영역에 적재된다. 이 때, Eden 영역보다 크기가 큰 객체라면 곧바로 Old Generation에 적재된다. (G1GC: Humongous Region)
      - Eden 영역, 혹은 Survivor0(1) 영역이 가득 차게 되면 Minor GC가 발생한다.
      - ★참조★가 되고 있는 Reachable 객체는 Mark 되어, 비워진 채 대기중이었던 Survivor 영역으로 이전된다. (예시: (마크된) Eden+Survivor0 -> Survivor1)
      - 기존의 Eden 영역과 Survivor 영역은 Sweep하여 초기화한다.
      - GC에 의해 제거되지 않은 Survivor 내의 객체는 생존 횟수에 따른 age를 부여받는다. 일반적으로 15회 이상 생존 시 Old Generation 영역으로 promote된다. 이 때, Old Generation에 위치할 공간이 모자라다면 Major GC가 발생한다.

6. Major GC(Old Generation)
   6-1. Old Generation은 Young Generation보다 넓은 공간을 가지고 있다. (Java 8 Default: Young 33%, Old 67%)
   6-2. Young Generation에서 Promote를 거쳐 적재된 자주 쓰이는 객체와, Humongous한 크기의 객체가 이 공간에 적재된다.
 
   6-3. Major GC가 발생하는 경우
      - Old Generation의 공간이 부족한 경우
      - Promote 된 객체가 저장될 공간이 부족한 경우
      - Humongous한 객체가 저장될 공간이 부족한 경우
      - 명시적으로 GC를 호출한 경우
      - Old Generation 영역의 객체가 동적으로 변경되어 커지는 경우 (드묾)
   
   6-4. Mark & Sweep & Compaction. Minor GC와 달리 복사 알고리즘을 사용하는 것이 아닌, 
      Mark된 Reachable 객체만을 남기고 Unreachable 객체를 삭제한 뒤 
      Compaction을 통해 파편화된 메모리를 정리하고 연속적인 메모리 블록으로 객체들을 이동시키는 작업을 수행한다.
      
      
<hr>

### 명시적인 GC 호출이 권장되지 않는 이유


명시적인 GC 호출, 즉 System.gc()가 권장되지 않는 이유는 크게 3가지가 있다.

1. STW(Stop The World)는 최소화 될 수록 좋다.

2. Major GC를 호출하는 것이기 때문이다.
   2-1. Major GC는 Minor GC에 비해 10배 이상 느리다.
     - 모든 Heap 메모리 영역에서 Mark & Sweep & Compaction 과정이 일어나기 때문이다.
     - 기본적으로 Young Generation보다 공간이 넓기도 하지만, Old Generation에서 파편화를 정리하는 Compaction 알고리즘은 Copy 알고리즘보다 느리다.
     - Copying 알고리즘은 Reachable 객체만 복사해서 메모리를 정리하면 되는데, Compaction 알고리즘은 객체들을 이동시키는 작업을 수행해야 하기 때문.
     
   2-2. 어차피 GC가 즉시 불러와지지 않는다. JVM이 시스템 상황을 보고 실행 시점을 최적화하기 때문에 의미가 없다.
   
<hr>

다음 포스팅은 Java9부터 Default GC가 된, Java11과 Java17에서도 여전히 Default GC인 G1GC에 대해 기록해 보려고 합니다.
