#  Bosh Deploy Concept

- Bosh Director를 통해 VM(Software)을 배포에 대한 개념 & 방법에 대해 기술한 문서이다.
- Bosh Director를 통해 VM의 Manifest를 설정/배포 및 배포 후 Errand 실행 관련 내용이다.
- Bosh CLI v2를 기준으로 작성하였다.

##  1. Bosh Deploy Core Concept
- Bosh를 통해 VM(Software)를 배포하기 위해서는 크게 아래와 같은 절차가 필요하다.
	- Cloud Config 업로드
	- Manifest File 작성
	- Stemcell 업로드
	- Release 업로드
	- Deploy 실행

### 1.1. Cloud Config 업로드
- AWS/vSpghere/Openstack/Azure 등 IaaS 특정 구성을 정의하는 Yaml 형식의 파일으로써 Bosh Director를 통해 배포하는 모든 VM에 대하여 적용되는 Configuration File이다.
- 작성 예시는 아래와 같다. (vSphere)
```
azs:
- name: z1
  cloud_properties:
    datacenters:
    - clusters: [z1: {}]
- name: z2
  cloud_properties:
    datacenters:
    - clusters: [z2: {}]

vm_types:
- name: default
  cloud_properties:
    cpu: 2
    ram: 1024
    disk: 3240
- name: large
  cloud_properties:
    cpu: 2
    ram: 4096
    disk: 30_240

disk_types:
- name: default
  disk_size: 3000
- name: large
  disk_size: 50_000

networks:
- name: default
  type: manual
  subnets:
  - range: 10.10.0.0/24
    gateway: 10.10.0.1
    az: z1
    dns: [8.8.8.8]
    cloud_properties:
      name: vm-net1
  - range: 10.10.1.0/24
    gateway: 10.10.1.1
    az: z1
    dns: [8.8.8.8]
    cloud_properties:
      name: vm-net2

compilation:
  workers: 5
  reuse_compilation_vms: true
  az: z1
  vm_type: large
  network: default
```
- Cloud Config를 Bosh Director에 업로드 하는 방법
```
$ bosh -e {ailas-env} upload-cloud-config {cloud-config-file}
```

### 1.2. Manifest 파일 작성
- Bosh Director를 통해 VM(Software)를 배포하기 위해서는 Software가 사용하는 Spec, Config Properties 등이 기술되어 있는 Manifest 파일이 필요하다.
- Deploy Manifest 파일은 배포에 필요한 컴포넌트 및 속성 정보를 YAML 파일 형식으로 구성이 되어 있으며 크게 Release 정보를 설정 하는 Block, Network 정보를 설정하는 Block, Resource Pools 스템셀과 CPI 정보를 설정하는 Block, Disk Pool 사이즈를 설정하는 Block, Job 정보를 설정하는 Bloc, Property를설정하는 Block 등이 존재한다.
- 작성 예시는 아래와 같다.(zookeeper)
```
---
name: zookeeper

releases:
- name: zookeeper
  version: 0.0.5
  url: https://bosh.io/d/github.com/cppforlife/zookeeper-release?v=0.0.5
  sha1: 65a07b7526f108b0863d76aada7fc29e2c9e2095

stemcells:
- alias: default
  os: ubuntu-trusty
  version: latest

update:
  canaries: 2
  max_in_flight: 1
  canary_watch_time: 5000-60000
  update_watch_time: 5000-60000

instance_groups:
- name: zookeeper
  azs: [z1, z2, z3]
  instances: 5
  jobs:
  - name: zookeeper
    release: zookeeper
    properties: {}
  vm_type: default # cloud config에 명시되어 있는 Ailas
  stemcell: default
  persistent_disk: 10240 
  networks:
  - name: default

- name: smoke-tests
  azs: [z1] 
  lifecycle: errand
  instances: 1
  jobs:
  - name: smoke-tests
    release: zookeeper
    properties: {}
  vm_type: default
  stemcell: default
  networks:
  - name: default
```

### 1.3. Stemcell 업로드
- Bosh Director를 통해 VM(Software)를 배포하기 위해서는 특정 IaaS에 맞는 Packing 된 OS Image가 반드시 필요하다.
- Bosh Director에 Stemcell을 업로드하는 방법은 아래와 같다.
```
$ bosh -e {ailas-env} upload-stemcell {stemcell-file | stemcell-downlaod-url}
```

