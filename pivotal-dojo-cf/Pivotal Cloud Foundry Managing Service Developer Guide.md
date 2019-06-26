#  Pivotal Cloud Foundry Managing Service Developer Guide

- Pivotal Cloud Foundry의 Service Instance를 사용자 요구에 맞게 제공/대기(Provisioning )하고 Application과 통합하기 위한 방법에 대해 설명 한다.
- Modify Last Version 2.5

## 1. Services and Service Instances

 -  Pivotal Cloud Foundry는 Service Market Place를 제공하며 사용자에게 On-Demand 방식의 Service를 제공한다.
 - Platform 관리자는 Market Place에 Service들의 Plan을 지정해 놓고, Platform 사용자들은 해당 Service의 Plan을 호출하여 Service Instance를 생성, Application에 사용 한다. 
 - Service는 Database, Eureka, NFS 등이 될 수있고 Plan은 해당 Service에 대한 Spec이나 공유/단일 Application용 등을 지정 한다.
 
### 1.1. Pivotal Cloud Foundry Market Place와 Service의 기본 예시

#### 1.1.1.  Cf Org에 할당 한 marketplace 목록 확인
```
$ cf marketplace
Getting services from marketplace in org my-org / space test as user@example.com...
OK

service            plans               description                       broker
p-mysql            100mb, 1gb          A DBaaS                           mysql-broker
p-riakcs           developer           An S3-compatible object store     object-store-broker
```

#### 1.1.2. Service Instance 생성
```
$ cf create-service rabbitmq small-plan my-rabbitmq

Creating service my-rabbitmq in org console / space development as user@example.com...
OK
```

#### 1.1.3. Service Instance 생성 시 Json Format의 Parameter 전송 방법
```
$ cf create-service my-db-service small-plan my-db -c '{"storage_gb":4}'

Creating service my-db in org console / space development as user@example.com...
OK

$ cf create-service my-db-service small-plan my-db -c /tmp/config.json

Creating service my-db in org console / space development as user@example.com...
OK
```

#### 1.1.4. Service Instance 생성 시 Bind 하는 Application의 VCAP_SERVICES의 Tag 지정 방법
```
$ cf create-service my-db-service small-plan my-db -t "prod, workers"

Creating service my-db in org console / space development as user@example.com...
OK
```

#### 1.1.5. 생성한 Service Instances 목록 확인
```
$ cf services
Getting services in org my-org / space test as user@example.com...
OK

name       service       plan        bound apps   last operation      broker
mybucket   p-riakcs      developer   myapp        create succeeded    object-store-broker
mydb       p-mysql       100mb                    create succeeded    mysql-broker
```

#### 1.1.6. 생성한 Service Instance의 상세 확인
```
$ cf service mydb

service instance:       mydb
service:                p-mysql
plan:                   100mb
description:            mysql databases on demand
documentation url:
dashboard:              https://p-mysql.example.com/manage/instances/abcd-ef12-3456
service broker:         mysql-broker

This service is not currently shared.

Showing status of last operation from service mydb...

status:    create succeeded
message:
started:   2019-02-13T12:02:19Z
updated:   2019-02-13T12:02:19Z

There are no bound apps for this service.
```

#### 1.1.7. Upgrade/Downgrade Service Plan 변경 방법
```
$ cf update-service mydb -p new-plan
Updating service instance mydb as user@example.com...
OK

$ cf update-service mydb -c /tmp/config.json

Updating service instance mydb as user@example.com...
```

#### 1.1.8. Service Instance 삭제 방법
```
$ cf delete-service mydb

Are you sure you want to delete the service mydb ? y
Deleting service mydb in org my-org / space test as user@example.com...
OK
```

## 2.  User-Provided Service Instances

- Pivotal Cloud Foundry는 사용자 지정 Service Instance를 사용하여 Market Place에 등록 되어 있지 않은 Service를 Application에 Bind 할 수 있다.
- User-Provided Service는 Application에 Service 사용 자격 증명을 전달 하거나 Syslog와 호환하여 Application에 Bind 할 수 있다.
- User-Provided Service가 Instance로 등록 되게 되면 일반 Service Instance 처럼 사용이 가능 하다.
- 사용 예시로 User-Provided Service를 통하여 Oracle DB를 사용 할 경우 Application에 Oracle DB의 Host, Username, Password 등의 사용 자격 증명을 Application에 전달하여 Application 내부에서 사용이 해당 자격 증명을 Source Code를 통해 사용 가능 하도록 한다.

### 2.1 Pivotal Cloud Foundry User-Provided Service Instances 사용 예시

#### 2.1.1. User-Provided Service Instance 생성 방법
```
$ cf cups SERVICE_INSTANCE -p '{"username":"admin","password":"pa55woRD"}'

$ cf cups my-db-mine -p "host, port"

host> rdb.local

port> 5432

Creating user provided service my-user-provided-route-service in org my-org / space my-space as user@example.com...
OK
```
#### 2.1.2. User-Provided Service Instance 확인 방법
```
$ cf services
Getting services in org system / space dev as admin...

name          service          plan       bound apps   last operation     broker
my-db-mine    user-provided               
```

