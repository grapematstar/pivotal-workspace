
#  Bosh Deploy Concept

- Bosh Director를 통해 VM(Software)을 배포에 대한 개념 & 방법에 대해 기술한 문서이다.
- Bosh Director를 통해 Update되는 VM의 실행 절차와 Bosh의 Credentials에 대한 정리 문서이다.
- Bosh CLI v2를 기준으로 작성하였다.

##  1. Bosh의 VM Update

### 1.1. 기본 Bosh Director의 VM Update 방식
- 기존 Bosh Director를 통해 Stemcell/Release의 배포 변경 및 Job의 Properties 변경에 따른 VM Update시 Bosh Director는 모든 Job를 중지하고 IaaS에서 VM을 삭제/ 신규 생성 후 Persistence Disk, VM, OS의 프로비져닝, Job를 전체 재시작하는 단계를 거치게 된다, 해당 Process는 수 분이 걸릴 수 있으며 작업 중에 Downtime이 발생하게 된다.
- 아래는 기존 Bosh Director의 VM Update 절차이다.

[VM_Update](https://bosh.io/docs/changing-deployment-vm-strategy/)

- 잠재적으로 기존 Bosh Director의 VM Update시 Downtime이 발생하는 단계는 아래와 같다.
	- All Job Stop
	- Disk Attach
	- All Job Start (Prestart, Start, PostStrat)


### 1.2. 설정을 추가하여 변경 한 Bosh Director의 VM Update 방식
- Bosh Release Version 267.2 이후 부터 Director는 create-swap-delete 기능을 사용하여 VM의 Update 시 Job이 중지되기전 새로운 VM을 생성하고 Persistence Disk, VM, OS의 프로비져닝이 선행으로 이루어져 Downtime이 최소화 된다.
- 아래는 create-swap-delete 기능을 활성화 하고 진행 되는 VM Update 절차 이다.

[VM_Update](https://bosh.io/docs/changing-deployment-vm-strategy/)

-  기존의 Update 대상 VM을 Orphaned 상태로 변경 후 5분 내에 자동으로 삭제 한다.

#### 1.2.1.  create-swap-delete의 사용 방법
- Bosh Update 대상의 Deployment의 Manifest Update Block에 아래의 값을 지정하여 사용 한다.
```
update:
  canaries: 4
  ...
  vm_strategy: create-swap-delete
```

### 1.3. create-swap-delete 사용 시 주의 사항
- 신규 Deploy시 새로 생성하는 VM의 IP가 Random 한 값으로 발생하여 Static IPs를 통하여 지정한 VM에 대하여는 적용 될 수 없다.
- create-swap-delete를 사용하게 되면 신규 Deploy시 IaaS의 사용량이 급상승하여 Deploy를 준비하기 위해 Account 에 할당한 Service Limit 또는 인프라의 용량 등을 확인해야 한다.


##  2. Bosh의 Credentials on Memory
- Bosh Credential 정보, Job의 Username/Password/Cert 정보 등을 Tmpfs라는 Properties를 정의하여 VM 내부의 Disk가 아닌 환경 변수로 사용 하도록 할 수 있다.
	-  Tmpfs 사용은 Bosh Release Version 268.5.0, Stemcell Version 250 이상 부터 사용 가능 하다.

###  2.1. Bosh Director에서 Tmpfs Credentials 사용 방법
- Bosh에서 각 각의 Component들의 내부 통신 할 때 NAT Message Bus가 해당 Memory(환경 변수)에 있는 Credentials를 사용하도록 한다.
``` 
- name: bosh
  ...
  properties:
    ...
    director:
      ...
      enable_nats_delivered_templates: true
``` 

#### 2.1.1 Bosh Director에서 Tmpfs Credentials 주의 사항
- Credentials과 Job의 생성 정보가 환경 변수에 들어 있어 재부팅이 어려우며 구성을 복구하여 재생성해야 할 수 있다.
- Tmpfs의 Default Memory는 100MB로 해당 Memory 크기를 늘려야 할 수 있다.

###  2.2. 전체 Instance Group에서 Tmpfs Credentials 사용 방법
- use_tmpfs_config flag를 사용하여 전체 Deployment에 반영 한다.
```
features:
  use_tmpfs_config: true
```

###  2.3. 특정 Job에서 Tmpfs Credentials 사용 방법
- env 속성을 만들어 전역 설정을 변경하여 특정 Job에 반영 한다.
```
instance_groups:
- name: zookeeper
  ...
  env:
    bosh:
      job_dir:
        tmpfs: true
        tmpfs_size: 128m
```
