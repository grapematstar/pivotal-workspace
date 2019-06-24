
#  Pivotal Cloud Foundry Operating Security

- Pivotal Cloud Foundry에서는 Platform, Application 단의 여러 보안 구성을 지원한다.

- 전제 조건
	- vShpere 환경을 전제하에 작성 하였다.
	- Ops Manager가 구축되어 있어야 한다.
	- Bosh Director가 구축되어 있어야 한다.
	- Pivotal Cloud Foundry가 구축 되어 있어야 한다.
- Modify Last Version 2.5

## 1. Pivotal Cloud Foundry Security

- Pivotal Cloud Foundry Security에서 제공하는 Security는 아래와 같다.
	- Application과 Container Data의 분리
	- 내부 Component의 연결 암호화
	- 사용자 역할 권한 부여를 통해 Space Access 관리
	- Resource Starvation 상황의 서비스 공격 방지
	- 적은 수의 Network의 노출

### 1.1. System Access 노출 제한

- Pivotal Cloud Foundry 기본적인 구성에서는 거의 모든 Component 들이 Private Network(Vlan)으로 배포 되며 DMZ 영역에 Go Router의 Endpoint에 도달하는 Load Balancer,  Platform 관리를 위한 Jump box, OutBound를 위한 NAT VM 3가지가 필요하다, External과의 연결을 적은 수로 하여 보안 취약성을 최소화 한다.

![cf-security][cfsecurity-image-1]

### 1.2. Isolation Segments

- Isolation Segments를 통하여 하나의 PAS에서 Application을 격리 된 공간에 배포하도록 구성해 네트워크의 복잡성과 중복 관리 요소를 줄 일 수 있다.
- PAS 내의 ORG/SPACE에서 하나의 Isolation Segments을 사용하도록 구성하여 격리된 환경의 다른 ORG/SPACE에 영향을 받지 않으며 Service를 구축 할 수 있다.
- Cloud Controller는 placement_tag을 통하여 Isolation Segments를 구분하여 해당 Cell에 접근 가능 하도록 한다.

![cf-security][cfsecurity-image-2]

### 1.3. Pivotal Cloud Foundry 인증 & 권한 UAA

- UAA는 Pivotal Cloud Foundry 및 다양한 Deployment에 대한 사용자 ID Manager Service
- UAA는 OAuth2 인증 서버 역할을하며 Platform또는 Application 대한 Resource/API End Point를 요청하는 Access Token을 발급 한다.

### 1.4. ORG/SPACE에 대한 Access 권한 관리

- Pivotal Cloud Foundry 사용자가 Application을 관리하기 위해서는 cf CLI를 통하여 API Endpoint에 Target을 설정하고 UAA 사용자 Login을 해야하며 사용자는  Pivotal Cloud Foundry의 Org/Space에 대한 Access 권한이 반드시 존재해야 한다.
- 주요 Access 권한에 대한 Permission scope는 아래와 같다.
	- cloud_controller.read
	- cloud_controller.write
	- cloud_controller.admin
	- cloud_controller.global_auditor

### 1.5. Service Broker에 대한 Access 권한 관리

- Pivotal Cloud Foundry 내부 Component Cloud Controller를 통해 Market Place의 Service Broker를 등록 할 경우 관련한 모든 API에 대해 인증 절차를 하게 되고 권한이 없는 사용자 또는 사용자를 입력하지 않을 경우 해당 요청을 Reject 한다.

### 1.6. Software 취약점 관리
- Pivotal Cloud Foundry는 Release와 Stemcell을 사용하여 Software 취약점 관리한다.
- Release를 통하여 Code 부분의 취약점을 지속적으로 보완하며 Stemcell을 통하여 OS 레벨의 보안 패치가 이루어진다.

### 1.7. Application Security

- Pivotal Cloud Foundry는 아래와 같은 기능으로 Application의 Code와 구성을 보호 한다.
	- Application을 Push 할 경우 UAA/SSL을 통한 CF API의 호출을 보안
	- RBAC(역할 기반 엑세스 제어)를 통해 권한이 있는 사용자만이 특정 Application에 접근
	- Cloud Controller는 Application의 구성을 암호화 한 Database Table에 저장
	- Cloud Controller의 Security Group을 통해 Application의 Inbound/Outbound를 설정
	- Secure Container 환경에서 Application 실행

### 1.8. Security Event Log의 추적