- Bosh Director에 업로드한 Stemcell을 확인 하는 방법은 아래와 같다.
```
$ bosh -e {ailas-env} stemcells
Using environment '192.168.50.6' as client 'admin'

Name                                         Version  OS             CPI  CID
bosh-warden-boshlite-ubuntu-trusty-go_agent  3468.17* ubuntu-trusty  -    6c9c002e-bb46-4838-4b73-ff1afaa0aa21

(*) Currently deployed

1 stemcells

Succeeded
```
- Bosh Director에 업로드한 Stemcell을 Deploy Manifest에서 사용하는 방법은 아래와 같다.
```
stemcells:
- alias: default
  os: ubuntu-trusty
  version: 3468.17
```

### 1.4. Release 업로드
- Bosh Director를 통해 VM(Software)를 배포하기 위해서는 동작하는 VM에 관련한 Process 실행 파일, Process 실행을 위한 Config, Yml, Properties 파일 등이 존재하고 있는 Release를 반드시 업로드 하여야 한다.
- Bosh Director에 Release를 업로드하는 방법은 아래와 같다.
```
$ bosh -e {ailas-env} upload-release {release-file | release-download-url}
```

- Bosh Director에 업로드한 Release를 확인하는 방법은 아래와 같다.
```
$ bosh -e {alias-env} releases

Using environment '192.168.50.6' as client 'admin'

Name       Version            Commit Hash
dns        0+dev.1496791266*  65f3b30+
zookeeper  0.0.5*             b434447

(*) Currently deployed
(+) Uncommitted changes

3 releases

Succeeded
```
- Bosh Director에 업로드한 Release를 Deploy Manifest에서 사용하는 방법은 아래와 같다.
```
releases:
- name: zookeeper
  version: 0.0.5
```

### 1.5. Bosh Deploy
- Bosh Director에 실행 Stemcell/Release/Cloud Config가 업로드 되어 있고, Deploy Manifest 작성이 완료 되어 있으면 Bosh Director의 VM(Software) Deploy 준비가 완료 된 상태이며 Deploy Manifest 파일을 바탕으로 배포를 진행 할 수 있다.
- Bosh Director가 VM(Software)를 배포하는 방법은 아래와 같다.
```
$ bosh -e {ailas-env} -d {deployment-name} deploy {deploy-manifest-file}

Using environment '192.168.56.6' as '?'

Task 1133

08:41:15 | Preparing deployment: Preparing deployment (00:00:00)
08:41:15 | Preparing package compilation: Finding packages to compile (00:00:00)
08:41:15 | Creating missing vms: zookeeper/6b7a51c4-1aeb-4cea-a2da-fdac3044bdee (1) (00:00:10)
08:41:25 | Updating instance zookeeper: zookeeper/3f9980b4-d02f-4754-bb53-0d1458e447ac (0) (canary) (00:00:27)
08:41:52 | Updating instance zookeeper: zookeeper/b8b577e7-d745-4d06-b2b7-c7cdeb46c78f (4) (canary) (00:00:25)
08:42:17 | Updating instance zookeeper: zookeeper/5a901538-be10-4d53-a3e9-3e23d3e3a07a (3) (00:00:25)
08:42:42 | Updating instance zookeeper: zookeeper/c5a3f7e6-4311-43ac-8500-a2337ca3e8a7 (2) (00:00:26)
08:43:08 | Updating instance zookeeper: zookeeper/6b7a51c4-1aeb-4cea-a2da-fdac3044bdee (1) (00:00:39)

Started  Mon Jul 24 08:41:15 UTC 2017
Finished Mon Jul 24 08:43:47 UTC 2017
Duration 00:02:32

Task 1133 done

Succeeded
```

