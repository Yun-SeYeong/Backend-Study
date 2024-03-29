# 시스템 설계(System Design)

### Single-Server Design

단순히 개인 프로젝트용으로 사용될시 괜찮은 방법이 될 수 있다. 트래픽이 적고 잠시 서버가 다운되어도 문제되지 않기 때문이다. 하지만 single point of failure이기 때문에 상업용으로는 사용할 수 없다.

#### 1. Seperating out the database

Database를 분리하여 서버와 Database가 동시에 장애가 발생하는 것을 막을 수 있다. 하지만 이 또한 데이터베이스가 장애가 발생해도 서비스가 동작하지 않고 서버가 장애가 발생해도 동작하지 않는다. 따라서 좋은 방법이라고 할 수는 없다.

#### 2. Vertical scaling

더 많은 트래픽을 감당하기 위해 Server의 사양을 업그레이드한다. 하지만 업그레이드에는 한계가 있다. 장점은 관리할 서버가 적어 유지보수가 편하다. 하지만 위와 똑같이 장애가 발생하면 해결될 때까지 서비스를 할 수 없다.

#### 3. Horizontal scaling

로드벨런서를 이용해 서버에 트래픽을 분산하고 단일 데이터베이스를 바라보도록 한다. 만약 서버한대가 죽어도 다른 서버로 트래픽을 분산하여 서비스가 항상 동작하도록 한다. 또한 무한 확장이 가능하다. 단점은 유지보수할 서버가 많다는점이다. 또한 이 시스템에서는 stateless 여야 한다. 왜냐하면 각 서버중 어떤 서버로 요청이 갈지 모르고 어떤 서버로 가더라도 서비스가 정상 동작해야하기 때문이다.

여기서 어떤 시스템이 정답이라고는 할 수 없다. 만약 규모가 작은 서비스를 만들때 horizontal scaling은 많은 서버를 관리해야하고 비용도 많이 들어가게 되어서 좋은 방법이 될수 없고 Youtube나 Google에서 vertical scaling은 치명적인 장애로 이어질 수 있다. 따라서 상황에 맞는 System-Design을 해야한다.

### Failover servers: Cold Standby

데이터베이스가 분리된 웹서비스에서 데이터베이스에 장애가 발생을 대비하기 위해 주기적으로 백업을 하고 장애가 발생하면 해당 내용을 다른 데이터베이스에 복구하여 서비스를 정성화 한다. 이 방법은 몇가지 단점이 있다. 첫번째는 다운 시간이 길어진다. 백업한 데이터를 다시 복구해야하기 때문에 데이터양에 따라 시간이 더 오래걸릴 수 도 있다. 두번째는 백업 이후 데이터는 복구할 수 없다. 주기적으로 수집하기때문에 수집 주기에 따라 일부 데이터손실을 용인해야한다. 한가지 장점은 저렴하다.

### Failover servers: Warm Standby

데이터베이스가 분리된 웹서비스에서 데이터베이스를 Replication하여 상시 백업하는 방식이다. 보통 데이터베이스마다의 자체 복사 메커니즘이 있어서 데이터베이스가 자동으로 관리한다. 데이터 손실이 일어날 수 있는 작은 여지는 있지만 주기적 백업보다는 훌륭한 접근법이다. 만약 장애가 발생하면 백업 데이터베이스로 리다이렉션한다 따라서 거의 데이터 손실이 없고 다운 시간도 짧다. 하지만 아직 수직 스케일링을 하고 있다. 트래픽이 증가하면 더 큰 데이터베이스로 변경하는 방법밖에 없다.

### Failover servers: Hot Standby

데이터베이스가 분리된 웹서비스에서 백엔드가 모든 데이터베이스에 쓰는 구조로 한다. 이 방법은 수평 스케일링에 가까워 졌다고 볼 수 있다. 장애가 발생하여도 데이터를 읽을 수 있고 트래픽의 분산도 할 수 있다.

### Horizontal Scaling of Databsases: Sharding

![](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/system-design/1.png)

