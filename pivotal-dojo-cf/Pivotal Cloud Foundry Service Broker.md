# Pivotal Cloud Foundry Service Broker 

## 1. Service Broker Architecture
- last update PAS 2.6 
- Service Broker API란 Pivotal Cloud Foundry의 Component Cloud Controller API와 연동하여 Application과 Service Instance를 연동하기 위한 구성 한 API
- Service Broker는 Service Plan Catalog를 제공하고 Application에 Bind 할 수 있도록 Service Instance를 생성 , Application Bind/UnBind 등에 대한 API를 제공 한다.

[Architecture][https://docs.pivotal.io/pivotalcf/2-4/services/overview.html](https://docs.pivotal.io/pivotalcf/2-4/services/overview.html)

1. CF CLI를 통해 Pivotal Cloud Foundry의 Cloud Controller에 Service create/bind/unbind/delete를 요청 한다.
2. CF CLI를 통해 등록한 Service Broker/Service Instance의 정보는 Pivotal Cloud Foundry Database cc_db에 저장한다.
3. Service Broker API는 Cloud Controller의 요청을 받아 Back End Service Instance에 적용 시킨다.

### 1.1 Service Broker의 구현
- Service Broker는 Application 또는 Http/Https Endpoint를 사용하여 구성 된다. 
- 아래는 Service Broker의 구현 방법이다, 단 Service Broker API코드를 포함하여야 한다.
	- Bosh를 통해 Deploy 한 Service
	- Pivotal Cloud Foundry와 함께 Package하고 배포 한 Service Broker
	- Pivotal Cloud Foundry를 통해 배포한 Application
	- 배포하는 Manifest에 Broker를 포함하여 배포 한 Service

## 2. Service Broker 관리
- Service Broker을 등록하면 Cloud Controller가 Service Broker의 카탈로그 API를 호출하여 카탈로그, 사용자 인증 정보를 cc_db에 저장하여 Service Broker가 호출 될 때 마다 해당 정보들을 사용한다. Service Broker는 Service Instance에 대한 모든 HTTP/HTTPS 요청에 대해 사용자 검증을하고, 검증이 실패 할 경우 Service Instance를 삭제 한다.

### 2.1. Service Broker 등록
```
# PAS 전체에 Service Broker 생성, Service의 Plan이 비공개로 되어 있다.
$ cf create-service-broker mybrokername someuser somethingsecure https://mybroker.example.com
# PAS의 Target Space에 Service Broker 생성, Service Plan이 공개로 되어 있다.
$ cf create-service-broker mybrokername someuser somethingsecure https://mybroker.example.com --space-scoped
```

### 2.2. Service Broker의 Plan Access Control
- Service Broker 생성 시 특정 Space를 지정하지 않게 되면 Default로 모든 Plan에 대한 Access가 Disable 되어 있다.

#### 2.1.1. Service Broker의 Plan에 대한 Service Access 확인
```
$ cf service-access
getting service access as admin...
broker: elasticsearch-broker
   service        plan     access    orgs
   elasticsearch  standard limited

broker: p-mysql
   service   plan        access   orgs
   p-mysql   100mb-dev   all
``` 
- Service Access Type
	- all: Service Plan을 모든 사용자가 사용 할 수 있음
	- none: 모든 사용자가 Service Plan을 사용 할 수 없음
	- limited: 특정 Org에서 Service Plan을 사용 할 수 있음

#### 2.1.2. Service Access 활성화
- cf 명령어 cf enable-service-access의 -p 및 -o를 사용하여 특정 Plan, Org에 대한 Service Plan의 Access를 활성화 할 수 있다.
- Service Plan가 특정 Org또는 전체 Org에 Access가 할당 되면 사용자는 cf MarketPlace에서 해당 Service Broker를 조회/생성하여 사용 할 수 있다.
```
$ cf enable-service-access p-mysql -o dev-user-org 
Enabling access to all plans of service p-mysql for the org dev-user-org as admin...
OK

$ cf service-access
getting service access as admin...
broker: p-mysql
   service   plan        access   orgs
   p-mysql                        dev-user-org
```

#### 2.1.3. Service Access 비 활성화
- cf 명령어 cf disable-service-access를 사용하여 특정 Plan에 대한 Access를 제한 할 수 있다.
```
$ cf disable-service-access p-mysql
Disabling access to all plans of service p-mysql for all orgs as admin...
OK

$ cf service-access
getting service access as admin...
broker: p-mysql
   service    plan        access   orgs
   p-mysql    100mb-dev   none
```

### 2.3.  Service Broker 조회
```
$ cf service-brokers
Getting service brokers as admin...
OK

Name            URL
my-service-broker https://mybroker.example.com
```

### 2.4.  Service Broker 수정
```
$ cf update-service-broker mybrokername someuser somethingsecure https://mybroker.example.com
```

### 2.5. Service Broker 삭제
- Service Instance가 존재 할 경우 Service Instance를 삭제 후 진행해야 한다.
```
$ cf delete-service-broker mybrokername
```

### 2.6. Purge a Service
- Service Instance를 삭제 하지 않고 Service Broker를 지울 경우 cc_db에 Service Instance의 data 정보는 남게 되고 Broker 정보는 삭제 되어 사용 할 수 없는 상태로 남겨 지게 되는데 cf Purge 명령어를 통해 Service Broker API를 호출 하지 않고 cloud controller만 호출하여 db 정보만 지우게 한다.
```
# 
$ cf purge-service-offering service-test
# 
$ cf purge-service-instance mysql-dev
```