- Bosh Director을 통해 배포 한 VM(Software)의 형상을 확인 하는 방법은 아래와 같다.
```
$ bosh -e {ailas-env} -d {deplyment-name} instances
Using environment '192.168.56.6' as '?'

Deployment 'zookeeper'

Instance                                          Process State  AZ  IPs
smoke-tests/42e003c1-1c05-453e-a946-c2e77935cff0  -              z1  -
zookeeper/3f9980b4-d02f-4754-bb53-0d1458e447ac    running        z2  10.244.0.2
zookeeper/5a901538-be10-4d53-a3e9-3e23d3e3a07a    -              z1  10.244.0.3
zookeeper/6b7a51c4-1aeb-4cea-a2da-fdac3044bdee    running        z3  10.244.0.6
zookeeper/b8b577e7-d745-4d06-b2b7-c7cdeb46c78f    running        z2  10.244.0.4
zookeeper/c5a3f7e6-4311-43ac-8500-a2337ca3e8a7    -              z1  10.244.0.5

6 instances

Succeeded
```

##  2. Bosh Persistence Disk 사용
- Bosh Director를 통해 설치한 VM(Software)에서 Persistence Type의 Disk를 사용 할 경우가 발생 할 수 있다. VM을 Hard Stop/Terminate/Delete 하게 되면 Data의 초기화가 발생하는데 Persistence Type Disk를 사용하게 되면 Data가 보존 된다.
- Persistence Disk는 크게 아래와 같은 상황에서 Data & Disk를 보존 한다.
	- VM 배포 Release, Stemcell Version Upgrade를 통한 Deploy 할 경우
	- $ bosh cck를 사용하여 VM을 복구 할 경우
	- VM을 강제로 Recreate 하였을 경우, --hard option을 추가하여 stop/start 했을 경우
- VM의 삭제로 인한 Orphaned Persistence의 보존기간 5일이 지나면 Garbage Collect 된다.
- Persistence Disk가 Orphaned 상태로 변경하는 시기는 아래와 같다.
	- Deploy Manifest의 Instance Groups에서 Persistence Disk를 사용하지 않음으로 변경 할 경우
	- Persistence Disk의 크기를 변경 할 경우
	- Deploy Manifest에서 migrated_from 구성 없이 Instance id가 변경 될 경우
	- Persistence Disk를 사용하는 VM이 Scaled Down 됬을 경우
	- Instance가 삭제 되거나 AZ가 사라졌을 경우
	- Deployment가 전체 삭제 됬을 경우

### 2.1. Bosh Deploy에 Persistent Disk 사용 방법
#### 2.1.1. Deploy Manifest에 Persistent Disk 선언
- Deploy Manifest의 instance_groups job Block에 Persistence Disk K/V 선언
- Persistence Disk의 크기가 정수 일 경우 MB로 계산 된다.
```
instance_groups:
- name: redis
  jobs:
  - {name: redis, release: redis}
  instances: 1
  resource_pool: default
  persistent_disk: 1024
  networks:
  - name: default
```

#### 2.1.2. Deploy Manifest에 Persistent Disk Pool 선언
- Disk Pool Block에서 생성 한 Disk Type을 Deploy Manifest의 instance_groups job Block에 persistent_disk_pool 명으로 선언
```
disk_pools:
- name: my-fast-disk
  disk_size: 1_024
  cloud_properties: {type: gp2}

instance_groups:
- name: redis
  jobs:
  - {name: redis, release: redis}
  instances: 1
  resource_pool: default
  persistent_disk_pool: my-fast-disk
  networks:
  - name: default
```

### 2.2. Persistent Disk 접근 방법
- Bosh CPI Release는 Bosh Director을 통해 VM을 Deploy 시 Persistent Disk 생성 요청이 들어오면 VM의 /var/vcap/store/에 Mount 시킨다.
- /var/vcap/store 디렉토리에 ext4 rw,relatime,data=ordered 0 0(읽기/쓰기/dump가 가능한) 권한을 할당하고 저장된 모든 Data를 유지 한다. 


### 2.3. Orphaned Disk 사용 방법
- Deploy에 대한 변경이 잘 못이루어져 특정 Persistence Disk를 다시 사용 해야 할 경우 아래의 절차를 이용하여 Disk를 교체 한다.
	1. Orphaned 상태인 Disk를 목록을 확인 한다.
	```
	$ bosh -e {ailas-env} disk --orphaned 
	```
	2. 교체 할 VM의 Instance를 Stop 한다.
	```
	$ bosh -e {alias-env} -d {deployment-name} stop  {instance-id}
	```
	3. Orphaned Disk를 Instance에 Attach 한다.
	```
	$ bosh -e {alias-env} -d {deployment-name} attach-disk {instance-id} {disk-id}
	```
	4. Attach한 VM의  Instance를 Start 한다.
	```
	$ bosh -e {alias-env} -d {deployment-name} start {instance-id}
	```

