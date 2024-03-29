# Go lang

> 2009년 11월에 구글에서 처음 발표된 후 2012년 3월에 정식 발표된 프로그래밍언어

### 장점

컴파일러의 속도가 굉장히 빨라서 인터프리터 언어처럼 쓸 수 있다. 이는 문법 구조를 개선함으로써 달성하였다. 또한 코드 역시 간결하면서도 컴파일 언어답게 높은 성능을 낼 수 있다.

Go를 이용해 프로그램들이 서로 소통하면서 상태를 공유하는 동시성 프로그램을 쉽게 만들 수 있다. 이는 멀티스레딩, 병렬 컴퓨팅 뿐아니라, 비동기성 입출력 또한 포함한다. 예를 들면 이벤트 기반 서버, 데이터베이스나 네트워크 작업과 같이 시간이 많이 걸리는 연산을 하는 동안 다른 일을 하는 것을 만한다.

Go는 GoRoutine이라는 비동기 메커니즘을 제공한다. 각각의 고루틴은 병렬로 동작하며 메시지 채널을 통해 값을 주고받는다. 고루틴은 멀티스레드 메커니즘이지만 자체적인 스케줄러에 의해 관리되는 경량 스레드이며 OS에서 관리하는 경량 스레드보다 더 경량이다. 따라서 고루틴은 CPU코어수와 무관하게 수백, 수천의 고루틴을 작성해도 성능에 문제가 생기지 않는다.

### 단점

바이트코드를 생성하는 언어가 아니므로, 바이너리만 배포할 경우 타깃 머신에 맞춰 각각 컴파일해야 한다. Go 언어의 설계 지향점은 시스템 프로그래밍 언어였지만, 가비지 컬렉션의 부재로 인해 박싱/언박싱이 불필요하게 많이 일어나는 등 C/C++을 대체할 수 있는 언어는 아니다. 실제로 C/C++에 비해 느리고 저수준 시스템 개발에서는 가비지 컬레션과 고루틴을 지원하기 위한 무거운 런타임 등으로 인해 사용이 불가능에 가깝다.

### Go Routine vs Java/C++ Thread

- #### 병행성과 병렬성
  
  병렬성은 실제 작업을 동시에 처리하게 된다. 예를 들면 2개의 작업이 있을때 두개의 CPU가 하나씩 동시에 작업하면 이건 병렬성이다. 하지만 한개의 CPU가 2개의 작업을 동시에 하는것처럼 알아서 스케줄링 하면 이건 병행성이 된다. Go의 특징중에 하나가 병행성이다.

- #### Time-Sharing
  
  Process 또는 Thread를 OS가 변경해주는걸 컨텍스트 스위치(Context Switch) 라고 한다. 문제는 OS의 스케줄링에 따라 컨텍스트 스위치의 오버헤드가 크다. Multi Thread의 대표적인 언어인 C++/Java는 Thread를 많이 사용하면 CPU의 이용률, 메모리 사용량이 급격하게 증가한다. Go는 고루틴이 자체적으로 Time-Sharing을 통해 스케쥴링을 하게 된다. 따라서 고루틴은 Thread에 종속적이지 않고 한개의 쓰레드에서 하나이상의 고루틴이 사용될 수 있게된다.