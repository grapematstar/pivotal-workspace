
# BOSH 설치

## 1. Jumpbox 구성

### 1.1.  jumpbox 서버는 BOSH 설치와 BOSH의 Director를 설정하여 VM(PCF, Concourse, Mysql) 등을 배포 및 관리 하기 위해 필요한 패키지 및 라이브러리, Manifest 파일 등의 환경을 가지고 있는 배포 작업 및 실행 서버이다. 환경 구성에 있어서 전제조건으로 jumpbox서버는 외부와 통신이 가능해야 한다.

### 1.2. jumpbox 구성을 위한 우분투 다운로드 링크 16.04 LTS, 64 bit

[http://releases.ubuntu.com/xenial/](http://releases.ubuntu.com/xenial/)

### 1.3. 내부 점프박스 설정 가이드

[https://bosh.io/docs/cli-v2-install/](https://bosh.io/docs/cli-v2-install/)

**※ Dojo 실습에서는 External, Internal jumpbox 2개의 서버를 구성하여 외부와 통신이 가능한 External jumpbox에서 패키지를 다운로드 후 Internal jumpbox 서버에 파일을 복사하여 구성하였다.**

## 2. bosh deploy manifest &dependency & binary 설치

### 2.1. bosh cli 다운로드(external jumpbox)
	$ wget https://github.com/cloudfoundry/bosh-cli/releases/download/v5.4.0/bosh-cli-5.4.0-linux-amd64
	$ sudo cp bosh-* /usr/local/bin/bosh 

### 2.1. bosh create-env를 위한 의존성 바이너리를 내려받기위해 외부 점프박스에서 아래 명령을 수행한다. (external jumpbox)

	$ sudo su
	$ mkdir bosh-binaries
	$ cd bosh-binaries
	$ apt-get download  build-essential zlibc zlib1g-dev ruby ruby-dev openssl libxslt-dev libxml2-dev libssl-dev libreadline6 libreadline6-dev libyaml-dev libsqlite3-dev sqlite3 libcurl3

### 2.2.  패키지 파일 설치 및 목록 확인 한다. (external jumpbox)
	$ dpkg -i *.deb 
	$ apt-get install -f
	$ apt list --installed
	
### 2.3. cf cli 다운로드(external jumpbox)

https://docs.cloudfoundry.org/cf-cli/install-go-cli.html#pkg-linux

	$ wget https://cli.run.pivotal.io/stable?release=debian64&source=github
	$ dpkg -i stable?release=debian64

### 2.4. Docker 다운로드(external jumpbox)

### 2.4.1. 외부에서 내려받은 Docker 이미지를 내부 하버에 밀어넣기 위해 내부 점프박스에 Docker 엔진이 필요하다.  외부 점프박스에서 받아 설치해보고 내부 점프박스에 설치.

	$ apt-get download libltdl7
	$ apt download docker.io
	$ dpkg -i libltdl7_2.4.6-0.1_amd64.deb 
	$ dpkg -i docker.io_18.09.2-0ubuntu1~16.04.1_amd64.deb 
	$ apt-get install -f
	root@TLKPCFJB1:/home/ubuntu# docker ps
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

### 2.5. bosh deployment manifest 파일 다운로드(external jumpbox)

	$ mkdir bosh-1 && cd bosh-1
	$ git clone https://github.com/cloudfoundry/bosh-deployment

### 2.6. credhub 다운로드(external jumpbox)
	$ wget https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/2.4.0/credhub-linux-2.4.0.tgz
	$ tar xf credhub-linux-2.4.0.tgz
	
## 3. bosh 설치 - external-jumpbox

### 3.1 bosh 설치 stemcell과 release 파일을 다운로드 하기 위해 manifest 파일의 cpi 정보를 잘못입력하고 설치를 진행한다.(external jumpbox)

	ex)
	$ /bosh-1$ cat deploy.sh 
	$ bosh create-env bosh-deployment/bosh.yml \
    --state=state.json \
    --vars-store=creds.yml \
    -o bosh-deployment/vsphere/cpi.yml \
    -o bosh-deployment/jumpbox-user.yml \
    -o bosh-deployment/uaa.yml \
    -o bosh-deployment/credhub.yml \
    -o bosh-deployment/misc/config-server.yml \
    -v director_name=bosh-1 \
    -v internal_cidr=172.28.83.0/24 \
    -v internal_gw=172.28.83.1 \
    -v internal_ip=172.28.83.60 \
    -v network_name="Service_172.28.83.x_v442" \
    -v vcenter_dc=Gwangyang_MES30 \
    -v vcenter_ds=PCF_Image \
    -v vcenter_ip=172.31.76.100 \
    -v vcenter_user=--- \
    -v vcenter_password=---- \
    -v vcenter_templates=bosh-1-templates \
    -v vcenter_vms=bosh-1-vms \
    -v vcenter_disks=bosh-1-disks \
    -v vcenter_cluster=pivotal_worker1

	ubuntu@dojojump2:~/bosh-1$ ./deploy.sh
	Deployment manifest: '/home/ubuntu/bosh-1/bosh-deployment/bosh.yml'
	Deployment state: 'state.json'

	Started validating
	  Downloading release 'bosh'... Skipped [Found in local cache] (00:00:00)
	  Validating release 'bosh'... Finished (00:00:01)
	  Downloading release 'bpm'... Skipped [Found in local cache] (00:00:00)
	  Validating release 'bpm'... Finished (00:00:01)
	  Downloading release 'bosh-vsphere-cpi'... Skipped [Found in local cache] (00:00:00)
	  Validating release 'bosh-vsphere-cpi'... Finished (00:00:00)
	  Downloading release 'os-conf'... Skipped [Found in local cache] (00:00:00)
	  Validating release 'os-conf'... Finished (00:00:00)
	  Downloading release 'uaa'... Skipped [Found in local cache] (00:00:00)
	  Validating release 'uaa'... Finished (00:00:02)
	  Downloading release 'credhub'... Skipped [Found in local cache] (00:00:00)
	  Validating release 'credhub'... Finished (00:00:01)
	  Downloading release 'config-server'... Skipped [Found in local cache] (00:00:00)
	  Validating release 'config-server'... Finished (00:00:01)
	  Validating cpi release... Finished (00:00:00)
	  Validating deployment manifest... Finished (00:00:00)
	  Downloading stemcell... Skipped [Found in local cache] (00:00:00)
	  Validating stemcell... Finished (00:00:05)
	Finished validating (00:00:16)

**※ 설치에 필요한 stemcell과 release 파일을 다운로드 완료 되면 bosh cli의 pid를 중단시키고 home 디렉토리에 .bosh 폴더가 생성 됬는지 확인한다.**

## 4. external jumpbox -> internal jumpbox
### 4.1  extenal jumpbox에서 실행 결과인 dpkg 파일, .bosh, bosh, cf, credhub, deployment manifest 파일들을 압축하여 intenal jumpbox에 scp 한다.

	ex)
	$ cd /home/ubuntu

	$ tar zcf bosh-cache.tar.gz .bosh
	$ scp bosh-cache.tar.gz ubuntu@TLKPCFJB1:/home/ubuntu
	
