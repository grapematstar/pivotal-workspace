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
# Credhub을 Concourse에서 사용하기 위해 아래 링크에서 yml 파일을 다운로드 한다.
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

# 설치 중 Schema-validation: missing column [credential_uuid] in table [permission] 에러가 발생 할 경우 아래와 같이 조치
 bosh/0:/var/vcap/packages/postgres-10/bin# ./psql -U vcap credhub -h /var/vcap/data/postgres-10/tmp/
```
#### 1.1.5. Concourse Deploy 확인
```
$ bosh -e ${alias-env} vms

Deployment 'concourse'

Instance                                     Process State  AZ    IPs            VM CID                                   VM Type  Active
db/a5acd5d4-0848-4eb7-bdd1-1abe234797ad      running        mgmt  172.28.83.196  vm-98c64825-6b60-4020-912a-68621d3e8eb2  micro    true
web/1a4d6bab-51f0-4754-bf8f-d674850904da     running        mgmt  172.28.83.200  vm-72593326-2bf7-4e50-8330-b1525036bf8e  micro    true
worker/028fd516-14de-4d6e-847f-8102ae1169ba  running        mgmt  172.28.83.202  vm-d6b0b89a-7a14-40a5-b44b-37648e98aabe  large    true
worker/9e1585d3-2f9e-42b2-80fb-e975921a1bde  running        mgmt  172.28.83.197  vm-559b532c-d482-41bb-bcec-9329a0c1df33  large    true
worker/be3b0cb4-1de2-4bb9-b98c-73d5063ccaf9  running        mgmt  172.28.83.203  vm-14175096-7730-424f-87b9-b51988dbc879  large    true
worker/ce99482f-4196-4781-8ee6-6522f69b59ee  running        mgmt  172.28.83.204  vm-12947d11-2cd6-4bfd-9d24-33c96d1d7f4c  large    true

browser: https://concourse.bosh.co.kr
```
#### 1.1.6. Concourse Test 환경 구축
```
$ credhub api https://concourse.bosh.co.kr:8844 --skip-tls-validation
$ credhub login    --client-name=admin  --client-secret=admin

$ credhub set -t value -n /concourse/main/test/hello -v test

$ credhub get -n /concourse/main/test/hello
id: 3cd51b78-426f-4145-b94e-baacf16c383d
name: /concourse/main/test/hello
type: value
value: test
```
