## Healthwatch Pipeline Automation

### 1. HealthWatch 참고 URL
- https://docs.pivotal.io/pivotalcf/2-4/monitoring/kpi.htm

#### 1.1. healthwatch 파이프라인 수정 & 실행
##### 1.1.1. concourse web 화면에서 upload-and-stage-healthwatch job을 실행시킨다.
	- upload-and-stage-healthwatch job 실행은 p-healthwatch의 타일을 opsmanager에 업로드 시킨다.
##### 1.1.2. opsmanager에서 p-healthwatch의 Properties를 기술한다.
	# install-products.yml 파일의 staged-healthwatch-config job을 수정한다.
	- config 파일을 git hub에 commit/push 하는 영역이 누락되어 아래 부분을 추가 한다.
	
	on_success: &make-state-commit
	 do:
	 - task: make-commit
	 image: platform-automation-image
	 file: platform-automation-tasks/tasks/make-git-commit.yml
	 input_mapping:
	 repository: configuration
	 file-source: generated-config # from staged-config.yml
	 output_mapping:
	 repository-commit: configuration
	 params:
	 FILE_SOURCE_PATH: p-healthwatch.yml # gen at stated-config.yml
	 FILE_DESTINATION_PATH: ((foundation))/generated-config/healwatch.yml # you can name it.
	 GIT_AUTHOR_EMAIL: "pcf-pipeline-bot@example.com"
	 GIT_AUTHOR_NAME: "Platform  Automation  Bot"
	 COMMIT_MESSAGE: 'healwatch.yml'
	 - put: configuration
	 params:
	 repository: configuration
	 merge: true
	 
##### 1.1.3.수정이 완료 되면 concourse web 화면에서 staged-healthwatch-config job을 실행 시킨다

	실행 결과는 아래의 디렉토리 구조에 config 파일이 생성 된다.
 	    platform-automation-configuration-template
	    └── dev-1
	        ├── generated-config
		        ├── healwatch.yml

	아래 config yml 파일을 아래의 디렉토리에 옮긴다.
		└── dev-1
	    ├── config
	    │   ├── healwatch.yml

##### 1.1.4. p-healthwatch stemcell을 업로드 시킨다.
	concourse web 화면에서 upload-stemcells job을 실행하여  p-healthwatch 관련 스템셀 이미지를 bosh director에 업로드 시킨다.
	업로드 한 결과는 opsmanager # Stemcell Library 화면에서 확인한다.

##### 1.1.5. p-healthwatch 설치
	concourse web 화면에서 apply-change job을 실행시킨다.
	기존의 director와 pcf를 반영하지 않기 위해서는 task의 om cli 실행 부분을 변경하여 git에 push하여 놓는다.
	om 명령어는 opsmanager shell에 접속하여 om help로 확인한다.

##### 1.1.6. p-healthwatch 설치가 완료 되면 Web 화면으로 확인한다.
	p-healthwatch web화면은 application 형태로 배포되며 system org의 p-healthwatch space에 적용 되어 있다.