## 5. Bosh 설치 - internal-jumpbox
### 5.1 external에서 가져온 cli 들을 ubuntu 실행 파일 디렉토리에 넣는다.

	ex) $ mv bosh /usr/local/bin
	ex) $ mv cf /usr/local/bin
	ex) $ mv credhub /usr/local/bin
	
### 5.2 .bosh 폴더를 home 디렉토리에 압축을 해제 한다.
	ex) $ tar xvf bosh-catch.tgz

### 5.3. bosh deployment deployment manifest 파일을 수정 & deploy script 작성

	ubuntu@TLKPCFJB1:~/bosh-1$ cat deploy.sh 
	bosh create-env bosh-deployment/bosh.yml \ # bosh 설치 manifest
    --state=state.json \ # bosh 설치 정보
    --vars-store=creds.yml \ # id/pwd/cert 등 인증서 파일 정보
    -o bosh-deployment/vsphere/cpi.yml \ # vsphere 인프라 환경 정보 manifest
    -o bosh-deployment/jumpbox-user.yml \ # bosh vm ssh 접속 정보 manfiest
    -o bosh-deployment/uaa.yml \ # bosh uaa oauth2 인증 정보 maninfest
    -o bosh-deployment/credhub.yml \ # bosh credhub key/value 서버 정보 manfiest
    -o bosh-deployment/misc/config-server.yml \ # bosh config 서버 정보 manifest
    -v director_name=bosh-1 \ // bosh 디렉터 명
    -v internal_cidr=172.28.83.0/24 \ # bosh 설치 cidr
    -v internal_gw=172.28.83.1 \ # bosh 설치 cidr의 gw
    -v internal_ip=172.28.83.60 \ # bosh 설치 ip
    -v network_name="Service_172.28.83.x_v442" \ # bosh 설치 네트워크 명
    -v vcenter_dc=Gwangyang_MES30 \ # vshpere 데이터 센터 명
    -v vcenter_ds=PCF_Image \ # vshpere  데이터 스토어 명
    -v vcenter_ip=172.31.76.100 \ # vshpere ip
    -v vcenter_user=pasadmin@xxxx.local \ # vshpere id
    -v vcenter_password=VMware1! \ # vshpere pwd
    -v vcenter_templates=bosh-1-templates \ # vshpere template 명 (자동 생성)
    -v vcenter_vms=bosh-1-vms \ # vshpere vm template 명 (자동 생성)
    -v vcenter_disks=bosh-1-disks \ # vshpere disk template 명 (자동생성)
    -v vcenter_cluster=pivotal_worker1 # vshpere cluster 명

