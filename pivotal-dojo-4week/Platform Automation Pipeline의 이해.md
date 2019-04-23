## Platform Automation Pipeline의 이해

### 1. Download Product (수동)

#### 1.1  platform automation for PCF 

	- network pivotal.io 
	- platform-automation-tasks-(.*).zip
	- platform-automation-image-(.*).tgz
	
	=> s3에 업로드 https://minio/pivnet_products/*

#### 1.2 PCF binary
http://docs.pivotal.io/platform-automation/v2.1/reference/pipeline.html#retrieving-external-dependencies

	- network pivotal.io 
	 - opsman ova # ops manager 설치 파일, version upgard를 하기 위해 2.4.8 버전 다운로드
	- PAS: tile, stemcell  # PCF 설치 바이너리 파일, version upgard를 하기 위해 2.4 버전 다운로드
	- healthwatch: tile, stemcell
	
	=> s3에 업로드 https://minio/pivnet_products/*

#### 1.3  docker image for pipeline
	-> harbor

#### 1.4 s3(minio) 디렉토리 구조에 다운로드 파일을 업로드
		
	|-- pivnet-products
	|   |-- cf-2.4.5-build.25.pivotal
	|   |-- healthwatch-stemcell
	|   |   `-- bosh-stemcell-97.71-vsphere-esxi-ubuntu-xenial-go_agent.tgz
	|   |-- pas-stemcell
	|   |   |-- bosh-stemcell-170.45-vsphere-esxi-ubuntu-xenial-go_agent.tgz
	|   |   `-- bosh-stemcell-170.48-vsphere-esxi-ubuntu-xenial-go_agent.tgz
	|   |-- pcf-vsphere-2.4-build.171.ova
	|   `-- p-healthwatch-1.4.5-build.41.pivotal
	`-- platform-automation
    |-- platform-automation-image-2.1.1-beta.1.tgz
    `-- platform-automation-tasks-2.1.1-beta.1.zip

### 2. 실습 환경 요약

#### 2.1. minio 서버 실습 환경
- http://{minio_doamin}:9000/minio/pivnet-products/

#### 2.2. harbor 서버 실습 환경
-  http://{harbor_domain}
##### 2.2.1. docker publl
	- docker pull harbor.pcf.posco.co.kr/concourse/pivotalcf/pivnet-resource:latest

#### 2.3. concourse 서버 실습 환경
 - http://{concourse_domain}

#### 2.4. 실습용 DNS 서버 주소
-  172.30.161.118 # bind9을 통해 실습 환경 도메인 서버 구축


### 3. Pipeline Test

fly -t leedh login -c https://concourse.pcf.posco.co.kr/ -k -u admin -p 'Posco!23
fly -t leedh sp -p test-resources-leedh -c test-resources.yml -l ./test-resources-params.yml
fly -t leedh unpause-pipeline -p test-resources-leedh

setting credhub

fly -t leedh sp -p test-resources-leedh -c test-resources.yml


Expected to find variables: access_key_id
bucket
endpoint
region
secret_access_key


ubuntu@TLKPCFJB1:~/concourse/concourse-bosh-deployment/cluster$ credhub find
credentials:
- name: /concourse/main/s3_access_key_id
  version_created_at: "2019-04-23T05:56:15Z"
- name: /concourse/main/s3_secret_access_key
  version_created_at: "2019-04-23T05:53:31Z"
- name: /concourse/shpark/s3_secret_access_key
  version_created_at: "2019-04-23T05:45:16Z"
- name: /concourse/shpark/s3_access_key_id
  version_created_at: "2019-04-23T05:44:29Z"
