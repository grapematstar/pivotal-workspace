
# Bosh Backup And Restore Pivotal Cloud Foundry
 - 전제 조건
	- Ops Manager는 정상 동작해야 한다.
	- 외부 Database를 사용 할 경우 BBR이 가능한지 확인 해야 한다.
	- BOSH Director가 정상 동작 해야 한다.
 
## 1. Bosh Backup And Restore가 실행 될 JumpBox 구성
- Bosh Director를 통해 Jumpbox 환경이 구성이 되어 있으면 생략 한다.

[jumpbox 구성]](https://docs.pivotal.io/pivotalcf/2-5/customizing/backup-restore/backup-pcf-bbr.html)

### 1.1. JumpBox Config 확인

- jumpbox는  backup을 위한 충분한 공간이 있어야하고 Private IP로 Director VM에 연결하기 때문에 Director와 동일한 IP 대역에 존재해야 한다.
- BBR을 실행하기 위해 Director Access Port(25555, 8443, 22)를 허용 해줘야한다.

### 1.2.  BBR CLI Install
```
# jump box에 bbr cli를 설치 한다.
# bbr cli 설치 주소 https://github.com/cloudfoundry-incubator/bosh-backup-and-restore/releases
$ wget https://github.com/cloudfoundry-incubator/bosh-backup-and-restore/releases/download/v1.5.1/bbr-1.5.1-linux-amd64

$ chmod +x bbr-1.5.*

$ mv bbr-1.5.* /usr/local/bin/
```

## 2.  Backup Pivotal Cloud Foundry
- Backup 전제조건
	- Pivotal Cloud Foundry Backup 시 정상 동작하는 Pivotal Cloud Foundry를 Backup 해야 한다. 
	- Pivotal Cloud Foundry은 플랫폼 규모에 따라 시간이 오래 걸릴 수 있음으로 반드시 nohub또는 screen, tmux을 통해 Session을 유지 시켜 준다.
	
### 2.1. Backup 실행 과정

- Bosh Pivotal Cloud Foundry Backup 대상과 관련 한 Component(Cloud Controller, Cloud Controller Clock, UAA, Credhub) Job에 아래와 같은 파일 구조가 존재 한다.
```
total 24
drwxr-x--- 2 root vcap 4096 May  9 02:22 ./
drwxr-x--- 3 root vcap 4096 May  9 02:22 ../
-rwxr-x--- 1 root vcap  604 May  9 02:22 post-backup-unlock*
-rwxr-x--- 1 root vcap   13 May  9 02:22 pre-backup-lock*
```
1. Pre-backup lock: Backup 전 Pre-backup Script를 실행하여 backup이 전체 Deployment에서 일관되도록 Job의 실행을 중지/읽기만 가능하는 Lock을 실행 한다. 
2. Backup: 주요 Backup 대상 Component에 대한 backup script를 실행한다.
3. Post-backup unlock: Backup 종료 후 Pre-backup lock Script 실행 후 Lock을 해제 한다.

- Bosh Pivotal Cloud Foundry Backup workflow 시나리오

[Bosh PCF Backup workflow][https://docs.cloudfoundry.org/bbr/bbr-devguide.html#pre-backup-lock](https://docs.cloudfoundry.org/bbr/bbr-devguide.html#pre-backup-lock)

### 2.2.  BBR CLI를 실행하여 Pivotal Cloud Foundry를 Backup
```
# bosh pivotal cloud foundry의 backup-restore를 clean 한다.
$ bbr deployment \
--target "${BOSH_ENVIRONMENT}" \
--username ${BOSH_CLIENT} \
--password ${BOSH_CLIENT_SECRET} \
--deployment "${DEPLOYMENT_NAME}" \
--ca-cert "${BOSH_CA_CERT}" \
backup-cleanup

# bosh pivotal cloud foundry를 backup이 가능한지 pre-backup-check을 실행한다.
$ bbr deployment \
--target "${BOSH_ENVIRONMENT}" \
--username ${BOSH_CLIENT} \
--password ${BOSH_CLIENT_SECRET} \
--deployment "${DEPLOYMENT_NAME}" \
--ca-cert "${BOSH_CA_CERT}" \
pre-backup-check

# bosh pivotal cloud foundry backup을 실행 한다.
$ bbr deployment \
--target "${BOSH_ENVIRONMENT}" \
--username ${BOSH_CLIENT} \
--password ${BOSH_CLIENT_SECRET} \
--deployment "${DEPLOYMENT_NAME}" \
--ca-cert "${BOSH_CA_CERT}" \
--artifact-path ./generated-backup \
backup

# bosh pivotal cloud foundry의 backup-restore를 다시 clean 한다.
$ bbr deployment \
--target "${BOSH_ENVIRONMENT}" \
--username ${BOSH_CLIENT \
--password ${BOSH_CLIENT_SECRET \
--deployment "${DEPLOYMENT_NAME" \
--ca-cert "${BOSH_CA_CERT}" \
backup-cleanup

# bosh pivotal cloud foundry backup이 완료되면 artifact-path 디렉토리에 아래와 같은 파일 구조가 생기는 것을 확인 한다.

total 76920436
drwx------ 2 ubuntu ubuntu        4096 May 24 19:45 ./
drwxrwxr-x 5 ubuntu ubuntu        4096 May 29 15:17 ../
-rw-r--r-- 1 ubuntu ubuntu       10240 May 24 19:45 backup_restore-0-azure-blobstore-backup-restorer.tar
-rw-r--r-- 1 ubuntu ubuntu       20480 May 24 19:45 backup_restore-0-backup-restore-notifications.tar
-rw-r--r-- 1 ubuntu ubuntu       20480 May 24 19:31 backup_restore-0-backup-restore-pcf-autoscaling.tar
-rw-r--r-- 1 ubuntu ubuntu     1249280 May 24 19:45 backup_restore-0-bbr-cfnetworkingdb.tar
-rw-r--r-- 1 ubuntu ubuntu   204410880 May 24 19:45 backup_restore-0-bbr-cloudcontrollerdb.tar
-rw-r--r-- 1 ubuntu ubuntu       30720 May 24 19:31 backup_restore-0-bbr-credhubdb.tar
-rw-r--r-- 1 ubuntu ubuntu       10240 May 24 19:45 backup_restore-0-bbr-routingdb.tar
-rw-r--r-- 1 ubuntu ubuntu      317440 May 24 19:45 backup_restore-0-bbr-uaadb.tar
-rw-r--r-- 1 ubuntu ubuntu   105984000 May 24 19:45 backup_restore-0-bbr-usage-servicedb.tar
-rw-r--r-- 1 ubuntu ubuntu       10240 May 24 19:45 backup_restore-0-nfsbroker-bbr.tar
-rw-r--r-- 1 ubuntu ubuntu       10240 May 24 19:31 backup_restore-0-s3-unversioned-blobstore-backup-restorer.tar
-rw-r--r-- 1 ubuntu ubuntu       10240 May 24 19:45 backup_restore-0-s3-versioned-blobstore-backup-restorer.tar
-rw-r--r-- 1 ubuntu ubuntu      164153 May 24 19:31 metadata
-rw-r--r-- 1 ubuntu ubuntu       10240 May 24 19:31 nfs_server-0-blobstore-backup.tar
-rw-r--r-- 1 ubuntu ubuntu 78454231040 May 24 19:45 nfs_server-0-blobstore.tar
```

## 3.  Restore Pivotal Cloud Foundry
- Restore 전제조건
	- Pivotal Cloud Foundry Restore 시 Backup 파일이 존재 해야 한다. 

### 3.1. Restore 실행 전 준비 사항
- Pivotal Cloud Foundry VM 중 State가 Running이 아닌 Failing, Unresponsive, Missing, Unbound 등 Error VM에 대한 Reference VM과 Disk를 삭제 한다.
```
# bosh cck 명령어를 실행하여 disk/vm 삭제
$ bosh -e ${ALIAS-ENV} \
-d {PCF-DEPLOYMENT-NAME} -n cck \
--resolution delete_disk_reference \
--resolution delete_vm_reference
```
- Pivotal Cloud Foundry Redeploy 실행
	- Ops Manager Dashboard의 Pivotal Cloud Foundry Tile에서 Resource Config MySQL Server의 Instance 수를 1개로 수정 한다.
	- Ops Manager Dashboard에서 Pivotal Cloud Foundry Apply Change를 실행 한다.
		- Apply Change 중 Error가 발생 할 경우 Ops Manager Dashboard Bosh Director Tile의 Director Config Recreate All VMs를 선택하고 다시 Apply Change를 실행 한다.

### 3.2. Restore 실행 과정

- Bosh Pivotal Cloud Foundry Backup 대상과 관련 한 Component(Cloud Controller, Cloud Controller Clock,UAA, Credhub) Job에 아래와 같은 파일 구조가 존재 한다.
```
total 24
drwxr-x--- 2 root vcap 4096 May  9 02:22 ./
drwxr-x--- 3 root vcap 4096 May  9 02:22 ../
-rwxr-x--- 1 root vcap  620 May  9 02:22 post-restore-unlock*
-rwxr-x--- 1 root vcap   13 May  9 02:22 pre-restore-lock*
```

1. Pre-restore lock: Restore 되는중 대상 Component를 변경할 수있는 Process를 중지하는Lock Script를 실행 한다.
2. Restore: 밀어넣은 파일을 기준으로 Restore 대상 Component의 Restore Script를 실행 한다. ex) db인 경우 mysql dump를 실행
3. Post-restore unlock: Pre-restore lock을 통해 중지한 Process를 재시작 한다.

- Bosh Pivotal Cloud Foundry Restore workflow 시나리오

[Bosh PCF Restore workflow][https://docs.cloudfoundry.org/bbr/bbr-devguide.html#pre-backup-lock](https://docs.cloudfoundry.org/bbr/bbr-devguide.html#pre-backup-lock)

### 3.3.  BBR CLI를 실행하여 Pivotal Cloud Foundry를 Restore
```
# bosh Pivotal Cloud Foundry restore 명령을 실행 한다.
$  bbr deployment \
--target ${BOSH-DIRECTOR-IP} \
--username ${BOSH-CLIENT} \
--password ${BOSH-PASSWORD} \
--deployment ${DEPLOYMENT-NAME} \
--ca-cert ${PATH-TO-BOSH-SERVER-CERTIFICATE} \
restore \
--artifact-path PATH-TO-PAS-BACKUP

# bosh Pivotal Cloud Foundry restore clean up 명령을 실행 한다.
bbr deployment \
--target BOSH-DIRECTOR-IP \
--username BOSH-CLIENT \
--password BOSH-PASSWORD \
--deployment DEPLOYMENT-NAME \
--ca-cert PATH-TO-BOSH-SERVER-CERTIFICATE \
restore-cleanup

# 정상적으로 restore가 끝나면 apply change 버튼을 클릭한다.

# bosh cck 명령어를 실행하여 Pivotal Cloud Foundry VM에 이상이 없는지 확인한다.
$ bosh -e ${ALIAS-ENV} \
-d {PCF-DEPLOYMENT-NAME} -n cck \

# apps manage ui에서 Application이 정상적으로 보여지는지 확인 한다.
```
