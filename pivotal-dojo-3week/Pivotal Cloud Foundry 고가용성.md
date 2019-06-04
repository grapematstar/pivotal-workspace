## 1. Pivotal Cloud Foundry 고가용성

### 1.1. HA in Pivotal Cloud Foundry
- Pivotal Cloud Foundry는 Avalibility Zone을 분산 설치하여 플랫폼의 Minor/Major의 Version Upgrade 에 대한 Rolling Update 또는 일부의 Avalibility Zone이 무너졌을 경우에 대비하여 Down Time 없이 Platform Service 가동을 보장 한다.

### 1.2.  HA Pivotal Cloud Foundry 구성

#### 1.2.1. Pivotal Cloud Foundry HA 구성을 하기 위해서는 아래와 같은 조건이 필요하다.

- External LB 구성: Production 환경에서 HA를 제공하기 위해서는 Pivotal Cloud Foundry를 배포한 각 각의 Go Router VM에 대한 Load Balancer가 필요하다.
- BlobStore 구성: Pivotal Cloud Foundry 배포 시 Application에 대한 Metadata가 들어가 있는 BlobStore의 instance 수 는 무조건 1개로만 배포된다.  BlobStore가 구성 된 AZ가 내려갈 시 플랫폼을 살릴수 없을 수 있는 현상이 나타나는 이유로 권고 사항은 외부 S3를 사용하거나 AZ가 내려갔을 경우 빠르게 BlobStore를 Restore 해야 한다.

#### 1.2.2. Scale Up
- Pivotal Cloud Foundry는 Memory, Disk, Cpu에 대한 수직적 확장을 지원 한다. 수직적으로 확장하기 위해서는 다음과 같은 조건이 필요 하다.
	-  Scale Up 대상에 Diego Cell이 있을 경우 해당 Diego Cell의 Application Instance 들이 다른 Diego Cell에 이동 함으로 다른 Diego Cell의 용량이 충분한지를 확인해야 한다.

#### 1.2.3. Scale Out
- Pivotal Cloud Foundry는 각 VM의 Instance 수를 수평적으로 확장을 지원 한다.
	- 3개 이상의 AZ로 구성 시 AZ 수는 홀 수여야 한다.
	- Scale Out시 Down Time을 최소화 하기 위해서 권고하는 Instance 수
		- [https://docs.pivotal.io/pivotalcf/2-4/opsguide/scaling-ert-components.html](https://docs.pivotal.io/pivotalcf/2-4/opsguide/scaling-ert-components.html)

#### 1.2.4. BOSH Resurrector
- BOSH Resurrector 기능을 바탕으로 Pivotal Cloud Foundry의 HA 기능을 향상 시킬 수 있다.
	- BOSH Resurrector Enable 상태에서 지속적으로 Hardware 장애 및 Network 중단을 모니터링하고 이상이 있을 경우 VM을 재시작/재생성 하는 기능을 제공 한다.
-  BOSH Resurrector Enable 방법은 아래와 같다.
	-  $ bosh -e ${alias-env} update-resurrection ${on/off}
	
### 1.3.  Pivotal Cloud Foundry 고가용성 Test
#### 1.3.1. 가용존 장애 상황 재현 1차
1.  물리서버(AZ1) 정지 => HA Proxy 내려가 고가용성 보장X
2.  컨테이너 분배 확인
3.  서비스정상여부 확인
4.  cf push 테스트
5.  OpsManager / BOSH 상태체크
6.  Healthwtch 점검

#### 1.3.2 .가용존 장애 상황 재현 2차
1.  물리서버(AZ2) 정지
2.  컨테이너 분배 확인 , cf login 및 cf push 간헐적 오류 발생 , cloud controller VM : failing -> 정상화
3.  서비스정상여부 확인 
    서비스는 정상 
    apps manager 비정상
4.  cf push 테스트
5.  OpsManager / BOSH 상태체크
6.  Healthwtch 점검

#### 1.3.2. 장애복구
1.  물리서버 기동 (10:50)
2.  bosh cck 수행 (10:51) ==> DiegoCell 정상 구동 확인 bosh cck 수행 (16개 VM)  
    (bosh cck -d cf-54c9c2f906b6aba996cd --resolution=reboot_vm)
3.  서비스정상여부 확인 -> 정상
4.  cf push 테스트 -> 정상
5.  OpsManager / BOSH 점검 (bosh tasks 확인 / bosh cck 점검)
6.  Healthwtch 점검

#### 1.3.3. Test결과
- 1차 Test 결과
- 10시 25분 전원 off  
- 10시 29분 부터 : 서비스는 정상 / 배포,appsmanager, healthwatch 안됨  
- 10시 28분 email 알림  
- 10시 33분 서비스는 정상 Application 배포는 안됨  
- 10시 36분 cf push 복귀 안됨
- 10시 43분 cf push 정상
- 10시 47분 az2 네트워크 정상 복구 
- 10시 50분 서버 정상 기동
- 10시 51분 bosh cck 복구 작업  
- 11시 17분 정상복구완료

2차 Test 결과
- 3시 41분 az2 shut down
- 3시 42분 appsmanager 접속 안됨
- 3시 43분 appsmanager 정상
	- 잠시 순단현상이 있고 나머지는 정상/application 배포는 안 됨
- 3시 52분 : healthwatch 일부 지표 집계가 잘 안됨
- 3시 53분 : 서버 재기동
- !!장애 상황에서 bosh vms/logs 명령은 자제 (bosh가 deployment에 Lock을 건다.)
- 4시 09분
- bosh cck -d cf-54c9c2f906b6aba996cd --resolution=reboot_vm -n
- 4시 35분 종료

