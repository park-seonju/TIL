## Server_Scalability

## Index

- Server
- Bound
- Web Server
- DB
- CDN
- 분산시스템

### 1. Server

#### (1) 1U

- Application
- Storage
- 1U의 크기는 1.75인치(약 4.45cm)

#### (2) 유형

- 랙 마운트형데이터 센터나 서버 룸에 설치된랙 안에 들어간다.19인치 랙에 수용하는 것을 전제로 한다.1U(1.75인치), 2U(미들 레인지 서버 이상) 유닛 단위
- 타워형사내 서버 룸에 설치되거나 사무실 또는 점포 등에도 설치

### 2. Bound

#### (1) CPU Bound

- 블록체인 해싱 연산 부하
- CPU 개수를 늘리면서 해결
- 프로세스 진행 속도가 CPU 속도에 의해 제한됨
- 작은 숫자를 곱하는 것과 같이 작은 숫자 집합에서 계산을 수행하는 작업은 CPU에 바인딩

#### (2) I/O Bound

- Input Output
- 네트워크 혹은 디스크 사용량 문제
- 프로세스가 진행되는 속도가 I / O 하위 시스템의 속도에 의해 제한됨을 의미

#### (3) 메모리 바운드

- 프로세스가 진행되는 속도가 사용 가능한 메모리 양과 해당 메모리 액세스 속도에 의해 제한됨

   

  많은 양의 메모리 데이터를 처리하는 작업 

#### (4) 캐시 바운드

- 프로세스 진행률이 사용 가능한 캐시의 양과 속도에 의해 제한되는 비율
- 캐시에 맞는 것보다 더 많은 데이터를 단순히 처리하는 작업은 캐시에 바인딩된

### 3. Web Server

> Web Server 늘리는 경우

#### (1) Scale-up

- 서버의 자체 성능을 증가
- 기존의 서버에서 더욱 고성능의 서버로 변경
- 수직 스케일

#### (2) Scale-out

- 기존의 서버와 같은 사양 또는 비슷한 사양의 서버 대수를 증가시키는 방법
- 수평 스케일
- 확장이 스케일 업보다는 다소 유연

#### (3) Load Balancer

- 트래픽을 받아서 여러 대의 서버에 분산시키는 하드웨어 또는 소프트웨어
- L4, L7 SwitchHAproxy, nginx
- 분산 방식Round Robin (가장 흔히 많이 사용되는 경우)Hash부하가 적은 쪽
- Health Check

#### (4) Auto Scailing

- 클라우드의 유연성을 돋보이게 하는 핵심기술
- CPU, 메모리, 디스크, 네트워크 트래픽과 같은 시스템 자원들의 메트릭(Metric) 값을 모니터링하여 서버 사이즈를 자동으로 조절
- AWS의 오토스케일링

### 4. DB

> DB를 늘리는 경우 

#### (1) Master - Slave

- 실시간 복제
- Write : Master
- Read : 웹 서버와 거리가 가까운 곳 Slave
- 리눅스는 마스터를 메인(main)이나 프라이머리(primary)로, 슬레이브를 세컨더리(secondary)나 레플리카(replica) 등으로 바꾸는 방안을 고려 중

#### (2) Sharding

- 하나의 거대한 데이터베이스나 네트워크 시스템을 여러 개의 작은 조각으로 나누어 분산 저장하여 관리
- Join 을 하기 위해서는 비정규화 수행

#### (3) Proxy

- HAproxy(High Availability Proxy)TCP와 HTTP 기반 어플리케이션을 위해 사용오픈소스 로드밸런싱의 표준 중 하나이며, 다양한 리눅스 버전에서 사용
- nginx트래픽이 많은 웹 사이트를 위해 네트워크 확장성을 주목적으로 설계한 경량 HTTP 서버리버스 프록시로도 활용이 가능

#### (4) 캐시 Layer

- 일반적으로 DB 앞에
- Read : 캐시
- Write : back
- Memcached, Edis

### 5. CDN

- 지리적으로 분산된 여러 개의 서버
- 웹 콘텐츠를 사용자와 가까운 곳에서 전송함으로써 전송 속도 향상
- 전 세계 데이터센터는 파일 복사본을 임시로 저장하는 프로세스인 캐싱을 사용
- 사용자는 가까운 서버를 통해 웹 활성화 디바이스 또는 브라우저에서 인터넷 콘텐츠에 빠르게 접속
- CDN은 웹 페이지, 이미지, 비디오 등의 콘텐츠를 사용자의 물리적 위치와 가까운 프록시 서버에 캐싱
- 콘텐츠가 로딩될 때까지 기다릴 필요 없이 영화 감상, 소프트웨어 다운로드, 은행 잔고 확인, 소셜 미디어 포스팅, 구매 등의 작업 가능

### 6. 분산시스템

- 주어진 작업을 여러 노드에서 나누어 수행
- Consensus Protocol 웹서버끼리 일을 수행하기 전에 합의
- 장애는 일상
- 모든 과정에서 자동화는 필수
- 시스템의 가시성 확보
- Scale-out 보다는 Elastic 유연하게