- Pivotal Cloud Foundry는 Bosh Director CLI와 Cloud Foundry CLI를 통하여 특정 VM/Application의 Event Log를 추적 할 수 있다.
```
# 최근 실행한 내역을 확인하고 bosh task {task ID}를 통해 상세 실행 내용을 확인 할 수 있다.
$ bosh tasks --recent
# 결과 값
ID     State       Started At                    Last Activity At              User                                Deployment         Description                          Result
24811  processing  Wed Jun 19 00:34:42 UTC 2019  Wed Jun 19 00:34:42 UTC 2019  p-healthwatch-84d14c8d77d6baa2c4c0  bosh-health-check  create deployment                    -
24809  done        Wed Jun 19 00:26:36 UTC 2019  Wed Jun 19 00:26:58 UTC 2019  p-healthwatch-84d14c8d77d6baa2c4c0  bosh-health-check  delete deployment bosh-health-check  /deployments/bosh-health-check
24808  done        Wed Jun 19 00:24:41 UTC 2019  Wed Jun 19 00:26:35 UTC 2019  p-healthwatch-84d14c8d77d6baa2c4c0  bosh-health-check  create deployment

$ cf evnet {my-app} 
Getting events for app my-app-1 in org system / space system as admin...

time                          event                      actor      description
2019-06-18T17:25:12.00+0900   audit.app.ssh-authorized   admin      index: 0
2019-06-12T11:47:07.00+0900   audit.app.process.crash    web        index: 0, reason: CRASHED, cell_id: 5244b492-3477-42bd-a6b0-d4d4cba5bfb9, instance: 15233296-66ec-4038-4f9a-0cb3, exit_description: Instance never healthy after 1m0s: Failed to make TCP connection to port 8080: connection refused
2019-05-24T09:16:47.00+0900   audit.app.droplet.create   admin
2019-05-24T09:16:28.00+0900   audit.app.update           admin      state: STARTED
2019-05-24T09:16:28.00+0900   audit.app.build.create     admin
```

## 2. Container Security

### 2.1. Pivotal Cloud Foundry Garden Container Isolation

- Pivotal Cloud Foundry의 Application은 Diego Cell Component의 Garden Container에서 실행된다. Garden Container는 Diego Cell의 가상/물리적인 공간을 사용하여 Application의 Process/Memory/File System을 격리하여 실행 한다.
- 동일한 Host에 있는 모든 Container들은 서로를 탐지하지 못하도록 설정되며 별도의 PID/namespace/network namespace/mount namepace/root file system을 가지고 있다.
- Pivotal Cloud Foundry는 내부에 Packaging 되어 있는 GrootFS라는 릴리즈를 통하여 Container의 File System을 구축한다.

### 2.2. Pivotal Cloud Foundry Garden Container Security
- Pivotal Cloud Foundry는 아래와 같은 절차를 통해 Container를 보호 한다.
	- 기본적인 권한이 없는 Container 환경에 Application 실행
	- 기능과 Access의 제한하여 Container 보안 강화
		- Host의 Root가 아닌 다른 User의 UID/GID를 Container UID/GID에 Mapping하여 Application에서 Host의 Root 권한을 부여하지 않음
		- /proc, /sys(Process, System Device) Mount Disk는 Read Only로 구성된다. 
		- 권한이 없는 사용자에 대해 Dmesg kernel Log Message Access를 차단 한다.
		- Container 내부에서 Dependency의 의존성을 없애기 위해 별도의 Script와 Binary를 실행 하지 않는다.
	- Application Security Group 기능을 통해 Application의 Inbound/Outbound 허용/차단 설정

## 3. Application Security Groups

- Pivotal Cloud Foundry는 Application Security Groups를 통해 Application Traffic에 대한 Protocol, Port 및 IP Range를 지정 할 수 있다.

### 3.1. Application Security Groups는 크게 2가지의 Type을 가지고 있다.

- Staging ASG: Application의 Staging 단계에 적용되는 보안 정책  (Network를 통하여 Resource를 가져 온다)
- Running ASG: Application의 Running 단계에 적용되는 보안 정책 (더이상 Resource가 필요 없음으로 조금더 강화 한 규칙을 선언 한다.)

### 3.2. Application Security Groups Scope

-  Application Security Group는 Platform 전체에 대한, Org에 대한, Space에 대하여 설정 할 수 있다. 그로인하여 3중으로 Application Security Group를 구성 할 수 있다.

### 3.3. Application Security Group Rules Type
- Protocol: TCP/UDP/ICMP Protocol을 지원 한다.
- Destination: 단일 IP 주소, 192.0.2.0-192.0.2.50과 같은 IP Range 또는 Traffic을 수신 할 수있는 CIDR Block
- Ports: 단일 Port, 여러 개의 쉼표로 구분 된 Port 또는 Traffic을 수신 할 수있는 Port Range
- Code: ICMP Protocol Code 
- Type: ICMP Protocol type
-  

### 3.4. CF CLI를 통해 기본적인 Application Security Group 관리 예시

```
# 신규 Applciation Security Group 생성 시 Json 형태의 security-group 파일을 생성 한다.
$ cat security-group.json

[
  {
    "protocol": "icmp",
    "destination": "0.0.0.0/0",
    "type": 0,
    "code": 0
  },
  {
    "protocol": "tcp",
    "destination": "10.0.11.0/24",
    "ports": "80,443",
    "log": true,
    "description": "Allow http and https traffic from ZoneA"
  }
]

# CF CLI를 이용하여 my-asg 명을 가지고 있는 Applciation Security Group를 생성 한다.
$ cf create-security-group my-asg ~/workspace/my-asg.json`
OK

# 생성한 Applciation Security Group 조회
$ cf security-group my-asg
Getting info for security group my-asgas admin
OK

Name    my-asg
Rules
        [
                {
                        "destination": "0.0.0.0/0",
                        "protocol": "icmp"
                        "type": 0,
					    "code": 0
                },
                {
					    "protocol": "tcp",
					    "destination": "10.0.11.0/24",
					    "ports": "80,443",
					    "log": true,
					    "description": "Allow http and https traffic from ZoneA"
                }
        ]

     Organization   Space
#0   system         system

```
[cfsecurity-image-1]:./images/cfsecurity-1.png
[cfsecurity-image-2]:./images/cfsecurity-2.png
