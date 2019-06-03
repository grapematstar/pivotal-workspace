## 1. BOSH Deploy Minio(s3) Storage

- 전제 조건
	- bosh가 설치 되어 있어야 한다.
	- 외부 환경과 인터넷 통신이 되는 Jumpbox가 존재해야한다.
- 가이드
	-  https://github.com/minio/minio-boshrelease
	- bosh release: https://bosh.io/releases/github.com/minio/minio-boshrelease?all=1
-  고려 사항: Opensource Minio는 스케일 아웃 구성이 가능하지 않음으로 설치 & 사용 전 협의 해야 한다.

### 1.1. Minio Deploy

#### 1.1.1. Minio Release & Stemcell Upload
```
# minio release download 주소
https://bosh.io/releases/github.com/minio/minio-boshrelease?all=1
$ wget https://bosh.io/d/github.com/minio/minio-boshrelease?v=2019-04-09T01-22-30Z
# minio release bosh upload
$ bosh -e ${alias-env} upload-release ${minio-release}

#minio stemcell download 주소
https://bosh.io/stemcells/bosh-vsphere-esxi-ubuntu-trusty-go_agent
$ wget https://s3.amazonaws.com/bosh-core-stemcells/vsphere/bosh-stemcell-3586.79-vsphere-esxi-ubuntu-trusty-go_agent.tgz

# minio stemcell bosh upload
$ bosh -e ${ailas-env} upload-stemcell ${stemcell}
```

#### 1.1.2. Minio Deployment Clone
```
# minio deployment를 clone 한다.
$ git clone https://github.com/minio/minio-boshrelease.git

# minio deployment git manifest 구조
manifest/manifest-dist-example.yml # 인스턴스 수를 분산 시 사용 Main Manifest 파일
manifest/manifest-fs-example.yml # 단일 VM을 사용 할 경우 Main Manifest 파일
manifest/manifest-nas-example.yml # NAS Mount 디렉토리로 minio를 배치하는 경우 사용하는 Main Manifest 파일. Cloud Foundry의 NFS 관련 Release 사용 추가 해야 한다.
```

#### 1.1.3. Minio Deployment Manifest 파일 편집 & deploy
```
# manifest.yml bosh cloud-config를 참고하여 편집 한다.
# 주로 변경되는 내용은 stemcells, networks, azs, vm_type, disk_type이 있다.
# 편집이 완료되면 deploy.sh 스크립트를 작성한다.
$ cat deploy.sh
bosh deploy -d minio manifests/manifest-dist-example.yml \
    -v minio_deployment_name=minio \
    -v minio_accesskey=admin \
    -v minio_secretkey=${MINIO_PASSWORD}
```
#### 1.1.4. Minio Deploy 확인
```
$ bosh -e ${alias-env} vms

Deployment 'minio'

Instance                                    Process State  AZ    IPs            VM CID                                   VM Type  Active
minio/b43434de-6a22-4ecf-b4be-e43834517d8e  running        mgmt  172.28.83.173  vm-a2dd1c48-9838-486b-b907-8aa207f6cb4c  micro    true

browser: https://<IP>:9000

issue_1: minio web ui에서 버킷 디렉토리 생성은 불가능 하다. VM에 접근하여 /var/vcap/store/minio에서 버킷 정보를 생성해줘야함.
```
