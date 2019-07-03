# Pivotal Cloud Foundry Opensource LogSearch 연동
- LogSearch는 ELK 기반으로 유용한 Filter 처리가 존재하는 Logstash.conf 내장 되어 있어 Pivotal Cloud Foundry의 Org/Space 별 Application의 상세 Log를 수집 한다.
-  Director, Pivotal Cloud Foundry Tile의 Syslog Setting을 통해 일반적인 VM Log도 수집 가능하다.
- 일반 Bosh ELK Deployment와는 다르게 별도의 Nozzle Plugin으로 Application 또는 Org와 연동이 필요 없이 Prometheus와 동일 하게 Firehose를 통해 자동 모든 Application을 설정한다.
- 지속적으로 Check하여 신규 Org/Space, VM에 대해서도 별도의 입력 필요 없이 자동으로 Log를 수집한다.
-  전제 조건
	- Bosh, Pivotal Cloud Foundry가 배포 되어 있어야 한다.
	- 외부와 통신이 되는 External Jumpbox가 존재 해야 한다.

## 1. LogSearch Deploy 준비

### 1.1. LogSearch Git을 아래 명령어를 통해 Clone 한다.

```
$ git clone https://github.com/cloudfoundry-community/logsearch-boshrelease.git
```

### 1.2. LogSearch 배포 시 사용하는 Manifest 소개
- logsearch-deployment.yml: 기본 ELK 기반의 LogSearch를 배포하기 위한 Main Manifest
- cloudfoundry.yml: Pivotal Cloud Foundry 내부 컴포넌트 UAA, NATS와 연동 하기 위해 사용하는 - Option Manifest
- logsearch-vars.yml: Pivotal Cloud Foundry UAA 인가를 사용하기 위한 Client가 존재하고 있는 Vars File

### 1.3. LogSearch  Release Download & Upload
#### 1.3.1. LogSearch  배포에 사용하는 Release를 로컬에 다운로드 & Bosh에 업로드 한다.  다운로드 할 Release는 아래와 같으며, LogSearch  Deployment 버전 별 차이가 있을 수 있다.

- logsearch/210.2.0
- bpm/1.0.4
- routing/0.188.0
- logsearch-for-cloudfoundry/210.2.0

```
# 로컬 다운로드
$ wget https://bosh.io/d/github.com/cloudfoundry-community/logsearch-boshrelease?v=210.2.0
$ wget https://bosh.io/d/github.com/cloudfoundry-incubator/bpm-release?v=1.0.4
$ wget https://bosh.io/d/github.com/cloudfoundry-incubator/cf-routing-release?v=0.188.0
$ wget https://s3.amazonaws.com/logsearch-for-cloudfoundry/logsearch-for-cloudfoundry-210.2.0.tgz

# bosh에 업로드
$ bosh -e {ailas-env} upload-release logsearch-boshrelease?v=210.2.0
$ bosh -e {ailas-env} upload-release bpm-release?v=1.0.4
$ bosh -e {ailas-env} upload-release cf-routing-release?v=0.188.0
$ bosh -e {ailas-env} upload-release logsearch-for-cloudfoundry-210.2.0.tgz
```

### 1.4. LogSearch  Manifest Modify
#### 1.4.1. Bosh의 Cloud Config에 맞게 Main Manifest 파일을 수정 한다. 주요 수정 내용은 아래와 같다.
- networks
- vms
- disks
- azs

#### 1.4.2. Kibana 배치 중 Timeout Error가 발생 할 수 있음으로 Main Manifest의 설정을 수정한다.
```
kibana:
  health:
    timeout: 1000 # 500 -> 1000으로 수정
  env:
    - NODE_ENV: production
  config_options:
    xpack.monitoring.enabled: false
    xpack.graph.enabled: false
    xpack.ml.enabled: false
    xpack.security.enabled: false
    xpack.watcher.enabled: false
```

#### 1.4.2.  Option Manifest의 설정에서 Pivotal Cloud Foundry의 Deployment Name을 수정한다.
```
cloud_controller: {from: cloud_controller, deployment: cf} # 해당 부분을 아래 처럼 전체 수정
cloud_controller: {from: cloud_controller, deployment: cf-7dd8b2324d3bed84c77c}
```

## 2. LogSearch Deploy

### 2.1. bosh 명령어를 통해 LogSearch를 배포 한다.

