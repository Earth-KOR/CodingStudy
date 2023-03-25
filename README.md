# CodingStudy
공부자료보관용

Kafka


* Apache Kafka는 무엇인가?
 -> Event Streaming Platform

* Event Streaming이란 무엇인가?
 -> 대용량의 데이터 (Big Data) 발생
 -> Event Stream은 연속적인 많은 이벤트들의 흐름을 의미

* Event Streaming의 동작 방식
 -> 이벤트가 발생 하였을 때 이벤트 등록
 -> 이벤트를 디스크에 저장
 -> 해당 이벤트를 분석 및 처리

* Event Streaming이 쓰이고 있는곳은?
 -> 실시간 차량 진단
 -> 사기 감지, 고객 행동 패턴 분석
 -> 실시간 추천
 -> 실시간 재고 정보

* Kafka vs Redis vs RabbitMQ
 -> Redis의 경우에는 Event Message를 저장하지 않고 처리함.
 -> RabbitMQ의 경우에는 Message Broker를 구현하여 Queue 형태로 처리함.
 -> Kafka의 경우에는 topic 개념이 들어와서 분산 처리의 형태로 처리함.
 -> 안전성, 성능 면으로 kafka가 우세함!

* 용어 정리
    * Topic -> 메세지가 저장되는 공간 단위
    * Producer -> 메세지를 생산해서 Kafka의 Topic으로 메세지를 보내는 곳
    * Consumer -> Topic의 메세지를 가져와서 처리하는 곳
    * Consumer Group : Consumer들의 집합 하나의 Consumer는 무조건 하나의 Consumer Group에 포함된다.
    * Offset -> Event의 위치를 나타냄.

* KafKa 동작 방식
 -> Producer와 Consumer는 서로 알지 못함. (고유의 속도로 작동)
 -> Consumer Group 끼리도 독립적임.
 -> Producer와 Consumer는 Commit Log(Partition) 안의 데이터를 Write & Read 함.
 -> Commit Log(Partition) 안의 이벤트는 변경 불가능하고 추가만 가능함.
 -> Partition안의 이벤트는 Segment 단위로 물리적으로 저장된다.
 -> Topic 생성 시, Partition의 개수를 지정 할 수 있으며, 미리 만들어진 Broker들에 분산되어 생성된다.
 -> Cluster > Broker > Patition(Topic의 단위들) > Segment
 -> Segment의 단위는 우리가 설정 할 수 있음 (bytes 1GB , hours 168h)
 -> Segment는 맨끝만 활성화 되어 있음. (0으로 돌아가지 않음)

* Broker
 -> Kafka Server라고 부름.
 -> Topic 내의 Partition 들을 분산, 유지
 -> 일부 Partition들을 포함하고 있음.
 -> 하나의 Broker에만 연결하면 Cluster 전체에 연결됨
 -> 각각의 Broker는 모든 Broker, Topic, Partition에 대해 알고있음.  -> Broker는 최소 3대 4대를 권함.
 -> Zookeeper는 Broker를 관리하는 소프트웨어
 -> Zookeeper는 홀수의 서버로 작동하게 설계되어 있음. (3, 5)
 -> Zookeeper는 하나의 Leader와 Follower들로 되어있음.
 -> Zookeeper의 Leader에서 정보를 가지고 있고 이를 바탕으로 복제하여 Follower 생성
 -> Zookeeper는 분산 작업을 제어하기 위한 Tree 형태의 데이터 저장소이다.
 -> Zookeeper를 사용하여 Kafka Broker들 간의 정보를 동기화 할 수 있음.


* Producer
 -> Producer에서 HEAD, BODY에 데이터를 담아서 보내준다.
 -> BODY에 key, value 형태로 데이터를 담아서 보내준다.
 -> Partitioner가 Producer가 보내준 key를 보고 Hash 알고리즘을 통해 Partition에다가 저장한다.
 -> 만약에 key가 null 이면 Default Partitioner 형태로 동작함.
 -> 초기 : Round Robin 후기 : Sticky 정책
 -> 초기에는 Partition 하나씩 보내다가 이제는 하나의 Partition에다가 통신 단위로 저장함.

* Consumer
 -> 각 고유의 속도로 Commit Log로 부터 순서대로 처리
 -> Topic __consumer_offsets 이라는 공간에 읽음처리한것을 저장함.
 -> Consumer Group은 각각 정확히 하나의 Partition에서 Record함.
 -> Consumer는 여러개의 Partition을 받을 수 있음 
 -> Key 선택이 잘 못되면 작업 부하가 고르지 않을 수 있음.
 -> Consumer Group 내의 다른 Consumer가 실패한 Comsumer를 대신하여 Partition에서 데이터를 가져와서 처리함.
 -> 동일한 Key를 가진 메세지를 동일한 Partition에만 전달되어 Key 레벨의 순서 보장

* Replication
 -> Broker가 갑자기 장애나서 고장나면 어떻게 될까?
 -> 그거에 대비해서 각 Partition은 Replication 정책을 사용.
 -> 실제 Write & Read가 일어나는 Leader가 있고, 다른 Broker에 다가 복제품을 심어놓는다.
 -> 실제 활동은 Leader에서 일어나고 다른 복제품들은 동기화를 위해 Leader의 Commit Log에서 데이터를 가져오기 요청을 통해 동기화 한다.
 -> Leader Partition에서 장애가 나면?
 -> 다른 Follower 중에서 새로운 Leader를 선출
 -> 하나의 Broker에 Leader가 몰리면 Hot Spot이 발생함.
 -> 방지하기 위해 Leader들을 분산시킴 (설정값 있음)


