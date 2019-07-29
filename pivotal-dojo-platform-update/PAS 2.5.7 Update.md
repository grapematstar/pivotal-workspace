
#  Pivotal Cloud Foundry v2.5.7
- 진행한 Ops Manager와 Pivotal Cloud Foundry 현재 Version은 아래와 같다.
	- Ops Manager v2.5
	- Pivotal Cloud Foundry v2.4.6

## 1. Pivotal Cloud Foundry Release Note 2.5
- [Release Note](https://docs.pivotal.io/pivotalcf/2-5/pcf-release-notes/runtime-rn.html)
- [Breaking Change](https://docs.pivotal.io/pivotalcf/2-5/pcf-release-notes/breaking-changes.html)

### 1.1. Pivotal Cloud Foundry 주요 변경 사항

#### 1.1.1. PAS Only Supports cflinuxfs3
- PAS v2.5 부터 cflinuxfs2는 지원하지 않음
- cf audit-stack Plugin을 통해 cflinuxfs2 Application을 검사하고 cflinuxfs3으로 Restage 해야 한다.

#### 1.1.2. Disable Network Policy Enforcement Between Apps
-  Application 사이의 Slik Network 정책을 비 활성화 할 수 있다. Silk Network를 비활성화 하면 기본적으로 모든 Application 사이의 통신이 열린다. Silk Network는 Container Interface로 Container에 가상 VXLAN Network 주소를 할당하여 관리한다.

#### 1.1.3. Service Mesh Routing Plane (Beta)
- Application에 대한 Http/TCP Routing이 아닌 Service Mesh Routing을 Beta 지원한다.

#### 1.1.4. Weighted Routing
- Service Mesh를 포함되어 배포한 PAS는 $ cf curl을 통하여 Application의 Routing 가중치를 지정 할 수 있다.

#### 1.1.5. Diego BBS Routing의 안정성 향상
- 불안정한 Cell에서 실행되는 Application에 대한 Routing 기능을 향상
- Cell의 Heart Beat를 놓쳤을 경우 Application이 교체 되었지만 v2.5 부터는 Application에 대한 Routing이 유지 된다.

#### 1.1.6. 다중 Port의 Application 지원
-  기존 80/443이나 TCP Roting을 사용하지 않고 $cf curl을 통해 사용자 지정 Port 번호나 다중의 Port를 사용 할 수 있도록 Application을 지원

#### 1.1.7. Configure Networking Policies Across Spaces Using the cf CLI
- $ cf CLI를 통해 PAS의 Space에 특정 Network 정책 구성이 가능

#### 1.1.8. Apps Manager에서 비동기 Service Bind를 지원
- Apps Manager를 통해 Service Bind 할 때 비동기식 Service Bind를 지원하여 기존의 Service Bind시 발생하던 Timeout 발생을 피할 수 있음.

#### 1.1.9. Specify Metadata for Apps, Orgs, and Spaces
- 주석/Label 형태의 Metadata를 $ cf curl을 통하여 App/Org/Space에 지정 할 수 있다.

#### 1.1.10. Generate Garden Component Logs with Binary File
- /var/vcap/packages/garden/bin/dontpanic를 통하여 Garden Container의 구성 요소의 Log를 추출 할 수 있다.

#### 1.1.11. mysql-restore and mysql-backup Jobs Are Removed
- PAS가 내부적으로 사용 되던 mysql-restore와 mysql-backup Job이 삭제 됨

#### 1.1.12. Cloud Controller Retrieves Container Metrics from Log Cache
- PAS v.2.5에서는 Cloud Controller가 Traffic Controller와 Log Cache에서 직접 Container Metric을 검색하는 것을 지원

#### 1.1.13. Removed Logging System Property
- Syslog Drain Buffer Size에 대한 입력 창이 제거 됨

#### 1.1.14. 


### 1.2. Pivotal Cloud Foundry 주요 변경 사항

## 2. Pivotal Cloud Foundry Update 절차

### 2.1. Pivotal Cloud Foundry Tile Download
- Pivotal Network에서 신규 Pivotal Cloud Foundry Tile v2.5.7를 Download 한다.

### 2.2. Pivotal Cloud Foundry Tile Configuration
- Pivotal Cloud Foundry v2.5.6 이상, Application의 간혈적인 502 Bad Gateway가 발생 할 경우 아래 Config 설정
	- Pivotal Cloud Foundry Tile의 Application Containers Config에서 "Prune Routes on TTL Expiry for TLS Backends"를 활성화 해야 한다.

### 2.3. Pivota Cloud Foundry Apply Change
- Pivotal Cloud Foundry를 Apply Change 한다.

## 3. Issues

### 3.1. PAS 배포 중 Stemcell Upload Error 발생
- PAS 배포 중 vSphere Cluster의 주소를 찾지 못하는 에러가 발생
	- 해결 방법: Bosh Director에 접속하여  Hosts 파일에 해당 Cluster의 host 주소를 설정
	
### 3.2. PAS 배포 중 Release Upload Error 발생
- PAS Relesae Upload 중 알수없는 Error 발생
```
Task 39496 | 09:17:42 | Creating new compiled packages: java-buildpack-cflinuxfs3/638df085bd48cc15fd1c94206f580623a7932e3f for ubuntu-xenial/250.38 (00:00:10)  
                    L Error: Failed to upload blob, code 1, output: 'Error running app - Putting dav blob a1e485b2-b873-4414-bb2c-7676e034f61b: Wrong response code: 500; body: &lt;html&gt;  
&lt;head&gt;&lt;title&gt;500 Internal Server Error&lt;/title&gt;&lt;/head&gt;  
&lt;body bgcolor="white"&gt;  
&lt;center&gt;&lt;h1&gt;500 Internal Server Error&lt;/h1&gt;&lt;/center&gt;  
&lt;hr&gt;&lt;center&gt;nginx&lt;/center&gt;  
&lt;/body&gt;  
&lt;/html&gt;
```

- Bosh 명령어가 통하지 않는것을 확인 후 Bosh Director VM에 접속하여 Job의 Process 확인
```
$ monit summary
Process 'system-metrics-server'     running
Process 'nats'                      running
Process 'postgres'                  running
Process 'director'                  Does not exist
Process 'worker_1'                  initializing
Process 'worker_2'                  initializing
Process 'worker_3'                  initializing
Process 'worker_4'                  initializing
Process 'worker_5'                  initializing
Process 'director_scheduler'        Does not exist
Process 'director_sync_dns'         Does not exist
Process 'director_nginx'            running
Process 'health_monitor'            running
Process 'uaa'                       running
Process 'credhub'                   not monitored
Process 'blobstore_nginx'           running
Process 'blackbox'                  running
System 'system_localhost'           running
```
- Bosh Director의 Log를 확인 결과 Postgres에 이상 감지
```
WARNING:  could not create relation-cache initialization file "global/pg_internal.init.1379": No space left on device
DETAIL:  Continuing anyway, but there's something wrong.
172.28.86.91 - - [25/Jul/2019:09:51:02 +0000] "POST /packages/matches HTTP/1.0" 500 - 0.0089
WARNING:  terminating connection because of crash of another server process
DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
HINT:  In a moment you should be able to reconnect to the database and repeat your command.
WARNING:  terminating connection because of crash of another server process
DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
HINT:  In a moment you should be able to reconnect to the database and repeat your command.
WARNING:  could not create relation-cache initialization file "global/pg_internal.init.1380": No space left on device
DETAIL:  Continuing anyway, but there's something wrong.
172.28.83.181 - - [25/Jul/2019:09:51:07 +0000] "GET /deployments?exclude_configs=true&exclude_releases=true&exclude_stemcells=true HTTP/1.0" 500 - 0.0099
WARNING:  could not create relation-cache initialization file "global/pg_internal.init.1381": No space left on device
DETAIL:  Continuing anyway, but there's something wrong.
172.28.83.181 - - [25/Jul/2019:09:51:07 +0000] "GET /configs?type=resurrection&latest=true HTTP/1.0" 500 - 0.0097
WARNING:  terminating connection because of crash of another server process
DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
HINT:  In a moment you should be able to reconnect to the database and repeat your command.
```
- Postgres Log 확인, Disk 용량이 부족하다는 것을 확인
```
2019-07-25 09:58:28 GMT LOG:  could not close temporary statistics file "pg_stat_tmp/db_0.tmp": No space left on device
2019-07-25 09:58:28 GMT LOG:  could not close temporary statistics file "pg_stat_tmp/global.tmp": No space left on device
2019-07-25 09:58:29 GMT FATAL:  could not write init file
2019-07-25 09:58:30 GMT FATAL:  could not write init file
2019-07-25 09:58:31 GMT FATAL:  could not write init file
2019-07-25 09:58:32 GMT FATAL:  could not write init file
2019-07-25 09:58:32 GMT FATAL:  could not write init file
2019-07-25 09:58:32 GMT FATAL:  could not write init file
2019-07-25 09:58:38 GMT LOG:  using stale statistics instead of current ones because stats collector is not responding
2019-07-25 09:58:56 GMT FATAL:  could not write init file
```
- Ubuntu Disk Size 확인 결과 Persistence Disk Size 사용률이 100% 임을 확인
```
 df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           3.9G   23M  3.9G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1       2.9G  1.5G  1.3G  53% /
/dev/sdb2        42G  5.1G   35G  13% /var/vcap/data
tmpfs           1.0M   64K  960K   7% /var/vcap/data/sys/run
/dev/sdc1        50G   47G     0 100% /var/vcap/store
tmpfs           3.9G     0  3.9G   0% /var/vcap/data/bpm/tmpworkaround
```

- Bosh Director의 Resource Config Persistence Disk Size을 증설 후 Issue 해결

### 3.3. Diego Cell Garden Timeout Error
- Diego Cell이 Update 중 Garden의 Postscript에서 에러 발생, Diego Cell이 일시적으로 내려 갈 때 Application이 다른 Diego Cell에 옮겨지게 되는데 Diego Cell의 용량이나 Memory가 부족하여 발생한 에러 로 추정
	- 해결 방법:  Version을 돌린 후 Diego Cell의 크기를 Scale Up/Out 또는 불필요한 Application을 전체 제거 후 Version Update 진행
```
Fri Jul 26 04:58:24 UTC 2019: Pinging garden server...
Fri Jul 26 04:58:24 UTC 2019: Attempt 1...
Fri Jul 26 04:58:25 UTC 2019: Attempt 2...
Fri Jul 26 04:58:26 UTC 2019: Attempt 3...
Fri Jul 26 04:58:27 UTC 2019: Attempt 4...
Fri Jul 26 04:58:28 UTC 2019: Attempt 5...
Fri Jul 26 04:58:29 UTC 2019: Attempt 6...
Fri Jul 26 04:58:30 UTC 2019: Attempt 7...
Fri Jul 26 04:58:31 UTC 2019: Attempt 8...
Fri Jul 26 09:37:14 UTC 2019: Timed out pinging garden server.
Mon Jul 29 01:06:16 UTC 2019: Pinging garden server...
Mon Jul 29 01:06:16 UTC 2019: Attempt 1
```
