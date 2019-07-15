#  Pivotal Cloud Foundry Blobstore Redeploy 중 Error 발생

- Pivotal Cloud Foundry의 실제 Buildpack, Droplet, Application 등의 Metadata 정보들이 저장되어 있는 Blobstore를 재설치 중 Error 발생
- Pivotal Cloud Foundry Version 2.4

## 1. BlobStore Error 확인

### 1.1. Error 발생 원인

- Blobstore의 용량을 증설 계획으로 Deploy 중 사용하고 있는 용량보다 작게 설정하여 Deploy를 시도 하다 에러가 발생
- 용량 증설 계획 300G -> 500G, 실제 배포 용량 300G -> 100G

```
 instance_groups:  
 - name: nfs_server  
-   persistent_disk_type: '307200'  
+   persistent_disk_type: '102400'

Error: Response exceeded maximum allowed length # 실제 배포 Error 메세지
```
- Blobstore 용량을 다시 300G로 변경 후 Deploy를 시도 시 다시 에러가 발생, Blobstore Job의 PreStart 스크립트에서 에러 발생
```
Task 54620 | 07:57:13 | Updating instance nfs_server: nfs_server/709f8037-8953-4806-aad2-b608f6d0a25d (0) (canary) (00:00:20)  
                    L Error: Action Failed get_task: Task 13331b15-4877-4315-5edd-a324b492cd0b result: 1 of 4 pre-start scripts failed. Failed Jobs: blobstore. Successful Jobs: bpm, syslog_forwarder, bosh-dns.  
Task 54620 | 07:57:33 | Error: Action Failed get_task: Task 13331b15-4877-4315-5edd-a324b492cd0b result: 1 of 4 pre-start scripts failed. Failed Jobs: blobstore. Successful Jobs: bpm, syslog_forwarder, bosh-dns.
```

## 2. BlobStore Error 발생 후 Platform 증상
- $ Application cf push 배포가 되지 않음.
- Application Droplet Backup이 되지 않음.
- bosh cck 명렁어 및 bosh restart vm시 동일한 에러가 발생
- BlobStore의 모든 Monit의 형상이 나타나지 않음
- Bosh를 통해 VM Recreate 시 VM 내의 일부 Disk 데이터가 삭제되어 진행을 하지 못함.

## 3. BlobStore Error 추적

### 3.1. Blobstore VM Log 확인

#### 3.1.1.  Blobstore VM에 접속하여 Blobstore Job의 pre-start 로그를 확인
```
$ bosh -e {ailas-env} -d {cf-deployment} ssh {blobstore-vm}

$ cat var/vcap/sys/log/blobstore/pre-start.stderr.log
+ setup_blobstore_directories
+ local run_dir=/var/vcap/sys/run/blobstore
+ local log_dir=/var/vcap/sys/log/blobstore
+ local store_dir=/var/vcap/store/shared
+ local data_dir=/var/vcap/data/blobstore
+ local store_tmp_dir=/var/vcap/store/shared/tmp/uploads
+ local data_tmp_dir=/var/vcap/data/blobstore/tmp/uploads
+ local nginx_webdav_dir=/var/vcap/packages/nginx_webdav
+ mkdir -p /var/vcap/sys/run/blobstore
+ mkdir -p /var/vcap/sys/log/blobstore
+ mkdir -p /var/vcap/store/shared
+ mkdir -p /var/vcap/store/shared/tmp/uploads
+ mkdir -p /var/vcap/data/blobstore
+ mkdir -p /var/vcap/data/blobstore/tmp/uploads
+ chown vcap:vcap /var/vcap/store/shared
chown: changing ownership of '/var/vcap/store/shared': Read-only file system
```
#### 3.1.2.  Blobstore Job Setup Pre-start 중 /var/vcap/store/shared에 대한 Mount 파티션 권한이 Read Only를 확인.

```
$ cat /proc/mounts | grep store
/dev/sdd1 /var/vcap/store ext4 ro,relatime,data=ordered 0 0
```

#### 3.1.3.  Test 환경(다른 PAS의) /var/vcap/store Mount 파티션 권한 확인
```
$ cat /proc/mounts | grep store
/dev/sda1 /var/vcap/store ext4 rw,relatime,data=ordered 0 0
```

#### 3.1.4.  Dev 환경과 Test 환경에 대한 Mount 파티션의 권한이 다르다는 것을 확인하고 Dev PAS의 Blobstore 트러블 슈팅
- mount 파티션 권한을 read & write로 변경
```
$ mount -o remount,rw /dev/sdd1
```
- bosh restart vm

```
$ bosh -e {ailas-env} restart {blobstore-vm}
```

- Bosh Restart VM 후 Monit Summary로 Blobstore Job이 정상 동작 함을 확인
```
$ monit summary
The Monit daemon 5.2.5 uptime: 3d 15h 59m

Process 'loggregator_agent'         running
Process 'blobstore_nginx'           running
Process 'blobstore_url_signer'      running
Process 'route_registrar'           running
Process 'bosh-dns'                  running
Process 'bosh-dns-resolvconf'       running
Process 'bosh-dns-healthcheck'      running
System 'system_localhost'           running
```
- $ cf Push 명령어를 통해 Application이 정상 배포 되는지 확인