## 3. Service Instance Credentials
- Service Instance를 Application에 Bind를 하면 Application 내부 Source Code 상에서 Service Instance를 구현 및 통신하는데 사용하는 자격 정보가 전달 된다.
- Service Instance의 자격 증명은 Application의 VCAP_SERVICES 명을 가지고 OS 시스템 환경 변수와 Cloud Controller cc_db에 저장 된다.
- External 및 Local Client의 경우 Service Key를 사용하여 자격 증명을 생성하여 Service Instance와 직접 통신 할 수 있다.

### 3.1. Application Bind 사용 예시

#### 3.1.1. Service Instance를 Application에 Bind 방법
```
$ cf bind-service my-app mydb
Binding service mydb to my-app in org my-org / space test as user@example.com...
OK
TIP: Use 'cf push' to ensure your env variable changes take effect

$ cf restart my-app
```

#### 3.1.2.  Route를 이용한 Service Instance bind 방법, 특정 Service Instance에서만 가능하다.
```
$ cf bind-route-service shared-domain.example.com --hostname my-app my-service-instance
Binding route my-app.shared-domain.example.com to service instance my-service-instance in org my-org / space test as user@example.com...
OK
```

#### 3.1.3.  Service Instance bind 시 Parameter를 전달 하는 방법, 특정 Service Instance에서만 가능하다.
```
$ cf bind-service rails-sample my-db -c '{"role":"read-only"}'

Binding service my-db to app rails-sample in org console / space development as user@example.com...
OK

$ cf bind-service rails-sample my-db -c /tmp/config.json

Binding service my-db to app rails-sample in org console / space development as user@example.com... OK
```

#### 3.1.4.  Service Instance bind 시 Bind의 Naming 생성 방법
```
$ cf bind-service my-app my-service --binding-name postgres-database
OK
```

#### 3.1.5.  Service Instance bind 시 Bind의 Naming 생성 방법
```
$ cf bind-service my-app my-service --binding-name postgres-database
OK

"VCAP_SERVICES": {
  "service-name": [
    {
      "name": "postgres-database",
      "binding_name": "postgres-database",
      ...
    }
  ]
}
```
#### 3.1.6.  Service Instance bind 해제 방법
```
$ cf unbind-service YOUR-APP YOUR-SERVICE-INSTANCE
```

### 3.2. Credentials Service Key 사용 예시
- Service Key를 사용하여 External Service에 대한 사용 자격 증명에 대해 수동으로 설정하여 Application에 등록 할 수 있다.

#### 3.2.1. Service Key 생성 방법
```
$ cf create-service-key MY-SERVICE MY-KEY
Creating service key MY-KEY for service instance MY-SERVICE as me@example.com...
OK

$ cf create-service-key MY-SERVICE MY-KEY -c '{"read-only":true}'
Creating service key MY-KEY for service instance MY-SERVICE as me@example.com...
OK

$ cf create-service-key MY-SERVICE MY-KEY -c PATH-TO-JSON-FILE
Creating service key MY-KEY for service instance MY-SERVICE as me@example.com...
OK
``` 

#### 3.2.2. 생성 한 Service Key 목록 조회 방법
```
$ cf service-keys MY-SERVICE
Getting service keys for service instance MY-SERVICE as me@example.com...

name
mykey1
mykey2
```
#### 3.2.3. 생성 한 Service Key 상세 조회 방법
```
$ cf service-key MY-SERVICE MY-KEY
Getting key MY-KEY for service instance MY-SERVICE as me@example.com...

{
  uri: foo://user2:pass2@example.com/mydb,
  servicename: mydb
}
```

#### 3.2.4. Service Key 삭제 방법
```
$ cf delete-service-key MY-SERVICE MY-KEY

Are you sure you want to delete the service key MY-KEY ? y
Deleting service key MY-KEY for service instance MY-SERVICE as me@example.com...

OK
```
## 4. Sharing Service Instances
- Space Developer 권한을 갖은 사용자를 통해 서로 다른 Space의 Application이 하나의 Service Instance를 공유하여 사용 할 수 있도록 한다.
- 플랫폼 관리자가 생성한 Service를 공유하게 되면 일반 권한을 갖고 있는 사용자는 해당 Service Instance에 대한 Bind/Unbind만 할 수 있고 Update/Delete는 불가능 하게 한다.

### 4.1 Sharing Service Instance 사용 예시

#### 4.1.1. Service Instance를 공유하기 위해 Flag를 활성화 한다.
```
$ cf enable-feature-flag service_instance_sharing
```

#### 4.1.2. Shard Service Instance 생성
```
$ cf share-service SERVICE-INSTANCE -s OTHER-SPACE [-o OTHER-ORG]
```
- 공유 할 Org에 같은 이름의 Service Instance를  갖고 있는 Shard Service Instance를 사용 할 수 없다.
- Service Instance의 Plan을 공유 하기 위해서 초기 1회 cf enable-service-access 명령어를 실행 해줘야 한다.

#### 4.1.3. Service Instance Shard 해제
```
$ cf unshare-service SERVICE-INSTANCE -s OTHER-SPACE [-o OTHER-ORG] [-f]
```
- Service Instance 공유를 해제 하면 Bind 되어 있던 모든 Application에서 Unbind 되어 에러를 발생 시킬 수 있다.


## 5. Outbound IP Addresses
-   Application이 Pivotal Cloud Foundry 외부 Service와 통신 할 수있게하려면 Outbound IP 주소를 기반으로 Application 연결을 허용하도록 Service를 구성해야 한다.
- External Service에서 Diego Cell의 IP 대역을 허용, Application의 접속 Domain을 등록해야 한다.
