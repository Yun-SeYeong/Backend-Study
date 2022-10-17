# Java



### JVM 구조와 Java의 실행방식

Java는 OS에 종속적이지 않고 프로그램을 동작시킬 수 있는데 이유는 JVM이 OS를 제어하기 위해 필요한 일을 대신 해주기 때문이다.

Java코드는 JVM에서 인식하지 못한다. 따라서 JVM에서 인식할 수 있는 바이트코드로 변환해야한다. 이때 .class 파일이 된다.

JVM은 클래스 로더, Excution Engine, Runtime Data Area로 구성된다.

클래스로더는 JVM내에 클래스 파일을 로드하고 링크한다.

Execution Engine은 바이트코드를 실제로 실행 가능한 상태로 변환한다. 종류로는 인터프리터, JIT, Garbage Collector가 있다. 인터프리터는 바이트 코드를 명령어 단위로 읽고 실행한다. JIT는 특정 시점에서 바이트 코드 전체를 컴파일하여 실행한다. Garbage Collector는 사용하지 않는 인스턴스를 제거한다.

Runtime Data Area는 프로그램을 수행하기 위해 OS로 부터 할당받은 메모리공간이다.



### Garbage Collection

Java는 메모리를 명시적으로 지정하여 해제하지 않는다. 따라서 GC가 사용하지 않는 메모리를 해제 한다. 이 작업을 할때 JVM에 모든 작업을 멈추게 되는데 이를 stop-the-world라고 한다. 이때 GC를 실행하는 쓰레드 외의 모든 쓰레드는 중단한다. 이를 수동으로 System.gc()를 통해 GC를 실행할 수는 있지만 성능에 매우 큰 영향을 미치므로 사용하지 않아야 한다.



GC는 두가지 전제를 가지고 설계되었다.

- 대부분의 객체는 금방 접근 불가능 상태가 된다.

- 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.

이러한 가설을 weak generational hypothesis라고 한다. 이를 전제로 HotSpot VM은 크게 2개로 물리적 공간을 나누었다. 하나는 Young영역 다른 하나는 Old 영역이다.



#### Young 영역

새롭게 생성되는 객체는 여기에 위치한다. 대부분의 객체는 금방 접근 불가능 하기 때문에 Young영역에 많은 객체가 생성되었다가 사라진다. Young영역에서 객체가 사라질 때 Minor GC가 발생한다고 한다.



#### Old 영역

Young영역에서 살아남은 객체가 여기로 복사된다. 대부분 Young영역보다 크게 할당되며 Young영역보다 GC가 적게 발생한다. 여기서 객체가 사라지면 Major GC가 발생했다고 한다.



#### Permanent Generation 영역

Permanent Generation영역은 Method Area라고도 한다. Class 또는 Method의 Meta정보, Static 변수와 상수 정보들이 저장되는 공간이다. 여기서도 GC가 일어나기도 한다. 이때 Major GC의 횟수에 포함된다.



만약 Old영역에 있는 객체가 Young영역의 객체를 참조하고 있을 때가 있다. 이때 Young영역에 GC가 실행되면 Old영역에 참조된 모든 객체를 참조하지 않고 Old영역에 존재하는 512바이트의 카드테이블에 활성화된 객체정보를 보고 GC대상인지 판별한다.





### Young 영역의 구성

Young 영역은 크게 3가지로 나뉜다.

- Eden

- Survivor (2개)

각 영역의 처리 절차는 다음과 같다.

- 새로 생성한 객체는 Eden에 위치한다.

- GC를 실행하고 살아남은 객체는 Survivor로 이동한다.

- 위 과정을 반복하다 Survivor영역이 꽉 차면 여기서 살아남은 객체를 다른 Survivor로 이동하고 꽉 찬 Survivor영역을 비운다.

- 위 과정을 반복해도 살아남은 객체는 Old영역으로 이동한다.



Young영역에서는 빠른 메모리 할당을 위해 두가지 기술이 적용되어있다.



#### dump-the-pointer

영역의 마지막으로 할당된 객체를 추적한다. 따라서 새로 생성되는 객체에 대해서는 사이즈가 할당가능한지만 확인하고 할당하게된다.



#### TLABs(Thread-Local Allocation Buffers)

Young영역에 할당된 객체가 멀티 스레드에서 참조된다면 참조될 때마다 lock이 걸릴 것이고, 성능이 매우 떨어질 것이다. 따라서 각 쓰레드별로 Eden영역에 작은 덩어리를 가지게 하고 각자 가지고 있는 TLAB에만 접근하게 해서 락없이 메모리 할당을 할 수 있도록 해준다.



### Old영역에 대한 GC

old영역은 기본적으로 데이터가 꽉차면 GC를 실행한다.



