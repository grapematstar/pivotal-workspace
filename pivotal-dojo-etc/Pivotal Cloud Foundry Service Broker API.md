## Pivotal Cloud Foundry Service Broker API

### 1. Service Broker API Architecture
- last update PAS 2.6 
- Service Broker API란 Pivotal Cloud Foundry의 Component Cloud Controller API와 연동하여 Application과 Service를 연동하기 위한 구성 한 API
- Service Broker는 Service Plan Catalog를 제공하고 Application에 Bind 할 수 있도록 Service Instance를 생성 , Application Bind/UnBind 등에 대한 API를 제공 한다.

[Architecture][https://docs.pivotal.io/pivotalcf/2-4/services/overview.html](https://docs.pivotal.io/pivotalcf/2-4/services/overview.html)

1. CF CLI를 통해 Pivotal Cloud Foundry의 Cloud Controller에 Service create/bind/unbind/delete를 요청 한다.
2. CF CLI를 통해 등록한 Service Broker/Service Instance의 정보는 Pivotal Cloud Foundry Database cc_db에 저장한다.
3. Service Broker API는 Cloud Controller의 요청을 받아 Back End Service Instance에 적용 시킨다.

#### 1.1 Service Broker의 구현
- Service Broker는 Application 또는 Http/Https Endpoint를 사용하여 구성 된다. 
- 아래는 Service Broker의 구현 방법이다.
	- Bosh를 통해 Deploy 한 전체 서비스
	- Pivotal Cloud Foundry와 함께 Package하고 배포 한 Service Broker
	- Pivotal Cloud Foundry를 통해 배포한 Application
	- 배포하는 Manifest에 Broker를 포함하여 배포 한 Service

