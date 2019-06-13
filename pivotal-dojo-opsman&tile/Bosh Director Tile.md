##  Bosh Director Tile

- Bosh Director Tile은 Bosh 를 설치하는 Tile으로 Bosh 설치에 대한 전반적인 Network, AZ, Syslog, Instance Type 등의 Config 정보를 설정 할 수있다.
- Bosh Director Tile은 Ops Manager를 설치 시 자동으로 Tile을 구성 한다.
- 전제 조건
	- vShpere 환경이 구축되어 있어야 한다.
	- Ops Manager가 구축되어 있어야 한다.
- Modify Last Version 2.5
 
### 1. Bosh Director Tile Config 구성

#### 1.1. vCenter Config 화면

- Bosh Director를 설치 할 vShpere 환경 정보를 설정 할 수 있다.
- vShpere 환경 설정 정보는 Bosh Director를 통해 배포 하는 모든 Deployment에 영향을 준다.

[vCenter][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Name: vCenter Config의 별칭
- vCenter Host: vSphere 접속 Host Name/IP
- vCenter Username: vSphere 접속 User 명
- vCenter Password:  vSphere 접속 User의 PWD
- Datacenter Name: vCenter의 Datacenter 명
- Virtual Disk Type: vShpere의 가상 Disk 유형 선택
- Ephemeral Datastore Names: 배포한 VM의 Ephemeral Disk를 저장하는 DataStore 명, ",로 복수 개 구분"
- Persistent Datastore Names:  배포한 VM의 Persistent Disk를 저장하는 DataStore 명, ",로 복수 개 구분"
- vSphere의 Network 구성 설정
	-  Standard vCenter Networking: vShpere의 기본 Network 구성
	- NSX Networking: vShpere의 Network Virtualization 구성
		- NSX Mode: NSX-V or NSX-T를 선택
		- NSX Address: NSX manager 주소
		- NSX User Name: NSX Admin에 연결할 User 명
		- NSX Password: NSX Admin에 연결할 User의 PWD
		- NSX CA Cert: NSX 서버에 인증하는 Cert Key (OpenSSL을 통하여 검색 가능)
- VM Folder: 배포 한 VM이 배치되는 Folder(자동 생성), Director 배포가 완료 되면 변경 불가
- Template Folder: 배포 한 Stemcell이 배치 되는 Folder(자동 생성) , Director 배포가 완료 되면 변경 불가
- Disk path Folder: 배포 한 Disk가 배치되는 Folder(자동 생성), Director 배포가 완료 되면 변경 불가

#### 1.2. Create Director Config
- Bosh Director VM Job의 Config를  설정 할 수 있다.

[Director Config][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- NTP Servers : NTP server의 주소 ",로 복수 개 구분"
- JMX Provider IP Address:  JMX Provider IP 주소 (Firehose를 통해 Metric이 전달 됨으로 빈 값을 권고)
- Bosh HM Forwarder IP Address: Bosh Health Moniter의 Metric Forwarder IP 주소  (Firehose를 통해 Metric이 전달 됨으로 빈 값을 권고)
- Enable VM Resurrector Plugin: Bosh Resurrector Plugin을 사용하여 Runtime 환경에서의 자동 복구 기능을 사용 할 것인지 확인
- Enable Post Deploy Scripts: Deployment를 배포 후 Release의 Post Script를 실행 할 것인가 선택 (PKS를 설치하기 위해서 반드시 True)
- Recreate All VMs: Apply Change 시 선택한 Product의 VM을 Recreate 할 것인지 선택 (Persistent는 삭제 및 재생성 되지 않는다, Apply Change 완료 후 자동으로 체크 해제 한다.)
- Recreate All Persistent Disks:  Apply Change 시 선택한 Product의 VM을 Recreate 할 것인지 선택, 기존의 Data를 삭제하지 않고 migrate 한 뒤 Recreate 한다.
- Enable bosh deploy retries: 해당 기능을 선택하면 Apply Change 중 Bosh가 실패한 명령에 대해 5번  재시도 한다.
- Skip Director Drain Lifecycle: Bosh Director가 재생성 될 때 Drain Script가 재실행 되지 않도록 한다. Drain Script는 Job이 start/stop 될 때 실행 되는 Script
- Select Store BOSH Job Credentials on tmpfs: Bosh Director 설치 시 생성 되는 Credential 파일을 임시 파일 저장소(tmpfs) Memory에 저장
- Allow Legacy Agents:  False를 할 경우  모든 Product의 Stemcell Version이 3468 이상 일 경우 기존의 Agent 사용이 중지되고 Ops Manager가 TLS 통신을 구성
- Keep Unreachable Director VMs: Deploy 실패 후 Bosh Director에 대한 문제를 확인하기 위해 접근을 금지 한다.

---

[Pager Duty][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- HM Pager Duty Plugin: Bosh Director의 HM과 PagerDuty를 연동
	- Service Key: PagerDuty의 API Service Key
	- HTTP Proxy: PagerDuty와 함께 사용할 Http Proxy를 입력

---
---

[Pager Duty][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- HM Email Plugin: Bosh Health Monitor의 Status를 Email로 연동 
	- Host: SMTP 서버 주소
	- Port: SMTP 서버 Port 주소
	- Domain: Domain 입력
	- From: 발신자 입력
	- Recipients:  수신자 입력 ",로 복수 개 구분"
	- Username: SMTP 서버 사용자
	- Password: SMTP 서버 사용자의 PWD
	-  Enable TLS: TLS 사용 여부 선택
---

---

[CredHub Encryption Provider][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- CredHub Encryption Provider: Bosh Credhub가 내부적인 Bosh의 Director와 내부 VM으로 저장 할지, HSM 하드웨어로 저장 할지 선택
	- internal: 내부 Credhub를 사용 할 경우 선택
	- Luna HSM: SafeNet Luna HSM를 사용 할 경우 선택
		- Encryption Key Name: HSM이 CredHub 데이터를 암호화하고 해독하는 데 사용하는 Key Name, Director를 배포 한 이후 변경하면 서비스가 중단 될 수 있다.
		- Provider Partition:  Encryption Key를 저장하기 위한 파티션 (참조 [https://docs.pivotal.io/pivotalcf/2-5/customizing/hsm-config.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/hsm-config.html)) 
		- Provider Partition Password: Provider Partition Password (참조 [https://docs.pivotal.io/pivotalcf/2-5/customizing/hsm-config.html](https://docs.pivotal.io/pivotalcf/2-5/customizing/hsm-config.html))
		- Provider Client Certificate: Credhub가 HSM Client와 연결 할 때 사용하는 certificate 
		- Provider Client Certificate Private Key: Credhub가 HSM Client와 연결 할 때 사용하는 Certificate 
		- HSM Host Address: HSM Server 주소
		- HSM Port Address: HSM Server Port (Default HSM Server Port는 1792)
		- Partition Serial Number: Partition  번호
		- HSM Certificate: Credhub와 HSM 사이의 양방향 통신을 위한 mTLS Key
---

---

[Blobstore Location][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Blobstore Location: BlobStore를 Internal, External Endpoint로 구성 할 수 있다. Bosh Director를 배포 한 후에는 수정이 불가능 하다. 입력 값은 아래와 같다.
	- Internal: 내부 Blob 저장소를 사용 할 경우 선택
	- Enable TLS: Blob 저장소에 TSL를 사용 할 것 인지 선택
	- S3 Compatible Blobstore: Blob 저장소를 외부 S3 Storage로 사용 할 경우 선택 
		- S3 Endpoint: S3 Storage의 Endpoint 입력 
		- Bucket Name: S3 Storage의 Bucket 명
		- Access Key: S3의 Access Key
		- Secret Key:  S3의 Secret Key
		- V2/V4 Signature Select: V4일 경우 Region을 입력
	- GCS Blobstore:  Blob 저장소를 GCS로 사용 할 경우 선택
		- Bucket Name: GCS Buket 명
		- Storage Class: GCS Buket Storage의 Class 선택
		- Service Account Key: GCS API, Private가 있는 Json 파일
---

---

[Database][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Database Location: Bosh Database Component 구성을 Internal/External Mysql 선택을 할 수 있다.  Bosh Director를 배포 한 후에는 수정이 불가능 하다. 입력 값은 아래와 같다.
- Database TLS 허용을 선택하고 추가적 입력이 가능하다.
	- Internal: Bosh Interanl Database 사용
	- External MySQL Database:  Bosh Exteranl Mysql 사용
		- Host: External Database 주소
		- Port: External Database Port
		- Username: External Database 사용자
		- Password: External Database 사용자 PWD
		- Database: Database 명 ex) bosh
		- Advanced DB Connection Options: JSON 형식의 Database 대한 추가 옵션
		- TLS CA: 인증서의 CA 입력
		- TLS Certificate: Database와 TLS 상호작용을 하기 위한 Client Certificate 입력
		- TLS Private Key:  Database와 TLS 상호작용을 하기 위한 Client Private Key 입력
---

[Database][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Director Workers: Bosh Director가 Task를 실행 할 수 있는 Worker의 수 설정, Default는 5
- Max Threads: Bosh Director가 동시에 실행 할 수 있는 최대 Thread 수, 권고 사항은 Default 값으로 놓고 사용 Defualt 값은 32
- Director Hostname: Bosh Director에 사용자 설정 Domain을 연동
- Custom SSH Banner: Bosh Director를 SSH로 접속하게 될 때 보여지는 Banner

#### 1.3. Create Availability Zones Config

[Availability Zones][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Bosh Director는 vSphere의 Cluster를 통하여 AZ를 구분 할 수 있으며 여러개의 AZ로 분산해서 사용하여 VM과 Application에 대한 고 가용성과 로드 밸런싱을 제공 한다. Ops Manger Bosh Director를 통해 배포하는 Deployment의 Instance가 2개 이상 일 경우 자동으로 분산 설치 되며 Pivotal 권고 AZ Instance 수는 3개 이상이다.
- Availability Zones Page에서 "Add Availability Zone" 버튼을 클릭 한다.
	- Name: AZ의 명칭 입력
	- IaaS Configuration:  vCenter Config의 명칭 입력
	- Clusters:
		- Cluster: vCenter의 Cluster 명 입력
		- Resource Pool: vCenter의 Resource Pool 명 입력

#### 1.4. Create Networks Config
- Ops Manager Director Tile에서 설치 할 Bosh Net/Pas Net/Service Net 등 여러개의 Network을 할당하여 사용 할 수 있다. 해당 Network는 이 후 Bosh Director로 설치 하는 모든 Deployment에 영향을 주며 deploy 시 해당 정보를 바탕으로 bosh cloud config를 작성 한다.
- Enable ICMP checks: Ops Manager에서 해당 Network로 ICMP Ping 검사를 한다.
- Networks Config 설정 화면에서 "Add Network" 버튼을 클릭 한다.
	- Name: Network Config의 명칭
	- Subnets:
		- vSphere Network Name: vSphere Network의 명칭
		- CIDR: vSphere Network의 CIDR
		- Reserved IP Ranges: vSphere Network의 CIDR 중 Deploy 할당 제외 대역
		- DNS: vSphere Network의 DNS Server
		- Gateway: vSphere Network의 Gateway 주소
		- Availability Zones:  생성할 Network에 대한 Availability 선택

#### 1.5. Assign AZs and Networks Config

[Assign AZs and Networks][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Bosh를 설치 할 AZ와 Network를 설정 한다.
- Bosh를 설치 할 때 반드시 단일 AZ로 구성한 Network를 설정해야 한다.

#### 1.6. Security Config

[Security][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Trusted Certificates: Bosh를 통한 모든 Deployment가 해당 사용자 설정 Root CA Certificate 신뢰하게 한다.
- Include OpsManager Root CA in Trusted Certs: Ops Manager의 Root CA를 Bosh를 통해 배포한 모든 Deployment들이 신뢰하게 한다.
- Generate VM passwords or use single password for all VMs: Bosh를 통한 배포한 모든 Deployment에 대한 VM Password를 생성하거나, Bosh Password와 같게 할 수 있다. Password를 Generate하여 사용을 권고

#### 1.7. BOSH DNS Config

[BOSH DNS Config][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Excluded Recursors: Bosh의 내부 DNS에서 사용하지 않을 주소
- Recursor Timeout: Bosh 내부 DNS Recursion에서   응답 시간 제한
- Handlers: 특정 Domain을 다른 DNS에 Handler 할 Domain 주소 입력

#### 1.8. BOSH SysLog Config

[BOSH SysLog Config][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Do you want to configure Syslog for Bosh Director?: Syslog를 사용 할 경우 "Yes"를 선택 한다.
- Address: Syslog Remote Server의 주소
- Port: Syslog Remote Server의 Port
- Transport Protocol: 전송 Protocol 선택(UDP/TCP)
- Enable TLS: TLS 사용 여부 선택
	- Permitted Peer:  Syslog Remote Server의 Peer 입력
	- SSL Certificate: Syslog Remote Server의 SSL 인증서 입력
- Queue Size:  Queue 대기열 버퍼(메세지 수)의 Size, Default 100,000
- Forward Debug Logs: Debug Log를 전달(Log의 수가 많이 짐)
- Custom rsyslog Configuration: Rainerscript 형식의 Rsyslog의 구성 세부 사항 입력

#### 1.9. Resource Config

[Resource Config][https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/config.html#prerequisites)

- Bosh Director Config에서 Bosh Instance 수와 VM Spce을 지정 할 수 있다.


