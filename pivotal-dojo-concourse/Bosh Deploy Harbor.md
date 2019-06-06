## 1. BOSH Deploy Harbor(Private Docker Repo)

- 전제 조건
	- bosh가 설치 되어 있어야 한다.
	- 외부 환경과 인터넷 통신이 되는 Jumpbox가 존재해야한다.
- 가이드
	-  https://github.com/vmware/harbor-boshrelease
- 고려사항
	- HA 구성이 지원되지 않는다. 


### 1.1. Harbor Deploy
- private docker repo harbor는 docker 이미지를 저장하고 배포하는 registry 서버
- 사용자 관리, Access 제어, 활동 감사 등 고급 보안 기능을 제공
- Build 실행 환경과 같은 서버에 위치하면 image 전송률이 빨라 진다.

#### 1.1.1. Habor Release & Stemcell Upload
```
# harbor release download 주소
https://bosh.io/releases/github.com/vmware/harbor-boshrelease?all=1
$ wget https://bosh.io/d/github.com/vmware/harbor-boshrelease?v=1.7.5-build.10
# harbor release bosh upload
$ bosh -e ${alias-env} upload-release ${harbor -release}

#harbor stemcell download 주소
https://bosh.io/stemcells/bosh-vsphere-esxi-ubuntu-xenial-go_agent
$ wget https://s3.amazonaws.com/bosh-core-stemcells/250.38/bosh-stemcell-250.38-vsphere-esxi-ubuntu-xenial-go_agent.tgz

# harbor stemcell bosh upload
$ bosh -e ${ailas-env} upload-stemcell ${stemcell}
```

#### 1.1.2. Habor Deployment Clone
```
# harbor deployment를 clone 한다.
$ git clone https://github.com/vmware/harbor-boshrelease.git

# harbor deployment git manifest 구조
manifest/harbor.yml # harbor 설치 main mainfest 파일
manifest/deployment-vsphere-uaa.yml # harbor 설치 시 UAA 사용 Manifest 파일
manifest/deployment-vsphere.yml # harbor 설치 시 bosh dns 사용 Manifest 파일
```

#### 1.1.3. Habor Runtime Config Upload
```
### vi manifests/runtime-config-bosh-dns.yml
- include, exclude 설명 참고: https://bosh.io/docs/runtime-config/
- minio는 제외하도록 수정
addons:
- include:
    deployments:
    - harbor-deployment
  exclude:
    deployments:
    - minio
 
# Bosh Runtime Config Upload
$ bosh -e ${alias-env} update-runtime-config runtime-config-bosh-dns.yml
```

#### 1.1.4. Habor Deployment Manifest 파일 편집 & deploy
```
# manifest.yml bosh cloud-config를 참고하여 편집 한다.
# 주로 변경되는 내용은 stemcells, networks, azs, vm_type, disk_type이 있다.
# admin_password: ((harbor_admin_password)) # manifest/harbor.yml의 ((harbor_admin_password))를 변경한다.
# 편집이 완료되면 deploy.sh 스크립트를 작성한다.
$ cat deploy.sh
bosh -e {ailas-env}-n -d harbor-deployment deploy --recreate manifests/harbor.yml -v hostname=harbor.local
```
#### 1.1.5. Habor Deploy 확인
```
$ bosh -e ${alias-env} vms

Deployment 'harbor2'

Instance                                         Process State  AZ    IPs            VM CID                                   VM Type  Active
harbor-app/d88f93ad-1bcc-481c-a0c8-bb7ca4199e37  running        mgmt  172.28.83.201  vm-9dd64b1e-5498-4c18-9585-bdab3b831188  -        true

browser: https://<IP>:443
```
#### 1.1.6. Habor Test 환경 구축
```
# bosh harbor는 server.crt 인증서 파일을 시스템에 설치하지 않았음으로 해당 registry가 있는 서버에 아래 harbor hostname을 추가시킨다.
$ cat /etc/docker/daemon.json 
{
  "insecure-registries" : ["${HARBOR_HOST}"]
}
# insecure-registries를 적용하기 위해 harbor를 재시작 한다.
$ sudo service docker restart
```