대략적인 현대의 확장 가능한 데이터베이스 설계는 위 구조이다. 기본적으로 위 Router 같은 것을 통해 데이터를 어떤 샤드에 저장할지 결정하고 각 샤드에 분산하여 저장한다. 각 샤드는 하나 이상의 백업을 가질 수 있다. 따라서 수직, 수평 스케일링의 장점을 모두 가져온 것이다. 만약 샤드가 죽어도 각 샤드의 백업으로 대체할 수 있고 데이터 또한 해쉬알고리즘을 통해 분산하여 저장하게 된다. 

### More Specific Example: MongoDB

Mongo DB는 기본적으로 비관계 데이터베이스로 NoSQL기반이다.

Mongo DB는 애플리케이션 서버 (Fleet)에서 실행된다. 그리고 mongos라는 프로세스들을 가지고 있다. mongos는 replica set에 데이터를 배포하는 일을 한다. mongo set은 샤드와 같다.  샤드에는 주서버와 여러개의 보조 서버가 있다. 주서버는 라우터 역활을 하는데 보조 서버들을 구성하고 저장하는 방법을 결정한다.  주 서버가 죽어도 보조 서버는 새로운 주 서버를 찾게 된다. 따라서 주서버에서 single point of failure 처럼 보이지만 장애시 자동으로 교체되는 메커니즘을 가지고 있다. primary set은 secondary set의 접속정보등을 모두 알아야한다. 이때 필요한게 Config Server이다. 



### More Specific Example: Cassandra

Mongo DB처럼 서버를 관리하는건 언정적으로 서비스할 수 있지만 서버 비용이 많이 발생한다. cassandra는 이를 조금 다르게 해결한다. 카산드라는 링시스템인데 데이터를 여러 노드에 나누어 저장한다. 노드는 다중화 목적을 위해 여러개의 노드로 복제된다. 특이한건 모든 노드가 API 포인트가 된다. 단점은 모든 노드가 최신상태를 유지하기 위해 모든 노드에 복사해야한다. 어떤 노드에 데이터를 쓰고 다른 노드에서 읽을려면 시간이 걸리게된다.



### Sharded databases are sometimes called 'NoSQL'

#### 단점

- Shard를 통해 구성한 데이터베이스는 서로 다른 샤드간 데이터를 결합하는게 어렵다.

- 샤드를 추가할때 resharding을 해야한다.

- 일부 많은 트래픽이 몰리는 Hotsopts 문제가 있다.



### Data lakes

log파일과 같이 비정규화 된 데이터를 유저에게 제공할 일이 있다. 이때 데이터를 가공하고 쿼리하는 등 작업을 해야하는데 AWS에서는 S3, Glue, Athena, red shift를 이용하여 정규화하고 쿼리할 수 있다.



#### ACID

데이터를 쓰거나 수정할때 하나의 트렌젝션안에서 일어나는데 이때 항상 유지되어야하는 규칙이다.



### CAP Theorem

Cap 이론에서 가장 중요한건 일관성, 가용성, 분할 허용성이다. 이중 가장 중요하게 생각되는 두가지를 선택하게 된다.  MySQL은 다른 Mongo DB와 같이 샤드를 구성하는 서버보다 가용성이 좋고, 지속적으로 같은 데이터를 배포할 필요없기 때문에 지속성 또한 좋다.  하지만 수평 확장이 어렵다. 현대 제공되는 데이터베이스들은 이 3가지를 모두 제공하는 경우가 많다 CAP이론이 많이 약해졌다.



### Caching

디스크 Hit는 굉장히 리소스가 크다 따라서 디스크히트를 최소화 하는것이 중요하다. 만약 다음과 같이 로드벨런서를 이용해 데이터를 분산하고 하나의 DB에 접근한다고 한다. 따라서 트래픽이 몰리면 DB가 굉장히 부하가 강하게 된다. 이때 해결할 수 있는 방법이 Chaching layer를 만드는 것이다. 



#### LRU (Least Recently Used)

가장 오래된 것을 제거하는 방법으로 최근 사용한 항목을 linked-list의 HEAD에 넣게 되는 방식이다. 이렇게 되면 가장 최근사용한 항목이 제일 빠르게 조회되며 사용하지 않는 항목을 뒤로 밀리게된다.



