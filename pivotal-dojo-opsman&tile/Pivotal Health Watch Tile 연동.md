
#  Pivotal Health Watch Tile 연동

- Pivotal Health Watch Tile은 Pivotal Health Watch를 설치하는 Tile으로 Pivotal Health Watch 설치에 대한 전반적인 Domain, Network, Component, Errand 등의 Config 정보를 설정 할 수있다.
- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
	- Ops Manager가 구축되어 있어야 한다.
	- Bosh Director가 구축되어 있어야 한다.
	- PAS가 구축되어 있어야 한다.
	- Ops Manager에 Pivotal Health Watch Tile이 존재 해야 한다.
- Modify Last Version 1.4

## 1. Pivotal Health Watch 소개

- PCF Healthwatch는 Bosh Director와 Pivotal Cloud Foundry의 현재 상태, 성능 및 용량을 시각화하여 Monitoring하고 Event Alert와 연동하여 임계치 한계시 사용자에서 E Mail Alarm을 제공하는 Service

### 1.1. Pivotal Health Watch Architecture

- Pivotal Health Watch의 Ingestor Application이 Pivotal Cloud Foundry Loggregator Firehose의 Metric 정보를 수집 한다.
- Ingestor Application은 Metric 정보를 Redis에 전달하고 Woker Application이 Redis를 사용하여 Data를 집계하고 변환하여 Mysql에 저장 한다.
- Mysql에 변환된 데이터는 Pivotal Health UI에서 표기되거나 Aggregator App을 통해 추가로 변환하여 외부 Pivotal Cloud Foundry Firehose를 통하여 출력된다.

![healthwatch tile_Image][healthwatchtile-image-1]


## 2. Pivotal Health Watch Deploy

### 2.1. Pivotal Health Watch Tile 다운로드 & 업로드
- Pivotal Network에 접속하여 Pivotal Health Watch Tile를 다운로드 한다.
- 다운로드 한 Pivotal Health Watch Tile를 Ops Manager에 Import 한다.

### 2.1. Pivotal Health Watch Tile Config

#### 2.1.1. Assign AZs and Networks
 
- Pivotal Cloud Foundry를 배포 할 때 Singleton으로 구성 할 VM과 여러 다중으로 구성하는 AZ를 선택 한다.
- Bosh Director Tile에서 설정한 Network를 선택 한다.
- AZ 또는 Network를 증설 할 경우 Bosh Director Tile의 정보를 수정하고 Apply Change 해야한다.

#### 2.1.2. Healthwatch App Components Settings

- Foundation Name: Pivotal Healthwatch에서 생성 된 모든 Firehose Metric에 해당 Foundation Name이 Tag로 표시 됨
- Ingestor Count: Platform Metric을 수집하는 Pivotal Healthwatch의 Ingestor Application Instance 수 (PAS의 Doppler 수만큼 생성을 권장)
- Redis Worker Count: Redis의 데이터를 집계하여 가공하는 Worker  Application의 수(Ingestor Instnace 수 만큼 생성을 권장)
- BOSH Director Health Check Instance Count: Bosh Director의 상태를 검사하는 Application의 수
- BOSH Deployment Task Check Instance Count: Bosh 배포 작업에 대한 상태를 검사하는 Application의 수
- CLI Command Health Check Instance Count: Cloud Foundry CLI의 명령 상태를 검사하는 Application의 수 
- Ops Manager Check Instance Count: Ops Manager의 가동 시간을 확인하는 Application의 수
- App Canary Health Check Instance: Canary 앱 가동 시간 및 응답 확인하는 Application의 수
- Publish events to Event Alerts Tile: Event Alert Tile 기능을 사용 할 것 인지 선택

#### 2.1.3. Login info for health checks

- Ops Manager Validation Testing
	- Ops Manager URL:  Ops Manager URL을 입력 (Protocol을 입력 해야 하며 해당 URL은 Application에도 접근이 가능 해야 한다.)
- BOSH Health Check Availability Zone: BOSH가 VM을 배포 할 수있는 사용 가능한 AZ를 선택 한다.
- BOSH Health Check VM Type: BOSH가 VM을 배포 할 때 Spec을 선택 한다.
- BOSH Deployment Checker:  Pivotal Healthwatch가 BOSH Director에 연결 UAA 정보를 입력한다.

#### 2.1.4. Configure properties for Syslog Forwarding

- Do you want to configure Syslog Forwarding: Syslog 사용 유무/방식 선택
- External Syslog Host: Syslog Server 주소 입력
- External Syslog Port: Syslog Server Port 입력
- External Syslog Protocol: Syslog Server Protocol 입력

#### 2.1.5. Errands

-Post-Deploy Errands: Pivotal Health Watch 배포 후 사용할 Application을 $ cf push 하고 Smoke Test를 통해 간단한 스크립트를 실행 한다.
- Pre-Delete Errands: Pivotal Health Watch 삭제 전 Resource를 삭제 한다.

#### 2.1.5. Resource Config
- Pivotal Health Watch의 VM Spce을 설정 한다.

#### 2.1.6. Apply Change
- Apply Change를 실행하여 Pivotal Health Watch를 배포 한다.
- Pivotal Health Watch 배포가 완료되면 Apps Manager UI에서 Org=System Space=healthwatch의 healthwatch app의 urls를 통해 화면으로 접속 한다.