* In-Sync Replicas
 -> Leader Partition이 죽었을때 어떤놈을 Leader로 선출할까?
 -> Follwer Partition 중에 제일 잘 따라오고 있는 애를 기준으로 함. (High Water Mark)
 -> replica.lag.mas.messsages=4 기준으로 High Water Mark가 4안에 들어와 있으면 ISR 아니면 OSR
 -> ISR중에 리더를 뽑아야하기 때문에 항상 ISR 리스트를 가지고 있음!
 -> messsages는 위험 왜냐면 갑자기 부하가 심어질떄는 안되기떄문에!
 -> 그래서 time.max.ms로 한다! (패치요청 딜레이 시간)
 -> 이러한 ISR관리는 Leader Broker가 관리함!
 -> Follower가 너무 느리면 ISR에서 해당을 제거하고 ZooKeeper로 전달
 -> Controller가 Partition Metadata에 대한 변경사항을 ZooKeeper로 수신
 -> Controller란 Broker중에 하나가 되고, ZooKeeper를 통해 Broker Liveness 모니터링
 -> Controller는 Leader와 Replica 정보를 Clister내 다른 Broker들에게 전달
 -> ZooKeeper에 Replicas 정보의 복사본을 유지한 다음 더 빠른 엑세시를 위해 모든 Broker들에게 동일한 정보를 캐시함.
 -> Contoller가 Leader 장애시 Leader Election을 수행
 -> Controller가 장애나면 다른 Active Broker들 중에서 재선출
 -> Consumer 관련 Position
 -> Last Committed Offset , Current Position, High Water Mark, Log End Offset
 -> Committed의 의미
 -> ISR 목록의 모든 Replicas가 메세지를 성공적으로 가져옴!
 -> Consumer는 Committed 메세지만 읽을 수 있음
 -> Broker가 다시 시작할떄 Committed 메세지 목록을 유지하기 위해 모든 Partition에 대한 마지막 Committed offset은 replication-offset-checkpoint 파일에 기록된걸 가져옴
 -> Leader Epoch
 -> 새 Leader가 선출된 시점을 Offset으로 표시
 -> Message Commit 과정
 -> Broker위에 Fetcher Thread가 돌고있음  -> Producer가 메세지를 보냄
 -> 각 Follower들의 Fetcher Thread가 독립적으로 fectch를 수행하고 가져온 메세제릴 offset6에 메세지를 씀


Producer Asks
-> 프로듀서가 메세지를 보내고 보낸 요청이 성공 할 때 어떻게 동작하게 할꺼냐?
Asks = 0 -> 에크가 필요없다, 그래서 메세지 손실이 많이 발생할 수있지만 빠름
에크 = 1 => 디폴트 값임 Leader 한테 보내면 프로듀서한테 에크를 보내줌.
 리더가 장애가 발생하면 ? 메세지가 누락될수가 있음
에크 = -1 all => 리더한테 메세지를 보내고 받음, 팔로우가 fetch하고 다하면 그제서야 에크를 보냄!
-> 데이터가 이미 복제된걸 확인하고 에크를 보내기때문에 누락 없음! 근데 속도가 늦음!

Producer Retry

Retries 재시도 횟수
Retry.backoff.ms 재시도 사이에 추가되는 대기시간
Request.timeout.ms  프러듀서가 응답을 기다리는 시간
Delivery.timeout.ms send()후 성공 또는 실패를 보고하는 시간의 상한
retries를 조정하는 대신에 Delivery.timeout.ms 조정으로 재시도 동작을 제어!

Producer Batch

Batch 처리는 RPC 처리를 줄여서 Broker가 처리하는 작업이 줄어들기 때문에 더 나은 처리량을 제공
RecordAccumilator에 저장 ->produceRequest에 담아서 보냄
linger.ms = 0 디폴트 바로바로쏨
Batch.size 16kb
일반적인 설정은 링거.ms로 설정함

Message send 순서보장

Message 보냄 -> 배치로 모아서 보냄 -> (in-filght request)를 전송
모아둔 배치를 쫘아~악 보냄
만약에 배치0번이 실패하면???? -> 배치 1번의 순서는 어떻게 보장?
Enable.idempotence -> 하나의 패치가 실패하면 그 다음 배치가 들어오지만 만약 시퀸스가 안맞으면 실패처리를 함.

Page Cache 와 Flush

메세지는 파티션에 기록됨
파티션은 로그 세그먼트로 이루어져있음
성능을 위해 로그 세그먼트는 OS Page Cache에 기록됨
로그 파일에 저장된 메시지의 데이터 형식은 모두 동일 -> 제로카피가능


1. 서비스의 SPoF 현상을 막아야 한다.

데이터베이스 같은 경우는 DB는 두개로 나누어서 하나가 고장 났을 때 스위칭 가능하도록 설계가능

Web Server 을 이중화 하여 하나가 죽었을 때 다른 하나로 스위치 가능하게 설계 할 수 있음. (Active-Standby)

2. 확장성이란 ?


1. Server-level-Scalability

서버 측 요청 수가 증가에 따라 서버 확장이 용이?

소프트웨어 스택 선정
데이터베이스 설계
시스템 아키텍쳐
배포 관리

1. Code-level-Scalability
새로운 요구사항에 대해 코드 변경이 용이?

언어 선정
테이블 설계
디자인 패턴
버전 관리

서버는 확장이 용이하지만 (아무것도 공유X)
DB는 확장이 어려움(데이터를 공유하기 때문)
=> SAN Storage를 활용하여 확장
(SPoF 의 가능성은 열려있는 상태)

Range Partitioned Table (샤딩)
SAN Storage의 SPoF의 가능성을 극복하기위해
=> 어느 한쪽에만 부하가 몰리면 어떻게 해야?
