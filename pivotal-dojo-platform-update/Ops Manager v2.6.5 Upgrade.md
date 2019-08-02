#  Upgrade Note 5 - Ops Manager v2.6.5
- 진행한 Ops Manager 현재 Version은 아래와 같다.
	- Ops Manager v2.5.11
	- vSphere 5.5

## 1. Ops Manager Release Note 2.6
- [Release Notes](https://docs.pivotal.io/pivotalcf/2-6/pcf-release-notes/opsmanager-rn.html)

### 1.1. Ops Manager 주요 변경 사항
#### 1.1.1. Ops Manager Supports Multiple Stemcells for Products
- Ops Manager는 다수의 Stemcell을 사용하는 Product Tile을 지원 한다.
	- Stemcells_associations 기능이 있는 Product에 대해 API를 제공하여  Multiple Stemcell 정보를 관리한다.
- Ops Manager의 "Review Pending Changes"를 통해 새로운 Stemcell의 변동 사항을 확인 할 수 있다.

#### 1.1.2. vSphere Operators Can Use Whitespace in Folder Names
- Ops Manger Director vSphere 정보 입력 시 Folder에 대한 공백을 인식한다. (전체 정보에 대한 Test를 진행 해야 한다.)

#### 1.1.3. Tile Developers Can Implement a Form Verifier to Confirm vSphere Properties
- vShpere 환경의 Product Tile 구성 시 Configuration 잘 못된 경우 운영자에게 알림을 기능 추가

#### 1.1.4. Added Strategies for Backing Up BOSH Blobstore to S3
- Bosh Director Tile Config에 Blobstore를 외부 External에 Backup하는 전략 추가 (버전 관리)
	- S3 with versioning
	- S3 without versioning

#### 1.1.5. API Endpoint Returns Deployment Configuration Details
- /api/v0/staged/products/:product_guid/pre_deploy_check API를 사용하여 배포 전 구성 정보를 확인 하여 누락된 Properties를 확인하여 Apply Change 시 발생 할 수 있는 에러를 방지 할 수있다.
- Pre Deploy 검증에 대한 목록은 아래와 같다.
	- Network 할당
	- AZ 할당
	- Stemcell의 지정 여부
	- 오류가 있는 속성
	- 잘 못된 Resource 구성

#### 1.1.6. Fresh Installations of Ops Manager Create New Default Certificate Authority in CredHub
- Ops Manager v2.6을 설치할 때 BOSH Director CredHub에 새로운 기본 인증 기관을 생성 한다.

#### 1.1.7. Change Log Page 변경
-   Ops Manager Change Log Page Product의 Deploy Status, Deploy 시간 상태 등이 표기되며 UI 변동.

#### 1.1.8. Ops Manager Adds Support for All AWS Regions
- AWS 환경의 Ops Manager에서 전체 Region을 사용 할 수있음.

#### 1.1.9. AWS Deployments Use 5th Generation Instances
- Ops Manager v2.6 부터 AWS 환경의 Instance Type이 t2 -> t3, c4->c5 5세대로 변경 됨.
- 사용자 정의 instance type catalog를 사용 할 경우 Instance Type이 변동 되지 않음.

#### 1.1.10. Compiled Release Assets Included in BOSH Deployment Manifest
- Ops Manager를 통해 배포하는 Product에 대해 Default exported_from 설정이 되어 있어 VM을 배포 시 Compile 되어 있는 Release를 사용하여 배포 시간을 절약 할 수 있다.

#### 1.1.11. Passwords Not Supported for Ops Manager VM on vSphere
- Ops Manager VM에 접속하기 위해 필요하던 Password가 SSH Key Pair로 변동

## 2. Ops Manager Update 절차
### 2.1. Ops Manager Install 사전 준비
#### 2.1.1. Key Pair 생성
- Ubuntu Jumpbox에서 Key Pair 생성 (ssh-keygen 사용)

```
$ ssh-keygen -t rsa -f .ssh/{key-pair-name}

$ cd .ssh
$ ls -al
total 40
drwx------  2 ubuntu ubuntu 4096 Jul 31 10:39 .
drwxr-xr-x 40 ubuntu ubuntu 4096 Aug  2 11:34 ..
-rw-------  1 ubuntu ubuntu 1176 Jun 19 11:42 authorized_keys
-rw-------  1 ubuntu ubuntu 1679 Apr 16 13:01 id_rsa
-rw-r--r--  1 ubuntu ubuntu  398 Apr 16 13:01 id_rsa.pub
-rw-------  1 ubuntu ubuntu 4218 Aug  1 16:55 known_hosts
-rw-------  1 ubuntu ubuntu 3774 Jul 31 10:19 known_hosts.old
-rw-------  1 ubuntu ubuntu 1675 Jul 31 10:17 ubuntu
-rw-r--r--  1 ubuntu ubuntu  398 Jul 31 10:17 ubuntu.pub

# Ops Manager 설치 후 접속 방법
$ ssh -i {private-key} ubuntu@{opsmanager-ip}
```

### 2.2. Ops Manager Download
- Pivotal Network에서 신규 Ops Manager v2.6.5를 Download 한다.

### 2.3. 기존 Ops Manager Export Installation
- Ops Manager Web 화면에서 [Setting -> Export Installation Setting] 화면에서 v2.5 Version의 Ops Manager 설정 정보를 Export 한다.

### 2.3. Ops Manager Install
- 기존 v42.5 Ops Manager 전원 Off
- vShpere 환경에 Ops Manager를 설치한다 (기존 Ops Manager와 같은 Network 정보).
[OpsManager 설치 가이드](https://docs.pivotal.io/pivotalcf/2-6/om/vsphere/deploy.html)

### 2.5. Ops Manager Import
- 설치 완료 한 Ops Manager 화면에서 Upgrade, Import Existing Installation 버튼을 클릭하고 v2.5의 Installation.zip 파일을 Import 시킨다.
- Decryption Passphrase를 반드시 올바르게 입력한다.

### 2.6. Bosh Director Apply Change
- Bosh Director Config를 확인하고 Bosh Director만 Apply Change를 클릭 한다.

## 3. Issues

### 3.1. Ops Manager Install시 Error 발생 (Pipeline)
- Ops Manager Install Pipeline 실행 중에러 발생 
- Concourse에서 실행되는 Pipeline의 Task 실행 Container가 vSphere Cluster Host Name을 찾지 못하는 에러 발생
```
02-08-19 03:57:23] Uploading pivotal-ops-manager-disk1.vmdk... Error: Post https://pvpsdmgt1/nfc/524de607-a693-df3a-f8c4-6d4a6d936255/disk-0.vmdk: dial tcp: lookup pvpsdmgt1 on 172.30.9.9:53: no such host
govc[err]: govc: Post https://pvpsdmgt1/nfc/524de607-a693-df3a-f8c4-6d4a6d936255/disk-0.vmdk: dial tcp: lookup pvpsdmgt1 on 172.30.9.9:53: no such host
Error: could not create the vm: unexpected error: govc error: govc error: exit status 1
```
- 해결 방법
	- 1. Container 실행 Image에 vSphere Cluster Host Name을 Mapping
	- 2. Concourse Worker의 DNS 주소에 Bind9 구성
	- 3. Task가 실행 될 때 hosts 파일을 수정

### 3.2. Ops Manager Install시 Error 발생 (Pipeline)
- Ops Manager Install 시 Public Key에 대한 Config Property Error 발생

```
+ p-automator upgrade-opsman --config config/dev-leedh/config/opsman-2.6.5.yml --env-file env/dev-leedh/leedh-env/env.yml --image-file image/ops-manager-vsphere-2.6.5-build.173.ova --state-file generated-state/state.yml --installation installation/dev-leedh-installation-20190730.zip --vars-file vars/dev-leedh/vars/opsman-vars.yml

[](https://concourse.{domain}/teams/main/pipelines/190730-leedh-upgrade/jobs/upgrade-opsman/builds/3#L5d40e3d0:18)09:45:24

Error: could not unmarshal config file: yaml: unmarshal errors:

[](https://concourse.{domain}/teams/main/pipelines/190730-leedh-upgrade/jobs/upgrade-opsman/builds/3#L5d40e3d0:19)09:45:24

 line 14: field ssh_pubilc_key not found in type vmmanagers.VsphereConfig

[](https://concourse.{domain}/teams/main/pipelines/190730-leedh-upgrade/jobs/upgrade-opsman/builds/3#L5d40e3d0:20)
```
- 해결 방법: Ops Manager Configuration 파일을 수정

```
$ cat opsman-config.yml

---
opsman-configuration:
 vsphere:
 vcenter:
 url: xxx.xxx.xxx.xxx
 username: xxxxxxx
 password: xxxxxxx
 datastore: xxxxxxxxx
 ca_cert: 
 host:                  # vCenter host to deploy Ops Manager in
 datacenter: xxxxxxxx
 resource_pool: xxxxxxxxxxx      # or /<Data Center Name>/host/<Cluster Name>
 folder: 
 insecure: 1                            # default: 0 (secure); 1 (insecure)
 disk_type: thin                          # example: thin|thick
 private_ip: xxx.xxx.xxx.xxx
 dns: xxx.xxx.xxx.xxx
 ntp: xxx.xxx.xxx.xxx
 ssh_password: xxxxx
 ssh_public_key: "ssh-rsa  AAAAB3NzaC1yc2EAAAADAQABAAABAQDXvvapfAk273+2wFCaVvADbD3ESz23zMBdDfBTfLQ81N25+2z0Iyo+rFuFUb1J6AZ....생략"
 hostname: 
 network: xxxxxxxxxxxx        # vcenter network to deploy to
 netmask: xxx.xxx.xxx.xxxx
 gateway: xxx.xxx.xxx.xxxx
 vm_name: leedh-OpsMan-2.6                      # default: Ops_Manager
 memory: 8                                # default: 8 GB
 cpu: 1                                   # default: 1
```