#### Serial GC (-XX:+UseSerialGC)

mark-sweep-compact라는 알고리즘을 사용한다. 첫 단계는 객체가 살아있는지 확인하고 mark한다. 두번째는 heap의 앞부분부터 살아있는 것만 남긴다. 마지막은 살아있는 객체들을 heap의 앞부분부터 채워서 영역을 나눈다. 이방법은 메모리가 적고 CPU 코어 개수가 적을 때 적합하다.



#### Parallel GC(-XX:+UseParallelGC)

Serial GC와 알고리즘을 같지만 병렬적으로 처리한다는데 차이가 있다.메모리가 충분하고 코어의 개수가  충분할때 유리하다. ThroughputGC라고 부르기도 한다.



### Parallel Old GC(-XX:+UseParallelOldGC)

Parallel GC와 알고리즘에서 차이가 있다. 이 방식은 mark-summary-compactation 단계를 거친다. summary는 별도로 살아있는 객체를 확인한다는 점에서 Sweep과 다르다. 약간더 복잡한 단계를 거친다.



### CMS GC(-XX:+UseConcMarkSweepGC)

Initial Mark단계가 따로 있다. 이때는 클래스 로더에서 가장 가까운 객체중 살아있는 객체만 찾는 것으로 끝난다. 

다음은 Concurrent 단계로 방금 살아있다고 확인한 객체를 참조한 객체를 따라가면서 확인하다. 이단계에서는 다른 GC외의 다른쓰레드와 같이 작업한다.

다음은 Remark단계로 Concurrent Mark단계에서 새로 추가되거나 끊긴 객체를 확인한다. 

마지막으로 Concurrent Sweep단계로 쓰레기들을 정리한다.



이 방식은 stop-the-world가 짧다. 따라서 응답속도가 매우 중요할 때 CMS GC를 사용한다. Low latency GC라고 부르기도 한다.



하지만 단점도 있다. 메모리와 CPU를 많이 사용하게 되고 Compactation 단계를 기본적으로 제공하지 않는다. 조각난 메모리가 많아 compaction 작업을 실행하더라도 stop-the-world시간이 더 길어질 수 있기 때문에 compaction작업이 얼마나 자주, 오랫동안 수행되는지 확인해야 한다.



### G1GC

G1GC는 Eden, Survivor, Old 영역 모두 존재한다. 하지만 고정된 크기가 아니다. 기본적으로 매모리를 2048로 나누어 매모리를 할당하는데 이를 Region이라고 한다. 각 Region은 Eden, Survivor, Old, Humonogous, Available/Unused 영역으로 사용된다.

여기서 두가지 영역이 추가되었다

- Humonogous: Region크기의 50%를 넘는 크기의 객체를 저장하기 위한 공간

- Available/Unused: 아직 사용되지 않은 공간



FullGC는 IHOP에서 정한 수치를 넘기면 실행하게된다.

- Initial Mark: Old Region에 존재하는 객체들이 참조하는 Survivor영역의 객체를 찾는다.(STW)

- Root Region Scan: 위에서 찾은 Survivor객체를 스캔한다.

- ConcurrentMark: 전체 Heap에서 GC대상 객체를 선별하고 이후 과정을 이어간다.

- Remark: 최종적으로 GC할 대상을 식별한다. (STW)

- Cleanup: 살아있는 객체가 가장 적은 Region에 대한 미사용 객체를 제거한다.

- Copy: compaction을 수행한다.



### JDK8 과 JDK7비교



java7

```
<----- Java Heap ----->             <--- Native Memory --->
+------+----+----+-----+-----------+--------+--------------+
| Eden | S0 | S1 | Old | Permanent | C Heap | Thread Stack |
+------+----+----+-----+-----------+--------+--------------+
                        <--------->
                       Permanent Heap
S0: Survivor 0
S1: Survivor 1
```



java8

```
<----- Java Heap -----> <--------- Native Memory --------->
+------+----+----+-----+-----------+--------+--------------+
| Eden | S0 | S1 | Old | Metaspace | C Heap | Thread Stack |
+------+----+----+-----+-----------+--------+--------------+
```



변경된 점은 Parm영역에서 static 객체와 상수들이 Heap영역으로 이동하고 기존 Meta정보들이 Native영역으로 이동하였다. 따라서 meta데이터에 대해서는 os가 관리하도록 하고 static 객체와 상수들은 Heap영역으로 관리되어 GC대상에 최대한 포함 할 수 있도록 했다. 또한 metaspace의 공간의 제약을 없앴다고 할 수 있다.

























#### Reference

- [NAVER D2](https://d2.naver.com/helloworld/1329)[NAVER D2](https://d2.naver.com/helloworld/1329)


