## setup timezone pipeline | Job 생성

### 1. set time zone job 생성

#### 1.1 install-products-params.yml
##### 1.1.1. 다음 단계인 install-products.yml 를 실행 시키며 사용하기 위해 git 정보와 minio 정보를 업데이트 정의해준다.
	s3:
	 endpoint: https:///s3.pcfdemo.net
	 access_key_id: ((s3_access_key_id))
	 secret_access_key: ((s3_secret_access_key))
	 region_name: ""
	 buckets:
	 platform_automation: platform-automation
	 foundation: dev-1
	 pivnet_products: pivnet-products
	 installation: installation

	git:
	 platform_automation_tasks:
	 uri: pivotal@git.pcfdemo.net/platform/platform_automation_tasks.git
	 configuration:
	 uri: pivotal@git.pcfdemo.net:platform/platform-conf.git
	 variable:
	 uri: pivotal@git.pcfdemo.net:platform/platform-conf.git
	 user: 
	 email: ((git_user.email))
	 username: ((git_user.username))
	 private_key: ((git_private_key.private_key))

#### 1.2 install-products.yml 수정

##### 1.2.1. timezone 관련 bosh runtime 값을 적용시키기 위한 Task 파일의 github 생성 및 resource 추가
	 - name: platform-automation-custom-tasks-git
	   type: git
	   source:
	   private_key: ((git.private_key))
	   uri: ((git.platform_automation_custom_tasks.uri))
	   branch: master

##### 1.2.2. timezone-release가 존재하고 있는 minio resource 설정
	 - name: timezone-release-s3
	   type: s3
	   source:
	   endpoint: ((s3.endpoint))
	   access_key_id: ((s3.access_key_id))
	   secret_access_key: ((s3.secret_access_key))
	   region_name: ((s3.region_name))
	   bucket: ((s3.buckets.timezone))
	   versioned_file: timezone-1.0.tgz
	   skip_ssl_verification: true


##### 1.2.3. timezone job 생성
	 - name: setup-timezone
	 serial: true
	 plan:
	 - aggregate:
	 - get: platform-automation-image # 컨테이너 실행 이미지
	 params: {unpack: true}
	 - get: configuration
	 - get: platform-automation-custom-tasks-git
	 - get: platform-automation-tasks-git
	 - get: timezone-release-s3
	 - task: credhub-interpolate #temporary
	 <<: *credhub-interpolate
	 - task: timezone-task
	 image: platform-automation-image
	 file: platform-automation-custom-tasks/timezone-leedh.yml
	 input_mapping:
	 env: configuration-interpolated
	 platform-automation-custom-tasks: platform-automation-custom-tasks-git # 위 file을 실행 시키기 위해 input 값으로 넘겨 줘야 한다.
	 timezone: timezone-release-s3
	 config: configuration-interpolated
	 params: # input file에서 사용 할 공용 파라미터
	 ENV_FILE: ((foundation))/env/env.yml 
	 TIMEZONE_FILE: ((foundation))/config/timezone-runtime-config.yml

#### 1.3. platform-automation-custom-tasks git에 timezone-leedh.yml/timezone-leedh.sh 작성
##### 1.3.1. timezone-leedh.yml 작성
	---
	platform: linux
	inputs:
	- name: env # contains the env file with target OpsMan Information
	- name: platform-automation-custom-tasks
	- name: timezone
	- name: config
	params:
	 ENV_FILE: env.yml
	 TIMEZONE_FILE: timezone-runtime-config.yml
	run:
	 path: platform-automation-custom-tasks/timezone-leedh.sh # yaml에서 bash를 실행하면 라인을 맞추기 어렵기 때문에 따로 빼는것을 권고
	 
##### 1.3.2.  timezone-leedh .sh 작성

	#!/bin/bash
	set -x
	 om --env env/"${ENV_FILE}" bosh-env > bosh-env.sh # 컨테이너 이미지에 설치되어 있는 om 이미지를 통해 opsmanager에 지정한 bosh director의 정보를 가져온다.
	 source bosh-env.sh
	  
	 bosh env
	 bosh upload-release timezone/timezone-1.0.tgz
	 bosh releases 
	 bosh update-runtime-config --non-interactive config/"${TIMEZONE_FILE}"


#### 1.4. pipeline update & start
	# pipeline을 업데이트 시키고 concourse 화면에서 setup-timezone job을 실행 시킨다.
	$ fly -t leedh sp -p install-products-leedh -c install-products.yml -l install-opsman-params.yml
	
#### 1.5. timezone 적용 확인 방법
##### 1.5.1. release & config 확인
	# bosh가 설정 되어 있는 컨테이너 & Opsmanager에 접속
	$ bosh releases
	$ bosh config --name default --type runtime

##### 1.5.2. apply change
	# apply change 후에 $ bosh deployments를 통해 확인한다.
	# 적용한 vm에 접속하여 $ date 명령어를 통해 확인

#### 1.6. issue 확인
##### 1.6.1.  resource의 버전이나 경로가 맞지 않는 경우
	fly gp를 통해 resource 경로를 확인한다.
	그 경로가 올바른지 확인
	check resource 

##### 1.6.2. job 실행 후 task가 실행 될 때 경로 및 cli 에러

	# task가 실행하고 있는 container에 접속하여 에러를 확인한다.
	
	$ fly -t leedh containers | grepp {keyword}
	$ fly -t leedh i -j {pipeline_name}/{job_name}

	
