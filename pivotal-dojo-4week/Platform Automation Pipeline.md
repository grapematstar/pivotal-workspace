
## Platform Automation Pipeline

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

github 주소 https://github.com/do-workspace/platform-automation-pipelines-template

	./test-resources.yml # pipeline task yaml
	./test-resources-params.yml # task param yaml
	
	test-resources-params.yml 파일에 minio 정보 수정 후 아래의 명령어 실행
	
	# 수정 값
	endpoint: http://{minio_domain}:9000
	access_key_id: admin
	secret_access_key: Posco!23
	region: ""
	bucket: "platform-automation"
	
	$ fly -t leedh login -c https://concourse.pcf.posco.co.kr/ -k -u admin -p 'Posco!23
	$ fly -t leedh sp -p test-resources-leedh -c test-resources.yml -l ./test-resources-params.yml
	
	# 아래 명령어를 재 실행하여 정상 동작임을 확인한다.
	$ fly -t leedh unpause-pipeline -p test-resources-leedh
	
	

#### 3.1 .setting credhub

	./test-resources.yml # pipeline task yaml
	./test-resources-params.yml # task param yaml

	test-resources-params.yml 파일에 minio 정보 수정 후 아래의 명령어 실행
	
	# 수정 값
	endpoint: http://{minio_domain}:9000
	access_key_id: ((s3_access_key_id))
	secret_access_key: ((s3_secret_access_key))
	region: ""
	bucket: "platform-automation"

	$ fly -t leedh sp -p test-resources-leedh -c test-resources.yml

	# 아래의 예외 코드를 concourse 화면에서 확인 한다.
	Expected to find variables: access_key_id
	bucket
	endpoint
	region
	secret_access_key

	# concourse credhub에 로그인 & s3_access_key_id 등 변수를 넣는다.
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
	
	# 아래 명령어를 재 실행하여 정상 동작임을 확인한다.
	$ fly -t leedh sp -p test-resources-leedh -c test-resources.yml

### 4. Ops Manager Pipeline Install

#### 4.1 Ops Manager Pipeline Git Hub 다운로드
 
 https://github.com/myminseok/pivotal-docs/blob/master/platform-automation/install_opsman.md
```
platform-automation-configuration-template
└── dev-1
    ├── config
    │   ├── auth.yml    
    │   └── opsman-2.4.yml
    ├── download-product-configs
    ├── env
    │   └── env.yml
    ├── generated-config
    ├── state
    └── vars
```
* issue 
	* concourse는 sp할 때만 credhub를 참조하지만 task에서 따로 사용할 경우 credhub의 접속 정보가 필요하다.
	* Credhub team/ foundation이 같을 경우 설정이 서로 엉킬수 있어 Ops manager 타겟이 잘 못될 수 있다.

#### 4.2. Ops Manager Pipeline Manifest 수정

##### 4.2.1. Ops Manager Install Pipeline에 사용 할 Credhub 정보 저장

	$ credhub set -t value -n /concourse/main/git_user_email -v {git email}
	$ credhub set -t value -n /concourse/main/git_user_username -v leedonghyean

	# register ssh key for git. ex) ~/.ssh/id_rsa

	$ credhub set -t rsa  -n /concourse/main/git_private_key  -p ~/.ssh/id_rsa

	$ cd concourse-bosh-deployment/cluster
	$ bosh int ./creds.yml --path /atc_tls/certificate > atc_tls.cert
	$ credhub set -t certificate -n /concourse/main/credhub_ca_cert -c ./atc_tls.cert

	#grep concourse_to_credhub ./creds.yml
	$ credhub set -t user -n /concourse/main/credhub_client -z concourse_to_credhub -w udwusbr76j4u9zxoty9n

	$ credhub set -t user  -n /concourse/dev-leedh/opsman_admin -z {opsman id} -w {opsman pwd}
	$ credhub set -t value -n /concourse/dev-leedh/decryption-passphrase -v "{opsman pwd}"
	$ credhub set -t value -n /concourse/dev-leedh/opsman_target -v https://{opsman ip}

##### 4.2.2. Ops Manager Config 파일을 수정

	# 아래 Manifest 파일을 맞는 vSphere 환경으로 수정
	# credhub_dev는 concourse의 credhub 저장소 명칭
	do-workspace/platform-automation-configuration-template/{credhub_dev}/config/opsman-2.4.yml
	
	# 아래 명령어를 통해 pipeline을 생성하고 Concourse 화면에서 install-opsman을 동작 시킨다.
	$ fly -t leedh sp -p install-opsman-leedh -c install-opsman.yml -l ./install-opsman-params.yml

##### 4.2.3. Ops Manager Director Config 파일을 추출
- Opsmanager 서버에 로그인 후 Bosh Director Tile의 Config를 vSphere  환경에 맞게 수정
- Concourse 화면에서  staged-director-config를 실행 시킨다.

		위 실행 결과 값은 아래 파일 구조의 자동으로 git commit & push 후 director.yml로 생성된다.
		platform-automation-configuration-template
		└── dev-1
		    ├── generated-config
			    ├── director.yml

- 아래 director.yml 파일 내용을 아래 파일 구조의 director-2.4.yml에 적용시킨다.

		└── dev-1
		    ├── config
		    │   ├── director-2.4.yml 

##### 4.2.4. bosh director 설치
- 4.2.3 절차를 완료하였으면 Concourse 화면에서  install-director를 실행 시킨다.
- bosh director 설치가 정상적으로 완료 됬음을 확인 한다.

### 5. Pas Pipeline Install
