
# Bosh Backup And Restore Director
 - 전제 조건
	- Ops Manager는 정상 동작해야 한다.
	- 외부 Database를 사용 할 경우 BBR이 가능한지 확인 해야 한다.
 
## 1. Bosh Backup And Restore가 실행 될 JumpBox 구성

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

## 2.  Back Director
- Backup 전제조건
	- Director Backup 시 정상 동작하는 Director를 Backup 해야 한다. 
### 2.1. Backup 실행 과정

- Bosh Director Backup 대상과 관련 한 Component(UAA, Credhub) Job에 아래와 같은 파일 구조가 존재 한다.
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

- Bosh Director Backup workflow 시나리오

[Bosh Director Backup workflow][https://docs.cloudfoundry.org/bbr/bbr-devguide.html#pre-backup-lock](https://docs.cloudfoundry.org/bbr/bbr-devguide.html#pre-backup-lock)

### 2.2.  BBR CLI를 실행하여 Director를 Backup
```
# director backup을 하기전 backup disk 용량을 최소화와 빠른 실행을 위해 director를 clean up 한다.

# bosh director가 사용하지 않는 stemcell/release/metadata/disk를 clean 한다.
$ bosh -e ${alias-env} clean-up --all

# bosh director의 backup-restore를 clean 한다.
$ bbr director \
--private-key-path ${BOSH_PRIVATE_KEY_FILE} \
--username bbr \
--host ${BOSH_DIRECTOR_IP} \
backup-cleanup

# bosh director가 backup이 가능한지 pre-backup-check을 실행한다.
$ bbr director --host ${BOSH_DIRECTOR_IP} \
--username bbr  \
--private-key-path ${BOSH_PRIVATE_KEY_FILE} \
pre-backup-check

# bosh director backup을 실행 한다.
$ bbr director \
--private-key-path ${BOSH_PRIVATE_KEY_FILE} \
--username bbr \
--host ${BOSH_DIRECTOR_IP}\
--artifact-path ./generated-backup \
backup

# bosh director의 backup-restore를 다시 clean 한다.
$ bbr director \
--private-key-path ${BOSH_PRIVATE_KEY_FILE} \
--username bbr \
--host ${BOSH_DIRECTOR_IP} \
backup-cleanup

# bosh director backup이 완료되면 artifact-path 디렉토리에 아래와 같은 파일 구조가 생기는 것을 확인 한다.

total 7219824
drwxr-xr-x 2 ubuntu ubuntu       4096 Jun  5 11:12 ./
drwxrwxr-x 3 ubuntu ubuntu       4096 May 30 15:56 ../
-rw-rw-r-- 1 ubuntu ubuntu      30720 May 30 15:49 bosh-0-bbr-credhubdb.tar
-rw-rw-r-- 1 ubuntu ubuntu 7391959040 May 30 15:49 bosh-0-blobstore.tar
-rw-rw-r-- 1 ubuntu ubuntu    1054720 May 30 15:49 bosh-0-director.tar
-rw-rw-r-- 1 ubuntu ubuntu      35723 May 30 15:51 metadata
```

## 3.  Restore Director
- Restore 전제조건
	- Director Restore 시 Backup 파일이 존재 해야 한다. 
	- Director Restore 명령어 실행 전 Ops Manger에 Restore 대상 Director(껍데기)가 존재 해야 한다.
	
### 3.1. Restore 실행 과정

- Bosh Director Backup 대상과 관련 한 Component(UAA, Credhub) Job에 아래와 같은 파일 구조가 존재 한다.
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

- Bosh Director Restore workflow 시나리오

[Bosh Director Restore workflow][https://docs.cloudfoundry.org/bbr/bbr-devguide.html#pre-backup-lock](https://docs.cloudfoundry.org/bbr/bbr-devguide.html#pre-backup-lock)

### 3.2.  BBR CLI를 실행하여 Director를 Restore
```
# bosh director restore 명령을 실행 한다.
$ bbr director \
--private-key-path ${BOSH_PRIVATE_KEY_FILE} \
--username bbr \
--host ${BOSH_DIRECTOR_IP} \
restore \
--artifact-path ${PATH-TO-DIRECTOR-BACKUP}

# bosh director restore clean up 명령을 실행 한다.
bbr director \
--private-key-path PRIVATE-KEY-FILE \
--username bbr \
--host HOST \
restore-cleanup

# 정상적으로 restore가 끝나면 apply change 버튼을 클릭한다.
# bosh vms 명령어를 통해 bosh 위에 설치한 Tile들이 정상적으로 붙어 있는지 확인 한다.
$ bosh vms
```
