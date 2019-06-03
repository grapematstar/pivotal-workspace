## 1. BOSH Deploy Concourse(CI/CD Tool)

- 전제 조건
	- bosh가 설치 되어 있어야 한다.
	- 외부 환경과 인터넷 통신이 되는 Jumpbox가 존재해야한다.
- 고려사항

#### 1.1.1. Concourse Release & Stemcell Upload
```
# concourse release download 주소
https://bosh.io/releases/github.com/concourse/concourse-bosh-release?all=1
$ wget https://bosh.io/d/github.com/concourse/concourse-bosh-release?v=5.1.0
# concourse release bosh upload
$ bosh -e ${alias-env} upload-release ${concourse-release}

#concourse stemcell download 주소
https://bosh.io/stemcells/bosh-vsphere-esxi-ubuntu-xenial-go_agent
$ wget https://s3.amazonaws.com/bosh-core-stemcells/250.38/bosh-stemcell-250.38-vsphere-esxi-ubuntu-xenial-go_agent.tgz

# concourse stemcell bosh upload
$ bosh -e ${ailas-env} upload-stemcell ${stemcell}
```

#### 1.1.2. Concourse Deployment Clone
```
# concourse deployment를 clone 한다.
$ git clone https://github.com/concourse/concourse-bosh-deployment.git

# concourse deployment git manifest 구조
/lite/* # Bosh Director가 필요 없이 단일 VM에 Concourse 설치
/cluster/* # Bosh Direcotr를 통해 설치하며 Bosh를 통해 LifeCycle을 관리 할 수 있는 Concourse 설치
/cluster/concourse.yml # Concourse 설치 main manfiest 파일
/cluster/operation/static-web.yml # web ip를 직접 지정 하는 경우 사용하는 manifest 파일
/cluster/operation/basic-auth.yml # basic 인증 admin/password를 사용 할 경우 사용하는 manifest 파일
/cluster/operation/privileged-http.yml # atc 컴포넌트가 80 PORT로 bind 하도록 설정하는 manifest 파일
/cluster/operation/privileged-https.yml # atc 컴포넌트가 443 PORT로 bind 하도록 설정하는 manifest 파일
/cluster/operation/tls.yml # tls 인증서를 넣어주는 manifest 파일
/cluster/operation/tls-vars.yml # tls 인증서를 자동 생성 해주는 manifest 파일
```

#### 1.1.3. Concourse Deployment Manifest 파일 편집 & deploy
```
# add-credhub-uaa-to-web.yml 다운로드 후 추가 다운로드 url 
$ wget  https://raw.githubusercontent.com/pivotalservices/concourse-credhub/master/operations/add-credhub-uaa-to-web.yml
- concourse-bosh-deployment/cluster/operations/add-credhub-uaa-to-web.yml
# manifest.yml bosh cloud-config를 참고하여 편집 한다.
# 주로 변경되는 내용은 stemcells, networks, azs, vm_type, disk_type이 있다.
# 편집이 완료되면 deploy.sh 스크립트를 작성한다.
$ cat deploy.sh
export concourse_elb=concourse.bosh.co.kr  # domain 설정 해당 domain에 대한 hosts 파일이 등록되어 있어야 한다.
bosh deploy -e {ailas-env} --no-redact -d concourse concourse.yml \
  -o operations/basic-auth.yml \
  -o operations/privileged-http.yml \
  -o operations/privileged-https.yml \
  -o operations/static-web.yml \
  -o operations/tls.yml \
  -o operations/tls-vars.yml \
  -o operations/scale.yml \
  -o operations/worker-ephemeral-disk.yml \
  -o operations/add-credhub-uaa-to-web.yml \  #download 후 생성
  --var web_ip=172.28.xx.xxx \ #staticip지정
  --var network_name=mgmt-network \  #수정
  --var external_host=$concourse_elb \
  --var external_url=https://$concourse_elb \
  --var concourse_host=$concourse_elb \
  --var web_vm_type=micro \
  --var worker_ephemeral_disk=100GB_ephemeral_disk \
  --var worker_vm_type=large \
  --var db_vm_type=micro \
  --var db_persistent_disk_type=10240 \
  --var web_instances=1 \
  --var worker_instances=1 \
  --var deployment_name=concourse \
  --var local_user.username=admin \
  --var local_user.password=admin