```
$ bosh -e {ailas-env} -d logsearch deploy logsearch-deployment.yml \
  --vars-store env-repo/logsearch-vars.yml \
  -v cf_admin_password="XrUGwt0krgsWqku1Lk5O_-ND_FrB0HLK" \
  -v uaa_admin_client_secret="hXhXHK2JwbEnJlp7pM8bq_9TNNEXeiNt" \
  -v system_domain="sys.{doamin}" \
  -o operations/cloudfoundry.yml 
```

### 2.2. 도메인 정의
#### 2.2.1. LogSearch 배포 시 설정한 system_domain을 통해 자동으로 logs.((system_domain))으로 Domain이 생성 되는데 해당 부분에 대해 Bind9이나 Hosts를 등록하여 접근 가능하게 한다.

## 3. LogSearch & Pivotal Cloud Foundry 연동

### 3.1. Uaa Client 생성

#### 3.1.1. Pivotal Cloud Foundry의 Loggregator Firehose와 Uaa Login을 하기 위해 Uaa Clinet를 추가 한다.
```
$ uaac target https://uaa.{system_domain}

$ uaac token client get admin -s XrUGwt0krgsWqku1Lk5O_-ND_FrB0HLK

$ uaac client add firehose-to-syslog \
--name firehose-to-syslog \
--secret l9pyoirbhm2xg553cj5m \
--authorized_grant_types client_credentials,refresh_token \
--authorities doppler.firehose,uaa.none,cloud_controller.admin

$ uaac client add kibana \
--name kibana \
--secret eexj5xawrml9laroh7z2 \
--authorized_grant_types client_credentials,refresh_token \
--authorities uaa.none \
--scope openid,oauth.approvals,scim.userids,cloud_controller.read 
 

$ uaac client add kibana_oauth2_client -s eexj5xawrml9laroh7z2 --redirect_uri "https://logs.{system_domain}/login " \
  --scope "scim.userids, oauth.approvals, cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
  --authorized_grant_types "authorization_code , client_credentials , refresh_token" \
  --authorities="uaa.resource" \
  --autoapprove="openid , cloud_controller_service_permissions.read"
```

### 3.2. Kibana 접속

#### 3.2.1. logs.{system_domain}를 통하여 Web UI 접속을 한다.
- 접속 시 Id/Password는 Pivotal Cloud Foundry의 UAA Login 정보 (Apps Manager와 같다.)

#### 3.2.1. Index 확인

- Kibana UI의 오른쪽 메뉴 Management -> Kibana Index Patterns -> Create Index Pattern에 들어가서 아래와 같은 형식의 Patten을 확인한다.
	- logs-app-{ORG}-{SPACE}-2019.07.02: 해당 ORG/SPACE의 Application Log
	- logs-platform-2019.07.02: Bosh, Pivotal Cloud Foundry의 Syslog (설정을 하지 않으면 보이지 않을 수 있다.)

#### 3.2.2. Index 생성
- Index를 생성하여 Log를 확인한다.
- Log Example
	- @message:Connect from 172.28.86.52:52358 to 172.28.86.20:80 (health_check_http_url/HTTP) @timestamp:July 2nd 2019, 17:50:20.000 @raw:<46>1 2019-07-02T17:50:20+09:00 172.28.86.20 haproxy 156 - [instance@47450 director="" deployment="cf-7dd8b2324d3bed84c77c" group="tcp_router" az="az1" id="08c7b38a-9183-46a3-a7d6-2787c9998fcc"] Connect from 172.28.86.52:52358 to 172.28.86.20:80 (health_check_http_url/HTTP) tags:syslog_standard, platform, cf, haproxy, fail/cloudfoundry/platform-haproxy/grok @source.host:172.28.86.20 @source.az:az1 @source.deployment:cf-7dd8b2324d3bed84c77c @source.job:tcp_router @source.type:cf @source.component:haproxy @source.id:08c7b38a-9183-46a3-a7d6-2787c9998fcc @input:syslog @shipper.priority:46 @shipper.name:haproxy_syslog syslog_procid:156 syslog_msgid:- syslog5424_ver:1 @level:INFO syslog_sd_params.group:tcp_router syslog_sd_params.deployment:cf-7dd8b2324d3bed84c77c syslog_sd_params.az:az1 syslog_sd_params.id:08c7b38a-9183-46a3-a7d6-2787c9998fcc @type:haproxy syslog_sd_id:instance@47450 @index_type:platform _id:jIDgsWsBvSB4y3Sdrdwt _type:doc _index:logs-platform-2019.07.02 _score: -tor .... 생략
