
#  Ops Manager v2.5.11
- 진행한 Ops Manager 현재 Version은 아래와 같다.
	- Ops Manager v2.4
	- vSphere 5.5

## 1. Ops Manager Release Note 2.5
- [Release Notes](https://docs.pivotal.io/pivotalcf/2-5/pcf-release-notes/opsmanager-rn.html)

### 1.1. 주요 변경 사항
#### 1.1.1. Pending Changes Page
- Pending Changes Page의 "See Changes" 버튼을 클릭하여 Staged 한 Tile에 대한 변경 Manifest 정보를 확인 할 수 있다.

#### 1.1.2. Microsoft Azure Availability Zones Available
- MS Azure의 Availability Zone 활성화되어 사용이 가능하다.

#### 1.1.3. vSphere의 Host Group 추가
- Bosh Director Tile에서 AZ를 Cluster 하위인 Host Group에 연동하여 VM들을 조금 더 정확하게 관리할 수 있게끔 변경

#### 1.1.4. NATS Certificate Rotation Available in the Ops Manager API
- Ops Manger API를 사용하여 NATS CA를 다시 생성하고 Rotate 할 수 있다.

#### 1.1.5. Apply Custom VM Extensions to the BOSH Director in the Ops Manager API
- Ops Manager API "PUT /api/v0/staged/products/PRODUCT_GUID/JOBS/JOB_GUID/resource_config"를 사용하여 Bosh Director Tile과 Product Tile들의  VM Extension 정보를 변경 할 수 있다.

#### 1.1.6. Store BOSH Job Credentials on tmpfs
- Bosh Credential 정보를 Persistence Disk가 아닌 Memory에 저장하여 사용하게 하는 기능 추가, 해당 기능을 사용하다 Bosh Credential 정보를 상실하였을 경우 Recreate all VM을 통해 전체 VM을 재설치 해야한다.

#### 1.1.7. Forward Ops Manager Debug Logs to An External Store in the Syslog Form
- Bosh Director의 Debug Log를 외부 Syslog 서버로 Forwarding 하는 기능이 추가

#### 1.1.8. Custom Rsyslog Configuration Available in Ops Manager Syslog Template
- 사용자 지정의 Rsyslog Configuration를 지정하여 Rsyslog 서버에 반영 할 수 있음

#### 1.1.9. Pivotal Network의 Ops Manager Download Name 변경
- Pipeline에 대한 수정이 필요 할 수 있음

## 2. Ops Manager Update 절차

### 2.1. Ops Manager Download
- Pivotal Network에서 신규 Ops Manager v2.5.11를 Download 한다.

### 2.2. 기존 Ops Manager Export Installation
- Ops Manager Web 화면에서 [Setting -> Export Installation Setting] 화면에서 v2.4 Version의 Ops Manager 설정 정보를 Export 한다.

### 2.3. Ops Manager Install
- 기존 v2.4 Ops Manager 전원 Off
- vShpere 환경에 Ops Manager를 설치한다 (기존 Ops Manager와 같은 Network 정보).
[OpsManager 설치 가이드](https://docs.pivotal.io/pivotalcf/2-5/om/vsphere/deploy.html)

### 2.4. Ops Manager Import
- 설치 완료 한 Ops Manager 화면에서 Upgrade, Import Existing Installation 버튼을 클릭하고 v2.4의 Installation.zip 파일을 Import 시킨다.
- Decryption Passphrase를 반드시 올바르게 입력한다.

### 2.5. Bosh Director Apply Change
- Bosh Director Config를 확인하고 Bosh Director만 Apply Change를 클릭 한다.

## 3. Issues

### Ops Manager Install시 Error 발생
- Ops Manager 설치 시 Disk 배포 중 Cluster의 주소를 찾지 못하는 에러가 발생
	- 해결 방법: Ops Manager OVA 파일이 있는 PC의 Hosts 파일에 해당 Cluster의 host  주소를 설정

### Ops Manager Install시 IP가 붙지 않는 문제 발생
- vSphere Client 화면에서 Ops Manager VM이 생성되었지만 IP가 붙지 않는 문제가 발생
	- 해결 방법: NTP Server 주소를 넣지 않아 발생, NTP Server 주소를 넣고 vSphere VM 재기동 후 문제 해결

### Bosh Director Deploy 중 Error 발생
- Create Stemcell 중 vSphere Cluster의 주소를 찾지 못하는 에러가 발생
```
CPI 'create_stemcell' method responded with error: CmdError{"type":"Unknown","message":"getaddrinfo: Name or service not known (xxxxxx1:443)","ok_to_retry":false}
```
- 해결 방법: Ops Manager Hosts 파일에 해당 Cluster의 host 주소를 설정
