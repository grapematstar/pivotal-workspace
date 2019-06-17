
# Pivotal Event Alerts 연동

## 1. Pivotal Event Alerts Health Watch 연동 방법

- 전제 조건: Bosh, Pcf, Health Watch가 설치, smtp 서버 존재해야 한다, Pivoctal Mysql Tile 설치
- 기대 효과: 플랫폼의 KPI를 수집하고, 이벤트 룰을 설정해서, 이상 징후시 알림을 받아 문제 해결

### 1.1.  Pivotal Event Alerts 소개
- Pivotal Cloud Foundry의 Metric를 분석하여 VM에 대한 CPU, Disk, Memory, Router Health, Containers 수 등에 대하여 설정한 임계치가 넘어가면 경고 Alert를 주는 서비스
- Email, Slack, Webhook, Pivotal Healthwatch과 연동 가능

### 1.2.  Pivotal Event Alerts Tile Download & Ops Manager Upload
1. [https://network.pivotal.io](https://network.pivotal.io/) 에서 Pivotal Evnet Alert Tile을 다운로드 한다.
2. Download 완료 한 Pivotal Evnet Alert Tile을 Ops Manager에 [Import a Product] 한다.

### 1.2.  Pivotal Event Alerts Tile Configration

#### 1.2.1 . Ops Manager Main 화면에서 Tile의 정보를 수정 한다.
- Configure AZs and Networks
	-  Pivotal Event Alert를 설치 할 Cloud Config AZ와 Network 정보를 입력 한다.
- Configure Scaling Settings
	- Errand Event Alert Application에 대한 instance 수 정보 입력 (Alert 수가 많은 환경은 instance 수를 높인다)
- Configure MySQL Settings 
	- Pivotal Mysql Service Tile의 Plan 명을 입력 한다.
- Configure  SMTP Settings 
	- SMTP 서버 정보를 입력 한다. 

### 1.3.  Install the PCF Event Alerts Plugin

- Pivotal Event Alert를 Pivotal Health Watch를 연동하여 사용하기 위해서는 PCF Event Alerts Plugin를 설치해야 한다.

```
# cf plugin 설치
# pivotal network에서 해당 plugin을 다운로드 한다.
$ cf install-plugin linux64-1.1.1

# target 설정
$ cf eva-create-target 43235-email email xxx@xxx.com  ==>  생성후 메일 확인 필요
$ cf eva-targets

# 토픽 확인 및 설정 (publisher :  healthwatch 
$ cf eva-topics
$ cf eva-subscribe 43235-email healthwatch --topics  healthwatch.diego.availablefreechunks,healthwatch.diego.availablefreechunksdisk,rep.unhealthycell
$ cf eva-subscribe 43235-email healthwatch --all (모든 tipic 설정)
$ cf eva-unsubscribe 43235-email healthwatch (설정제거)

# 샘플테스트
$ cf eva-sample-publish healthwatch rep.unhealthycell

## healthwatch  임계치 조정 작업
가이드: https://docs.pivotal.io/pcf-healthwatch/1-4/api/alerts.html
참고 : https://github.com/myminseok/pivotal-docs/blob/master/platform-automation/install-healthwatch.md

# healthwatch-api를 사용하기 위해서는 healthwatch-api에 대한 권한이 있는 uaac token을 할당 해야 한다.
# uaac 토큰  받아오기
$ uaac token client get healthwatch_api_admin -s TIhDUHyEiv59F_zcrnC4MrfCItwXXNz_

$ uaac curl -k https://healthwatch-api.sys.xxx.xxx.co.kr/info

# healthwatch 의 topic 임계치 확인
$ uaac curl -k "https://healthwatch-api.sys.xxx.xxx.co.kr/v1/alert-configurations?q=origin == 'healthwatch' and name == 'Diego.AvailableFreeChunksDisk'"

# healthwatch 의 topic 임계치 설정
$ uaac curl -X POST "https://healthwatch-api.sys.xxx.xxx.co.kr/v1/alert-configurations"  \
       -H "Content-Type: application/json" \
       --data "{\"query\":\"origin == 'healthwatch' and name == 'Diego.AvailableFreeChunksDisk'\",\"threshold\":{\"critical\":5,\"warning\":10,\"type\":\"LOWER\"}}"
```
