## PAS?

### 1.  PAS Web Config

#### 1.1.  AZ and Network Assignments
- 싱글 VM에서 적용 되는 AZ 목록 ex) blobstore

#### 1.2. Domain
- apps -> 어플리케이션이 바인딩 되는 도메인
- sys -> 시스템에서 바인딩 되는 도메인
- 플랫폼팀과 개발팀의 업무적 도메인 충돌로 인하여 도메인을 2개로 분리하는 규칙

#### 1.3 Networking
- CF Deployment Manifest 파일에 적용되는 Network 
- Router IP: haproxy를 사용하지 않고 개별 L4 장치를 사용할 경우 연결 시킬 Router IP를 지정해줘야함 ,를 통해 여러개의 Router Static IP를 지정 할 수 있다. 빈칸일 경우 Random
- Haproxy: IP를 지정 할 경우 IP를 지정
- SSH Proxy IP: cf ssh를 할 경우 diego brain의 IP를 지정 # 내부에서는 가능하지만 외부에서는 접근 하기 어려워서 사용해야할수있음, L4를 사용 할 경우 지정해줘야함
- TCP Router: RabbitMq 같은 서비스를 포트하고 싶은 경우 사용해야 하는 TCP Router
- Certificates: Router와 Haproxy의 내부 서비스 간의 리다이렉트 시 사설 인증서가 포워딩 할 때 사용
- Logging of Client IPs in CF Router: Remote IP에 대한 Logging을 할 경우 
- SSL 인증서를 어디서 terminate 할 것 인가 (Infra LoadBalncer, HaProxy, Router)
- Enable Zipkin tracing headers on the Router: 분산 시스템(MSA) 형태의 어플리케이션 딴에서 API 호출에 대한 Zipkin Header를 삽입하여 추적이 가능하도록 함
- Routers reject requests for Isolation Segments: 별도의 Router와 Cell과 통신을 할 것인가에 대한 설정
- TCP Router를 외부 시스템과 연결해서 사용 할 수있다.

#### 1.3 Application Container
- 실제 컨테이너를 관리에 있는 Properties
- NFS LDAP: 인증된 NFS 만 사용하겠다는 Properties

#### 1.4 applicaton 3개는 apps manager에 관련있는 Properties


[vShpere PAS 구성도](https://docs.pivotal.io/pivotalcf/2-4/customizing/vsphere-nsx-t.html)

## PAS Pipeline

### 2.  PAS Pipeline 수정

#### 2.1.  platform-automation-pipelines-template/product/install-products.yml 수정
	# 실습은 PAS Install Pipeline을 구축하기 위함으로 install-products.yml의 apply-product-changes job에 configure-healthwatch 등 healthwatch에 대한 정보를 삭제한다.
	
	# PAS Pipeline Manfest 파일 수정이 완료 되면 아래 명령어를 통해 Concourse에 Pipeline을 생성한다.
	$ fly -t leedh sp -p install-products-leedh -c install-products.yml -l install-opsman-params.yml

#### 2.2.  PAS Pipeline 실행

##### 2.2.1. upload-and-stage-pas job을 실행
- pipeline을 실행 시킬 upload-and-stage-pas job을 실행하여 minio에 있는 platform-automation-image와 platform-automation-tasks를 컨테이너에 다운로드 한다.
- PAS를 설치 할 cf-2.4.5-build.25.pivotal를 컨테이너에 다운로드 한다.
- task의 om upload-product를 사용하여 PAS 타일을 Ops Manager에 업로드한다.

##### 2.2.2. Opsmanager의 PAS Tile에 정보를 입력한다.
- 도메인을 입력하고 bind9이 동작하고 있는 dns server에 해당 도메인을 연동 시킨다.
- Networking block 입력 시 생성한 cert/private key는 초기 pipeline 구성 시 config의 cf-env.yml 파일에 뽑아지지 않음으로 따로 저장한다.
- Concourse 화면의 staged-pas-config job을 실행 시킨다.

		위 실행 결과 값은 아래 파일 구조의 자동으로 git commit & push 후 director.yml로 생성된다.
		platform-automation-configuration-template
		└── dev-1
		    ├── generated-config
			    ├── cf-xxx.yml
- 위에서 뽑은 cf-xxx.yml 파일에 초기 Pipeline 설정임으로 아래 속성들을 수정 및 작성한다.
	
	-  `((uaa_service_provider_key_credentials.cert_pem))`-> tile에서 생성한 cert key 정보
	- `((uaa_service_provider_key_credentials.private_key_pem))`- tile에서 생성한 private key 정보
	- `credhub_key_encryption_passwords` passwords 정보 입력
	- `properties.networking_poe_ssl_certs`tile에서 생성한 cert key, private key 정보
	등 (( xxx ))로 감싸있는 manifest 파일을 설정한다.

- 설정한 yml 파일을  아래 폴더 구조로 덮어쓴다.
		
		└── dev-1
		    ├── config
		    │   ├── cf.yml

##### 2.2.3 configure-pas job을 실행 시킨다
	실행이 완료 되면 cf.yml 파일에 적용한 정보들이 PAS Tile의 Properties에 정상적으로 적용되었는지 확인 한다.
	
##### 2.2.4 apply-product-changes을 실행 시킨다.
	PAS가 정상적으로 설치 완료되면 Ops manager에서 Status를 확인한다.

### 3.  PAS Pipeline Issue

	1. Error could not execute "staged-config": cannot retrieve credentials for product 'cf': deploy the product and retry
	# PAS 초기 설정 시 PAS Pileline staged-pas-config에서 PAS tile에 적용 된 (( key )) 파일을 뽑아오지 못해 에러가 발생 해결 방법: install-products.yml Pipeline의 staged-pas-config job 아래SUBSTITUTE_CREDENTIALS_WITH_PLACEHOLDERS 설정 값을 true로 변경하여 job을 실행한다.

	2. Error {"errors":{"base":["Product cf-7dd8b2324d3bed84c77c cannot change its availability zones, as it is deployed."]}}
	# PAS Tile을 업로드하고 PAS를 설치 할 경우 AZ에 대한 정보는 변경 할 수 없음. 해결 방법은 bosh or Tile을 삭제 후 재 업로드하고 실행해야 함
	
	3. Error could not execute "configure-product": could not read file (vars/dev-leedh/vars/cf-vars.yml): open vars/dev-leedh/vars/cf-vars.yml: no such file or directory
	# Pipeline configure-product job에서 cf-vars.yml를 찾는 라인이 존재, cf-vars.yml이 존재하지 않는다면 해당 라인을 삭제한다.
	
	4. Error 'mysql_monitor/92215424-2e03-4fbd-9e6e-2f08efd421a0 (0)' is not running after update. Review logs for failed jobs: replication-canary
	# apply-product-changes 실행 시 아래와 같은 에러가 발생한다, 해결 방법은 cf.yml의 `ha_proxy.skip_cert_verify을 true로 변경 시킨다.
