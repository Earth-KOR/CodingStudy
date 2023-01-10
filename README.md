# CodingStudy
공부자료보관용

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
