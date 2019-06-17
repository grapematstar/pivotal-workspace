
# 1. Pivotal Cloud Foundry 내부 구조의 이해

## 1.1. Pivotal Cloud Foundry 의 컨셉
              
![cf-cencept][cf-image-0]

-  사용자에게 플랫폼의 확장성과 개방성을 제공하여 기존의 레거시 시스템에서 보다 빠르게 Application을 배포/관리 할 수 있다.
- Bosh를 통해 Pivotal Cloud Foundry VM을 생성/관리하고 Cloud Controller를 통해 Application의 LifeCycle을 관리 한다.
- UAA OAuth2를 사용하여 플랫폼에 대한 사용자의 접근 제어/권한 제어가 가능하다.
- HTTP/HTTPS 통신과 NATS 메세징 채널 통신을 지원하고 내부 DNS를 통해 VM 사이의 통신이 이루어져 IP가 바뀌더라도  통신이 가능하다.
- Loggregator를 사용하여 모든 VM의 Syslog를 출력하며 Nozzle/Firehose를 통해 내부 모니터링/경고 trigger 등 을 설정 할 수 있다.
- ORG/SPACE를 통해 namespace 형식의 플랫폼의 역할을 독립적으로 구분 할 수 있다.
- Application 구동 시 연동 Database 또는 타 Brand의 API를 호출 할 수있는 사용자 제공 Service API를 제공 한다.

## 1.2. Pivotal Cloud Foundry 아키텍처
              
![cf-arc][cf-image-1]

- Router:  System Doamin(UAA, CC,  LOGIN, SSH, Apps)에 대한 요청을 수신하는 Component
- Uaa: Pivotal Cloud Foundry에 접근하는 사용자에 대한 정보와 Role을 제공하는 Component
- Cloud Controller: Pivotal Cloud Foundry는 사용자의 Rest API/CLI 요청을 받아 처리 하는 Component
- Database: Cloud Controller API에 대한 정보를 가지고 있는 Mysql/Postgres Component
- BlobStore: Pivotal Cloud Foundry에서 배포 한 Application의 메타 데이터(Buildpack, Source Code 등) 정보를 가지고 있는 Component
- Nats:  Pivotal Cloud Foundry VM 사이의 내부 통신 메세지 채널 Component
- Diego Cell: Droplet을 통해 생성한 Application 정보를 바탕으로 Garden Container에 실제 Application을 배포 하는 Component
- Loggregator: Pivotal Cloud Foundry VM 과 배포 한 Application의 Metric, Container Log를 제공하는 Component
- BBS: Diego Cell의 현재 상태와 목표 상태 주기를 확인 하고 저장하는 Database Component

## 1.3. Pivotal Cloud Foundry 주요 구성 요소 상세

### 1.3.1. Pivotal Cloud Foundry Diego

![diego-arc][cf-image-2]

- Pivotal Cloud Foundry는 Application을 Container에 배포하고 LifeCycle을 관리하기 위해 Diego를 사용 한다.

- Pivotal Cloud Foundry 사하는 Diego의 동작방식은 아래와 같다.
------------------------------------------------
1. Cloud Controller는 Application 배포 명령을 Diego BBS VM에 전달하고 BBS는 Application의 목표 Spec(instance 수, Memory 등)을 저장 한다.

![diego-cell-1][cf-image-3]

------------------------------------------------

------------------------------------------------
2. BBS는 Application의 목표 Spec를 바탕으로 요청을 Diego Brain VM의 Auctioneer 프로세스에 전달 한다.

![diego-cell-2][cf-image-4]

------------------------------------------------

------------------------------------------------
3. Auctioneer는 Diego_Cell VM의 Rep에 Application 배포를 요청하고  Rep는 실제 Container 환경의 Garden Runc의 상태가 정상이면 해당 요청을 승인 한다.

![diego-cell-3][cf-image-5]

------------------------------------------------

------------------------------------------------
4. Rep는 Garden Runc를 호출하여 Container를 생성하고 Garden Runc는 $ cf push, Application 배포 시 생성한 Droplet을 Blobstore VM에서 다운로드하고 Garden Container 환경에서 Application을 실행 한다.

![diego-cell-4][cf-image-6]

------------------------------------------------

------------------------------------------------
5. Diego Cell VM의 route_emitter는 Application의 접근 경로를 Go Router에 등록 시킨다.

![diego-cell-4][cf-image-7]

------------------------------------------------
------------------------------------------------
6. Diego Cell VM 내부 프로세스 Loggreate agent가 Application의 Log와 Metric 정보를Doppler를 통해 Loggregator Component에 전달하여 사용자에게 4333 PORT로 제공 한다.

![diego-cell-4][cf-image-8]

------------------------------------------------

### 1.3.2. Pivotal Cloud Foundry Cloud Controller

 - Cloud Controller는 사용자가 플랫폼에 Access 할 수 있는 REST API EndPoint를 제공하며 Opg/Space/Service/MarketPlace/Buildpack 등에 관련한 정보를 CC_DB Database Table을 통해 관리 한다.
- 사용자가 API 요청을 보내면 Go Router가 Cloud Controller의 주소로 요청을 보내 처리 한다. 

![diego-cell-4][cf-image-9]

-   [V3 API Docs](http://v3-apidocs.cloudfoundry.org/)
-   [V2 API Docs](http://apidocs.cloudfoundry.org/)
 
### 1.3.3. Pivotal Cloud Foundry Loggregator

![diego-cell-4][cf-image-10]

- Pivotal Cloud Foundry는 Loggregator Component를 통해 VM과 Application Log를 Low Latency, Highly scalable, distributed systems 실시간으로 사용자에게 제공 한다.
1. Pivotal Cloud Foundry를 설치하게 되면 설치 Manifest 파일에 Loggregator agent 관련 Properties가 명시 되어 있고 이로 인하여 각 VM 내부에서 실행 되도록 구성되어 있다.
2. Loggregator Agent는 VM이나 Application에서 발생하는 Log를 Doppler Component로 포워딩 한다.
3. Doppler는 임시 받아온 Log를 임시 버퍼에 저장하고 다시 Traffic Controller Component로 전달 한다.
5. Traffic Controller는 사용자가 요청하는 API에 따라 Doppler에서 가공한 Log를 사용자에게 제공 한다.

### 1.3.4. Pivotal Cloud Foundry UAA

![diego-cell-4][cf-image-11]

- Pivotal Cloud Foundry Uaa는 플랫폼에 대한 ID/PASSWORD 인증에 대한 보안을 제공한다.
- 제공하는 인증 방법은 아래와 같다.
	- OAuth 2.0
	- OpenID Connect 1.0
	- SAML 2.0
	- LDAP
	- SCIM 1.0



[cf-image-0]:./images/cfimage-0.png
[cf-image-1]:./images/cfimage-1.png
[cf-image-2]:./images/cfimage-2.PNG
[cf-image-3]:./images/cfimage-3.PNG
[cf-image-4]:./images/cfimage-4.PNG
[cf-image-5]:./images/cfimage-5.PNG
[cf-image-6]:./images/cfimage-6.PNG
[cf-image-7]:./images/cfimage-7.PNG
[cf-image-8]:./images/cfimage-8.PNG
[cf-image-9]:./images/cfimage-9.PNG
[cf-image-10]:./images/cfimage-10.PNG
[cf-image-11]:./images/cfimage-11.PNG
