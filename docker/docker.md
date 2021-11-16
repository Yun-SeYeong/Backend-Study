# Docker

Docker는 Go언어로 작성된 리눅스 컨테이너 기반으로하는 오픈소스 가상화 플랫폼이다.



### VM 가상화 플랫폼 vs Docker 가상화 플랫폼

VM은 Host OS 위에 Hypervisor Engine을 올리고 그위에 Guest OS를 올려 사용한다. 이는 가상화된 하드웨어 위에 OSrㅏ 올라가는 형태로 Host와 거의 완벽히 분리된다. 하지만 Docker는 도커 엔진위에 Application 실행에 필요한 바이너리 파일만 올리게 된다. 따라서 VM은 Host OS와 완전히 분리되지만 무겁고 느릴 수 밖에 없다. 또한 Docker는 Host OS와 커널을 공유하기 때문에 IO처리 효율이 좋다.



### Docker Image

Docker Image는 컨테이너를 실행시키기 위한 실행파일, 설정 값들을 가지고 있다. 만약 ubuntu 이미지를 만들기 위해 Layer A,B,C 가 있다. 그럼 nginx컨테이너를 만들때는 Layer A,B,C 를 베이스로 nginx만 더하여 만들게 된다. 여기에 web app을 더하려면 그 위에 web app만 올려서 이미지를 만들면 된다.



### Docker Container

Docker Container는 image를 실행한 형태이다. 컨테이너는 API 또는 CLI를 통해 create, start, stop, delete를 할 수 있다.  컨테이너에 네트워크를 연결하거나 스토리지를 연결하거나 그위에 새로운 이미지를 생성도 할 수 있다. 기본적으로 컨테이너는 Host machine과 꾀 잘 분리되어있다. 네트워크, 스토리지 등을 host machine으로부터 분리할 수 있다.



### Container의 작동 원리

컨테이너는 cGroup과 namespace와 같은 리눅스 커널 기반의 기술을 이용하여 프로세스를 완벽히 격리된 환경으로 만들고 실행한다.



### cGroup

cGroup은 각 Host OS로 부터 공유할 수 있는 리소스에 대해 공유하거나 제한할 수 있게 한다. 이는 호스트의 리소스에 서비스 거부 공격(denial of service attacks)을 방지할 수 있어 리소스의 활용과 보안에 있어 중요하다. Container는 미리 정의된 제약조건에 의해 CPU와 메모리를 공유할 수 있다.

- cGroup의 서브시스템
  - [blkio](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/6/html/resource_management_guide/ch-subsystems_and_tunable_parameters#sec-blkio) - 물리 드라이브 (예: 디스크, 솔리드 스테이트, USB 등)와 같은 블록 장치(Block Device)에 대한 입력/출력 액세스에 제한을 설정한다.
  - [cpu](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/6/html/resource_management_guide/sec-cpu) - CPU에 cgroup 작업 액세스를 제공하기 위해 스케줄러를 사용한다.
  - [cpuacct](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/6/html/resource_management_guide/sec-cpuacct) - cgroup의 작업에 사용된 CPU 자원에 대한 보고서를 자동으로 생성한다.
  - [cpuset](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/6/html/resource_management_guide/sec-cpuset) - 개별 CPU (멀티코어 시스템에서) 및 메모리 노드를 cgroup의 작업에 할당한다.
  - [devices](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/6/html/resource_management_guide/sec-devices) - cgroup의 작업 단위로 장치에 대한 액세스를 허용하거나 거부한다.
  - [freezer](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/6/html/resource_management_guide/sec-freezer) - cgroup의 작업을 일시 중지하거나 다시 시작한다.
  - [memory](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/6/html/resource_management_guide/sec-memory) - cgroup의 작업에서 사용되는 메모리에 대한 제한을 설정하고 이러한 작업에서 사용되는 메모리 자원에 대한 보고서를 자동으로 생성한다.
  - [net_cls](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/6/html/resource_management_guide/sec-net_cls) - Linux 트래픽 컨트롤러 (tc)가 특정 cgroup 작업에서 발생하는 패킷을 식별하게 하는 클래식 식별자 (classid)를 사용하여 네트워크 패킷에 태그를 지정한다.
  - [net_prio](https://www.kernel.org/doc/Documentation/cgroup-v1/net_prio.txt) - cgroup의 작업에서 생성되는 네트워크 트래픽의 우선순위를 지정한다.



### namespace

namespace는 운영체제 내에서 프로세스가 가지는 다른 프로세스, 네트워킹, 파일시스템, 사용자 ID 구성 요소 등의 '가시성'을 제한하여 프로세스 상호작용을 따로 구분하는 방식이다.

namespace는 가상화 기술의 Hypervisor와 구조적으로 다르다.

Hypervisor는 Hardware resource를 가상화 하는데 Guest OS에 가상화된 형태의 H/W를 제공하여 완전히 다른 환경으로 분리하게 된다. 하지만 namespace는 동일한 OS와 동일한 kernel에서 작동하게 된다. 단지 각각의 고립된 사용 환경만 제공되는 것이다.

- namepspace 종류
  - UTS namespace : hostname을 변경하고 분할
  - IPC namespace : 프로세스간 통신 격리
  - PID namespace : Process IDf를 분할 관리
  - NET namespace : Network Interface, iptables 등 network 리소스와 관련된 정보를 분할
  - User namespace : user와 group ID를 분할 격리
  - Mount namespace : global file hierarchy 분할







Reference

- https://docs.docker.com/get-started/overview/
- https://blog.dnd.ac/link-docker/
- https://khj93.tistory.com/entry/Docker-Docker-%EA%B0%9C%EB%85%90