![healthwatch tile_Image][healthwatchtile-image-2]

## 3. Pivotal Healthwatch API Config

- Pivotal Healthwatch의 Metric 계산 및 임계치 정보는 UI에서 변경이 불가능하며  Pivotal Healthwatch API를 사용하여 Pivotal Healthwatch를 Platform 규모에 맞게 커스텀아이징 & 구성해야 한다.

### 3.1 Healthwatch API를 사용하여 Alert 구성을 검색 및 구성하는 방법

- Healthwatch API를 사용하기 위해 UAAC Token을 받아 온다.

```
# Key 값은 PAS Tile의 Credential에 존재 한다.
$ uaac token client get <my_healthwatch_admin_client> -s <my_healthwatch_admin_secret>
# 아래 결과 값 출력
Successfully fetched token via client credentials grant.
Target: https://uaa.sys.xxx.xxx.co.kr
Context: healthwatch_api_admin, from client healthwatch_api_admin
```


- Healthwatch API의 상태를 확인한다.
```
$ curl https://healthwatch-api.SYSTEM-DOMAIN/info
# 아래 결과 값 출력
HAPI is happy
```

- 모든 Pivotal Health Watch의 Alert를 확인 한다.
```
$ uaac curl https://healthwatch-api.SYSTEM-DOMAIN/v1/alert-configurations

# 결과 값
RESPONSE BODY:
[ {
  "query" : "origin == 'mysql' and name == '/mysql/available' and deployment == 'cf' and job == 'mysql,database'",
  "threshold" : {
    "critical" : 1.0,
    "type" : "EQUALITY"
  }
}, {
  "query" : "origin == 'mysql' and name == '/mysql/galera/wsrep_ready' and deployment == 'cf' and job == 'mysql,database'",
  "threshold" : {
    "critical" : 0.0,
    "warning" : 0.9998999834060669,
    "type" : "LOWER"
  }
}, 
....... 생략
```

- 부분 Pivotal Health Watch의 Alert를 확인 한다.

```
$ uaac curl "https://healthwatch-api.SYSTEM-DOMAIN/v1/alert-configurations?q=origin == 'some_origin' and name == 'Some.Metric.Name'" 

# 결과 값
RESPONSE BODY:
[ {
  "query" : "origin == 'healthwatch' and name == 'uaa.throughput.rate'",
  "threshold" : {
    "critical" : 15000.0,
    "warning" : 12000.0,
    "type" : "UPPER"
  }
} ]
```

- Pivotal Health Watch의 Alert를 Update 한다.
```
$ uaac curl -X POST "https://healthwatch-api.SYSTEM-DOMAIN/v1/alert-configurations"  \
      -H "Content-Type: application/json" \
      --data "{\"query\":\"origin == 'some_origin' and name == 'Some.Metric.Name'\",\"threshold\":{\"critical\":90,\"warning\":80,\"type\":\"UPPER\"}}"
# 결과 값
RESPONSE BODY:
{
  "query" : "origin == 'healthwatch' and name == 'uaa.throughput.rate'",
  "threshold" : {
    "critical" : 90.0,
    "type" : "UPPER",
    "warning" : 80.0
  }
}
```
- 변경한 Alert를 재 조회 한다.
```
$ uaac curl "https://healthwatch-api.SYSTEM-DOMAIN/v1/alert-configurations?q=origin == 'some_origin' and name == 'Some.Metric.Name'" 

# 결과 값
RESPONSE BODY:
[ {
  "query" : "origin == 'healthwatch' and name == 'uaa.throughput.rate'",
  "threshold" : {
    "critical" : 90.0, # 변경 확인
    "warning" : 80.0, # 변경 확인
    "type" : "UPPER"
  }
} ]
```

### 3.2 Healthwatch API를 사용하여 Free Chunk Sizes 구성 방법

- 전제조건: Pivotal Cloud Foundry UAA Component에 healthwatch.admin 권한이 존재하고 있는 Client의 Token을 가지고 있다.

- Healthwatch API의 상태를 확인한다.
```
$ curl https://healthwatch-api.SYSTEM-DOMAIN/info
# 아래 결과 값 출력
HAPI is happy
```

- 전체 Free Chunk 구성 확인
```
$ uaac curl https://healthwatch-api.SYSTEM-DOMAIN/v1/free-chunks

# 결과 값
[
  {
    "id": 1,
    "deployment": "healthwatch-default",
    "value": 4096,
    "type": "memory"
  }
]
```

- free chunk configuration 값 수정
```
$ uaac curl -X POST "https://healthwatch-api.SYSTEM-DOMAIN/v1/free-chunks" \
      -H "Content-Type: application/json" \
      --data "{\"value\":8192, \"deployment\":\"prod-isolation-segment\",\"type\":\"memory\"}"

# 결과 확인
200 OK
```
- free chunk configuration 변경 값 확인

```
$ uaac curl https://healthwatch-api.SYSTEM-DOMAIN/v1/free-chunks

# 결과 값
[
  {
    "id": 1,
    "deployment": "healthwatch-default",
    "value": 8192, # 변경 확인
    "type": "memory"
  }
]
```

[healthwatchtile-image-1]:./images/healthwatchtile-1.png
[healthwatchtile-image-2]:./images/healthwatchtile-2.png