### 2.4. Persistence Disk Snapshot
- 오류 & 주요 Update로 인하여 Persistence Disk를 이전 Version으로 Rollback하기 위해서 Bosh Director는 Snapshot 기능을 제공한다.
- Snapshot은 완전한 Disk 전체의 Backup이 아닌 원본 Disk가 삭제 될 경우 IaaS에서 처리를 해주지 않는다.
- Snapshot 기능을 사용하기 위해서는 Bosh Director 배포 Manifest 파일 중 아래의 형식을 추가 하고 재설치를 진행 한다.
- snapshot_schedule Properties를 추가하여 일정 시간 별 Snapshot을 자동으로 생성 할 수 있다.
 
```
properties:
  director:
    enable_snapshots: true
```

#### 2.4.1. Snapshot 사용 방법
- Snapshot 목록 확인
```
$ bosh -e {alias-env} snapshots
```
- Snapshot 생성
```
$ bosh -e {alias-env} take snapshot {job} {index}
```
- Snapshot 삭제
```
$ bosh -e {alias-env} delete snapshot {snapshot-cid}
$ bosh -e {alias-env} delete snapshots
```

## 3. Bosh Errands 사용
- Bosh Release의 Job 내부 Spec/template 파일에 /bin/run Script가 포함 될 경우 해당 Job은 Errands로 취급하고 Deploy후 Bosh 명령어를 통해 Errand를 실행 시킬 수 있다. 실행에 대한 결과는 Stdout, Stderr Log로 사용자에게 보여진다.

### 3.1. Bosh Errand 정의 방법
- Pivotal은 Errand를 사용하여 Pivotal Cloud Foundry의 배포가 끝나고 Apps Manager, Usage Service, App Autoscaler 등을 Application 형태로 배포한다.

- smoke-tests Relesae Job의 templates아래 run.sh 명시
```
---
name: smoke-tests

templates:
  run.sh: bin/run

consumes:
- name: conn
  type: zookeeper
  properties:
  - client_port

packages:
- smoke-tests

properties: {}
```
- run.sh 파일 작성
```
#!/bin/bash
set -e
<% conn = link('conn') %>
export ZOOKEEPER_SERVERS=<%= conn.instances.map { |i| "#{i.address}:#{conn.p('client_port')}" }.join(",") %>
/var/vcap/packages/smoke-tests/bin/tests
```

- Deploy Manifest의 Instance Groups에 해당 Job을 사용
```
- name: smoke-tests
  azs: [z1]
  lifecycle: errand # 해당 Line을 지정하게 Errand가 동작 될 때만 smoke-tests Instance가 생긴다.
  instances: 1
  jobs:
  - name: smoke-tests
    release: zookeeper
    properties: {}
  vm_type: default
  stemcell: default
  networks:
  - name: default
```

