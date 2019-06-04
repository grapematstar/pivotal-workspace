## Bosh Director BBR

 - 전제 조건
	- Ops Manager는 정상 동작해야 한다.
	- 외부 Database를 사용 할 경우 BBR이 가능한지 확인 해야 한다.
 
### 1. Bosh Backup And Restore가 실행 될 JumpBox 구성

#### 1.1. JumpBox Config 확인

- jumpbox는  backup을 위한 충분한 공간이 있어야하고 Private IP로 Director VM에 연결하기 때문에 Director와 동일한 IP 대역에 존재해야 한다.
- BBR을 실행하기 위해 Director Access Port(25555, 8443, 22)를 허용 해줘야한다.

#### 1.2.  BBR CLI Install
```
# jump box에 bbr cli를 설치 한다.
# bbr cli 설치 주소 https://github.com/cloudfoundry-incubator/bosh-backup-and-restore/releases
$ wget https://github.com/cloudfoundry-incubator/bosh-backup-and-restore/releases/download/v1.5.1/bbr-1.5.1-linux-amd64

$ chmod +x bbr-1.5.*

$ mv bbr-1.5.* /usr/local/bin/
```

### 2.  BBR Director
#### 2.1.  BBR CLI를 실행하여 Director를 Back Up 받는다
```
# director back을 하기전 
bbr director \
--private-key-path ${BOSH_PRIVATE_KEY_FILE} \
--username bbr \
--host ${BOSH_DIRECTOR_IP} \
backup-cleanup
```