### 5.4. bosh deploy script 실행
	$ chmod +x deploy.sh
	$ ./deploy.sh
	Deployment manifest: '/home/ubuntu/bosh-1/bosh-deployment/bosh.yml'
	Deployment state: 'state.json'

	Started validating
	  Downloading release 'bosh'... Skipped [Found in local cache] (00:00:00)
	  Validating release 'bosh'... Finished (00:00:01)
	  Downloading release 'bpm'... Skipped [Found in local cache] (00:00:00)
	  
	  -----------------------------생략---------------------------
	  
	  Compiling package 'lunaclient/b922e045db5246ec742f0c4d1496844942d6167a'... Skipped [Package already compiled] (00:00:)
	  Compiling package 'credhub/2e48e798c636d46064be8a2a1dbd2733e0d163e9'... Skipped [Package already compiled] (00:00:02)
	  Compiling package 'bpm/d139b63561eaa3893976416be9668dea539bf17d'... Skipped [Package already compiled] (00:00:00)
	  Updating instance 'bosh/0'... Finished (00:02:09)
	  Waiting for instance 'bosh/0' to be running... Finished (00:02:15)
	  Running the post-start scripts 'bosh/0'... Finished (00:00:27)
	Finished deploying (00:20:17)

	Stopping registry... Finished (00:00:00)
	Cleaning up rendered CPI jobs... Finished (00:00:00)

## 6. bosh target 설정
### 6.1. 설치 완료한 bosh를 사용하기 위해 bosh alias와 login을 등록 한다.
	ex)
	$ bosh alias-env b -e {$bosh 설치 ip} --ca-cert <(bosh int {$credfile path} --path /director_ssl/ca)

	$ export BOSH_CLIENT=admin
	$ export BOSH_CLIENT_SECRET=`bosh int ./creds.yml --path /admin_password`

### 6.2.  디렉터 정보 상세 확인
	ex)
	$ bosh -e {$bosh alias} env

### 6.3. bosh VM ssh접속
	ex)
	-  bosh ssh key 생성
	$ bosh int ./creds.yml --path /jumpbox_ssh/private_key > jumpbox_ssh.key

	-  jumpbox 계정으로 bosh 접속
	$ ssh  -i  jumpbox_ssh.key  jumpbox@{$bosh ip}

## 7. Credhub 설정
### 7.1. Credhub는 bosh를 통해 설치 할 vm들의 인증서/id/password 정보를 가지고 있는 서버이다. Credhub를 통해 PCF, VM 들의 uaa 정보/DB 정보 등을 확인 가능
	ex)
	$ grep credhub ./creds.yml
	ubuntu3@TLKPCFJB1:~/bosh-3$ cat login-credhub.sh 
	#!/bin/bash
	credhub api https://{bosh 설치 ip}:8844   --skip-tls-validation
	credhub login    --client-name=credhub-admin    --client-secret=afj3fyppgkstzrb7wr39
	
	$ chmod +x login-credhub.sh
	$./login-credhub.sh
	Login Successful

	$ credhub api
	https://172.18.53.151:8844

	$ credhub find
	credentials:
	[]