### 3.2. Bosh Errand 실행 방법
- Bosh Director를 통해 배포한 Errand 목록을 확인 한다.
```
$ bosh -e {alias-env} -d {deployment-name} errands

Using environment '192.168.56.6' as client 'admin'

Using deployment 'zookeeper'

Name
smoke-tests
status

2 errands

Succeeded
```
-  Errand를 실행 시킨다.
```
$ bosh -e {alias-env} -d {deployment-name}run-errand status

Using environment '192.168.56.6' as client 'admin'

Using deployment 'zookeeper'

Task 5609

Task 5609 | 01:31:57 | Preparing deployment: Preparing deployment (00:00:01)
Task 5609 | 01:31:58 | Running errand: zookeeper/0015b995-5ec3-4519-8c11-6521b0aa079d (3)
Task 5609 | 01:31:58 | Running errand: zookeeper/e31944b9-8bf5-4a42-8d6b-3402a85d24d8 (0)
Task 5609 | 01:31:58 | Running errand: zookeeper/3e977542-d53e-4630-bc40-72011f853cb5 (4)
Task 5609 | 01:31:58 | Running errand: zookeeper/671d5b1d-0310-4735-8f58-182fdad0e8bc (1)
Task 5609 | 01:31:58 | Running errand: zookeeper/d9e00366-8ab1-4ea2-bae3-14cd6bf562cd (2)
Task 5609 | 01:31:59 | Running errand: zookeeper/0015b995-5ec3-4519-8c11-6521b0aa079d (3) (00:00:01)
Task 5609 | 01:31:59 | Fetching logs for zookeeper/0015b995-5ec3-4519-8c11-6521b0aa079d (3): Finding and packing log files
Task 5609 | 01:31:59 | Running errand: zookeeper/e31944b9-8bf5-4a42-8d6b-3402a85d24d8 (0) (00:00:01)
Task 5609 | 01:31:59 | Fetching logs for zookeeper/e31944b9-8bf5-4a42-8d6b-3402a85d24d8 (0): Finding and packing log files
Task 5609 | 01:31:59 | Running errand: zookeeper/d9e00366-8ab1-4ea2-bae3-14cd6bf562cd (2) (00:00:01)
Task 5609 | 01:31:59 | Fetching logs for zookeeper/d9e00366-8ab1-4ea2-bae3-14cd6bf562cd (2): Finding and packing log files
Task 5609 | 01:32:00 | Fetching logs for zookeeper/0015b995-5ec3-4519-8c11-6521b0aa079d (3): Finding and packing log files (00:00:01)
Task 5609 | 01:32:00 | Running errand: zookeeper/3e977542-d53e-4630-bc40-72011f853cb5 (4) (00:00:02)
Task 5609 | 01:32:00 | Running errand: zookeeper/671d5b1d-0310-4735-8f58-182fdad0e8bc (1) (00:00:02)
Task 5609 | 01:32:00 | Fetching logs for zookeeper/e31944b9-8bf5-4a42-8d6b-3402a85d24d8 (0): Finding and packing log files (00:00:01)
Task 5609 | 01:32:00 | Fetching logs for zookeeper/d9e00366-8ab1-4ea2-bae3-14cd6bf562cd (2): Finding and packing log files (00:00:01)
Task 5609 | 01:32:00 | Fetching logs for zookeeper/671d5b1d-0310-4735-8f58-182fdad0e8bc (1): Finding and packing log files
Task 5609 | 01:32:00 | Fetching logs for zookeeper/3e977542-d53e-4630-bc40-72011f853cb5 (4): Finding and packing log files
Task 5609 | 01:32:01 | Fetching logs for zookeeper/671d5b1d-0310-4735-8f58-182fdad0e8bc (1): Finding and packing log files (00:00:01)
Task 5609 | 01:32:01 | Fetching logs for zookeeper/3e977542-d53e-4630-bc40-72011f853cb5 (4): Finding and packing log files (00:00:01)

Task 5609 Started  Mon Sep 18 01:31:57 UTC 2017
Task 5609 Finished Mon Sep 18 01:32:01 UTC 2017
Task 5609 Duration 00:00:04
Task 5609 done

Instance   zookeeper/0015b995-5ec3-4519-8c11-6521b0aa079d
Exit Code  0
Stdout     Mode: leader
Stderr     ZooKeeper JMX enabled by default
           Using config: /var/vcap/jobs/zookeeper/config/zoo.cfg

Instance   zookeeper/3e977542-d53e-4630-bc40-72011f853cb5
Exit Code  0
Stdout     Mode: follower
Stderr     ZooKeeper JMX enabled by default
           Using config: /var/vcap/jobs/zookeeper/config/zoo.cfg

Instance   zookeeper/671d5b1d-0310-4735-8f58-182fdad0e8bc
Exit Code  0
Stdout     Mode: follower
Stderr     ZooKeeper JMX enabled by default
           Using config: /var/vcap/jobs/zookeeper/config/zoo.cfg

Instance   zookeeper/d9e00366-8ab1-4ea2-bae3-14cd6bf562cd
Exit Code  0
Stdout     Mode: follower
Stderr     ZooKeeper JMX enabled by default
           Using config: /var/vcap/jobs/zookeeper/config/zoo.cfg

Instance   zookeeper/e31944b9-8bf5-4a42-8d6b-3402a85d24d8
Exit Code  0
Stdout     Mode: follower
Stderr     ZooKeeper JMX enabled by default
           Using config: /var/vcap/jobs/zookeeper/config/zoo.cfg

5 errand(s)

Succeeded
```


