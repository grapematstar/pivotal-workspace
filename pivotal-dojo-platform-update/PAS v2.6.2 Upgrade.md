
#  Upgrade Note 6 - Pivotal Cloud Foundry v2.6.2
- 진행한 Ops Manager와 Pivotal Cloud Foundry 현재 Version은 아래와 같다.
	- Ops Manager v2.6.5
	- Pivotal Cloud Foundry v2.5.11

## 1. Pivotal Cloud Foundry Release Note 2.6
- [Release Note](https://docs.pivotal.io/pivotalcf/2-6/pcf-release-notes/runtime-rn.html)
- [Breaking Change](https://docs.pivotal.io/pivotalcf/2-6/pcf-release-notes/breaking-changes.html)

### 1.1. Pivotal Cloud Foundry v2.6 주요 변경 사항

#### 1.1.1. Monitor System Metrics with System Metrics Agent
- System Metrics Agent를 사용하여 모니터링 기능 개선
	- VM 계산, Network 및 Storage Metric에 대한 가시성을 향상 
- 활성화 방법은 PAS Tile의 System Logging Config -> Enable System Metrics 속성을 활성화 한다.

#### 1.1.2. Metric Registrar Enabled by Default
- PAS Tile의 Metric Registrar Config -> Enable Metric Registrar 속성이 기본적으로 활성화 된다.

#### 1.1.3. Enable, View, and Rollback Revisions for Apps
- $ cf curl revision  기능을 통해 특정 시간에 Application이 사용하는 코드 및 구성에 대한 변화를 확인 할 수있다.

#### 1.1.4. Push Apps with Sidecar Processes (Beta)
- Container 안에서 Application과 통신 및 실행해야 하는 프로세스들을 사용 할 수있는 Sidecar를 실행 할 수있다.
- Application에 Localhost로 통신을 해야 할때나, APM(Application Performance Monitoring) 등에 유용하게 사용 됨.

#### 1.1.5. Garden Delegates Container Creation and Destruction to containerd by Default
- PAS의 Application Containers Config -> Enable Containerd Delegation를 활성화 하게 되면 Garden Container의 관리가 containerd에 위임된다, containerd는 다른 Container 기술까지 사용 할 수 있게끔 하는 관리자.

#### 1.1.6. NFS Legacy Mounter Removed
- PAS v2.6에서는 nfs-legacy NFS를 제거, nfs Instance는 계속 동작하지만 최신 nfs Mounter를 사용 한다.

#### 1.1.7. Extra Diego Root CA Added to Prevent Future Expiration Issues
- PAS v2.3에서 사용하는 CA가 2020년도에 만료 됨, Diego Root CA의 만료 기간을 방지하기 위해 Extra Diego Root CA 추가 되었다.

#### 1.1.8.Performance Improvements for Read-Write File Systems
- UID를 사용하여 Mount 되는 mapfs에 대한 파일을 읽고 쓰는 기능이 향상되었다.

#### 1.1.9. Increased CPUs for Router VMs
- 안정성 향상을 위해 Router VM의 CPU 수가 1->2 개로 증가

#### 1.1.10. Loggregator Syslog Agent Increases Scale For Syslog Drains
- Syslog Agnet를 추가하면 Loggregator System에서 지원하는 syslog Drain Service Binding 수가 증가하고 Loggregator VM의 작업 부하가 줄어든다.
- 설정 방법은 PAS Tile의 System Logging Config -> Enable agent-based syslog egress for app logs를 선택 한다.

#### 1.1.11. Terminate Specific Instances of an App in Apps Manager UI
- Apps Manager UI에서 Application의 특정 Instance를 종료 할 수 있다.

#### 1.1.12.  View and Edit Labels and Annotations Associated with an Organization
- Apps Manager UI 화면에서 Org/Space에 대한 Label Metadata를 수정 및 관리 할 수 있다.

### 1.2. Pivotal Cloud Foundry v2.6 Upgrade Check List

#### 1.2.1. Pivotal Cloud Foundry Backup
- Pivotal Cloud Foundry Version Upgrade 전 Platform에 대한 Backup을 권고

#### 1.2.2. Review Service Tile Compatibility
- 모든 Tile들을 Upgrade 대상 Pivotal Cloud Foundry와 호환되는지 확인

#### 1.2.3. Update Tiles and Add-Ons
- Pivotal Cloud Foundry Version Upgrade 하기 전 Upgrade 대상 Version과 모든 Tile과 Adds-on에 대한 호환성을 확인

#### 1.2.4. Upgrade Services Tiles
- 모든 Tile들을 Upgrade 대상 Pivotal Cloud Foundry와 호환되는 Version으로 교체

#### 1.2.5. Bosh Check Required Machine Specifications
- IaaS 별 Ops Manager의 권고 Disk 사양을 확인하여 적당한 Disk Size를 할당 해야 한다.

#### 1.2.6. PAS Remove Healthcheck from SSH Load Balancer
- SSH Load Balancer, Diego Brain에 할당한 Load Balancer의 HTTP Health Check 기능을 삭제 한다.

#### 1.2.7. PAS Configure Diego Cell Garbage collection
- PAS Application Container Config의 Docker Images "Disk-Cleanup Scheduling on Cell VMs"가 Clean up disk-space once usage fills disk로 설정되어 있을 경우 Minimum 10240 -> 15680으로 변경 해야 한다.

#### 1.2.8. PAS (Optional) Disable Unused Errands
- Upgrade 시간을 단축하기 위해 PAS의 Errand 기능을 Off 하고 배포 할 수 있다.

#### 1.2.9.  Check OS Compatibility of BOSH-Managed Add-Ons and Tiles
-   PAS v2.6으로 Upgrade전에 PAS용 IPsec, ClamAV, File Integrity Monitoring과 같은 PAS 추가 기능을 배포 한 운영자와 Windows 용 PAS를 배포했거나이를 계획중인 운영자는 지원 종료의 Stemcell이 변경됨에 따라 Add-ons 기능을 수정해야 할 수 있다.


#### 1.2.10. Check Backup and Restore External Blobstore Add-On
- Blobstore Add-On을 사용하여 Azure Blobstore에 대한 외부 Blobstore Backup을 활성화 한 경우 Upgrade하기 전에 sdk-preview 추가 기능을 제거하도록 런타임 구성을 Update 해야 한다.

#### 1.2.11. Check Certificate Authority Expiration Dates
- PAS 배포 시 생성한 인증서 정보의 만료 기간을 확인
- Ops Manager API를 통해 1 month(기간 설정) 이하의 인증서 확인
```
$ curl "https://OPS-MAN-FQDN/api/v0/deployed/certificates?expires_within=1m" \
 -X GET \
 -H "Authorization: Bearer UAA_ACCESS_TOKEN"
```
#### 1.2.14. Capacity - Confirm Adequate Disk Space
- Ops Manager와 PAS를 Upgrade 하게 되면 최소 20GB의 Disk 용량이 필요하게 되며, 기타 Tile들이 있을 경우 +알파
- Bosh Director의 Persistence Disk 사용률 이 50%가 넘게 되면 Bosh Director의 Disk 증설을 고려 해야 한다.

#### 1.2.14. Capacity  Check Diego Cell RAM and Disk
- Diego의 Disk/Memory가 충분히 있는지 확인한다.
- PAS 관련(Diego Cell이 Container에 할당 할 수 있는 남은 Memory/Disk 잔여량) KPI는 rep.CapacityRemainingMemory, rep.CapacityRemainingDisk

#### 1.2.15. Adjust Diego Cell Limits
- Diego Cell Update시 다른 Cell이 과부하가 걸리지 않도록 동시에 Update 할 수 있는 제한 지정
- PAS 1.10.0 이상의 Diego Cell max_in_flight은 4%

- 


## 2. Pivotal Cloud Foundry Update 절차

### 2.1. Pivotal Cloud Foundry Tile Download
- Pivotal Network에서 신규 Pivotal Cloud Foundry Tile v2.6.2를 Download 한다.

### 2.3. Pivota Cloud Foundry Apply Change
- Pivotal Cloud Foundry를 Apply Change 한다.

## 3. Issues
### 3.1. Pivotal Cloud Foundry Version Upgrade Error
- Concourse Pipeline을 통해 Upgrade 하던 중 PAS 설치는 완료 됬지만 Smoke Test 에러 에러 발생
- Smoke Test Error Log

```
Errand 'smoke_tests' completed with error (exit code 1)
Exit code 1
          CC code:       0  
          CC error code:   
          Request ID:    35dbe0ed-efc5-4dce-688f-323581428dec::76f058d5-7dd5-44d3-9e85-5fe3d1d132d8  
          Description:   {  
            "error_code": "UnknownError",  
            "description": "An unknown error occurred.",  
            "code": 10001  
          }  
            
```
- cf push/apps 등 명령어 에러 발생
- cloud_controller VM Log

```
{"timestamp":1564704928.5126135,"message":"Started GET \"/v2/apps/16078665-c93b-47e2-a668-7015b516b081\" for user: firehose-to-syslog, ip: 172.28.86.61 with vcap-request-id: d8a3211d-425b-4a51-51c9-ac0d5e65bddb::99a7e8db-e253-4424-a963-3933157451e3 at 2019-08-02 00:15:28 UTC","log_level":"info","source":"cc.api","data":{"request_guid":"d8a3211d-425b-4a51-51c9-ac0d5e65bddb::99a7e8db-e253-4424-a963-3933157451e3"},"thread_id":47445431623300,"fiber_id":70173994633920,"process_id":1,"file":"/var/vcap/data/packages/cloud_controller_ng/dad6ec7571496c7d17c5a731f36f9443d372eec2/cloud_controller_ng/middleware/request_logs.rb","lineno":12,"method":"call"}
```

- PAS Tile -> Cloud Controller Tile의 DB encrypt_key가 빈 값인데 Pipeline PAS Config File인 .cloud_controller.encrypt_key는 변수로 특정 값을 받아와 발생 한 에러라고 추정
    - 해결 방법: secret: .cloud_controller.encrypt_key -> secret: {기존 password} 으로 변경하고 다시 Apply Change

- Ops Manager Diff Manifest

```
       cc:
+        db_encryption_key: "((/opsmgr/cf-7dd8b2324d3bed84c77c/cloud_controller/encrypt_key.value))"
-        db_encryption_key: "((/opsmgr/cf-7dd8b2324d3bed84c77c/cloud_controller/db_encryption_credentials.password))"
```