#### LFU (Least Frequently Used)

일정 기간에 걸쳐 얼마나 Hit되었는지 보고 Hit가 적을걸 제거한다. 대규모 환경에서 큰 항목에 대해서는 LRU가 적절할 수 있지만 캐시 항목이 작고 꼼꼼히 체크해야한다면 LFU가 좋은 선택이 될 수 있다.



#### FIFO

말그대로 선입선출이다.



### A Few Caching Technologies



#### Memcached

memcached는 매우 단순한 in-memory key/value 저장소이다. 매우 오래동안 사용되어 검증되었고 API가 단순하다. 



#### Redis

기능이 다양하고 많은 사람들이 사용하면서 검증이 되었고 스냅샷이나 복제 능력, 트랜잭션 지원, pub/sub등 다양한 기능이 있다.



#### ElasticCache

AWS에서 제공하는 캐시로 AWS로 시스템을 구성한다면 고려해볼 수 있다.



### CDN

다른 나라에 컨텐츠를 전송하려면 많은 지연이 걸리게된다. 이때 필요한 나라에 Server를 두고 컨텐츠를 미리 캐시해두면 해당 데이터를 빠른 속도로 전달 받을 수 있다. 원본 컨텐츠와 연동 과정이 필요하기 때문에 주로 정적 컨텐츠를 사용하게된다.



### Resiliency

만약 IDC전체가 물에 잠기거나 화제가 발생하는등 문제가 발생했을때 어떻게 대처할지 생각할 필요가 있다. 데이터 센터를 가용영역이라고 부르기도한다. 물리적인 데이터 센터의 개념을 요약하는 것인데, 최소한 하나의 개체라고 생각한다. 보통 Geo-Routing을 통해 지역에 따라 적절한 데이터 센터로 트래픽을 전달한다. 이렇게하면 복원력을 가질 수 있다. 한 지역에서 장애가 발생하면 다른 지역으로 다시 라우팅 하면 되기 때문이다.



### HDFS architecture

HDFS는 하둡의 분산 파일 시스템으로 오픈소스 시스템이다. HDFS는 파일을 일정한 크기의 블록으로 나눈다. 그리고 해당 블록을 전체 클러스터에 복제한다. 이때 어디에 블록을 저장할지 결정하는게 name node이다. 클라이언트가 특정 데이터를 요청하면 name node를 통해 클라이언트로 부터 가장 가까운 사본이 어디있는지 알려준다.  여러개의 name node가 필요하거나 name node도 블록의 위치등을 기억해야하는데 이를 메타데이터 스토어에 저장하게된다.



### TF-IDF: Document search

주어진 문서중에서 가장 관련성이 높은 단어를 찾는 방법이다. 검색시 a, the 같은 단어가 주 된 키워드로 찾으면 안된다. TF-IDF는 이런 문제를 해결한다. 주어진 문서에서 키워드가 얼마나 자주 나타났는지 뜻하는 Term Frequency와 전체 문서에서 키워드가 얼마나 자주 나타났는지를 나타내는 Document Frequency를 나누어 단어가 얼마나 중요한지 나타낸다. 이 방법은 데이터가 방대한 곳에서는 사용하기 어렵다.



### PageRank: Searching the Web

PageRank는 해당 페이지에 내용이 아닌 얼마나 link되었는지를 본다. 



### Message Queue

메세지 큐는 pub/sub구조로 서로 다른 시스템 간의 데이터를 스트리밍 할 수 있다. 또한 subscriber가 처리가 늦어져도 Queue에 데이터를 저장되어 있어 문제 없이 처리 될 수 있다.



### Design a Data Warehouse for Log Data with AWS

![2.png](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/system-design/2.png)



먼저 Kinesis를 통해 데이터 구조화하여 S3에 스트리밍한다. AWS에서는 해당 데이터를 Glue를 통해 데이터를 SQL을 통해 확인한다. 이를 Athena에서 대화형으로 쿼리할 수 있게된다. 이부분은 Serverless로 AWS를 이용하여 구성하였다. 

위와 비슷한 구조로 Redshift를 통해 데이터를 조회하고 QuickSight를 통해 차트를 그릴 수 있다. 
