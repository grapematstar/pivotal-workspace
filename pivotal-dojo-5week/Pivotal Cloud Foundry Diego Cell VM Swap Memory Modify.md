## 1. Pivotal Cloud Foundry Diego Cell VM Swap Memory  용량 조절 방법

- 전제 조건
	- Bosh/Pivotal Cloud Foundry 가 설치 되어 있어야 한다.

### 1.1 Diego Cell Swap 용량 조절

#### 1.1.1. Diego Cell 설치 시 Disk 용량의 반이 Swap으로 자동으로 할당
```
# diego cell 접속
$ bosh -d cf-54c9c2f906b6aba996cd ssh diego_cell/0

# df -h로 용량 확인 64G의 디스크를 할당 할 경우 /dev/sdb2에 32G Swap에 32G의 용량이 할당 되며 Swap은 사용되지 않음.

diego_cell/d097fe0e-0357-4698-b887-c0af632417a8:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         16G     0   16G   0% /dev
tmpfs            16G     0   16G   0% /dev/shm
tmpfs            16G  1.7G   15G  11% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            16G     0   16G   0% /sys/fs/cgroup
/dev/sda1       2.9G  1.4G  1.4G  49% /
/dev/sdb2        32G   21G  9.9G  68% /var/vcap/data
tmpfs           1.0M   52K  972K   6% /var/vcap/data/sys/run
tmpfs            16G     0   16G   0% /var/vcap/data/bpm/tmpworkaround
/dev/loop0       32G  4.7G   28G  15% /var/vcap/data/grootfs/store/unprivileged
/dev/loop1       32G   65M   32G   1% /var/vcap/data/grootfs/store/privileged
diego_cell/d097fe0e-0357-4698-b887-c0af632417a8:~$ free -m
              total        used        free      shared  buff/cache   available
Mem:          32168        5893        2940        1723       23334       23614
Swap:         32167           8       32159
```
#### 1.1.2. Diego Cell 용량 조절 방법
```
$ uaac target https://127.0.0.1/uaa --skip-ssl-validation # Opsmanager cli를 사용하기 위해 opsmanger uaa에 접속한다.
$ uaac token owner get
Client ID: opsman
Client Secret: [Press Enter]
Username: admin
Password: *******

$ cat ~/.uaac.yml
$ export TOKEN="....access_token value" # uaac의 access_token을 환경 변수로 저장

$ curl https://$HOST/api/v0/staged/products/$PGUID/bs/$JGUID/resource_config 

## This will print the Diego Cell job guid.
$ curl -H "Authorization: bearer $TOKEN" https://$HOST/api/v0/staged/products/$PGUID/jobs | jq -r '.jobs[] | select(.name == "diego_cell") | .guid'. 

$ curl -H "Authorization: bearer $TOKEN" https://127.0.0.1/api/v0/staged/products/cf-54c9c2f906b6aba996cd/jobs/diego_cell-fc84ea1e3638be043ae5/resource_config -k | jq . > rc.json

ubuntu@opsmanager-2-4:~$ cat rc.json 
{
  "instance_type": {
    "id": "2xlarge"
  },
  "instances": 5, # 모든 diego_cell instance의 설정 정보
  "additional_networks": [],
  "nsx_security_groups": null,
  "nsx_lbs": [],
  "additional_vm_extensions": [],
  "swap_as_percent_of_memory_size": 0 # 해당 값을 0으로 설정한다.
}

# 수정한 Diego Cell의 리소스 파일을 교체 한다.
ubuntu@opsmanager-2-4:~$ curl -H "Authorization: bearer $TOKEN" https://127.0.0.1/api/v0/staged/products/cf-54c9c2f906b6aba996cd/jobs/diego_cell-fc84ea1e3638be043ae5/resource_config -k -X PUT -d @rc.json -H "Content-Type: application/json"

$ apply change
```
#### 1.1.3. Diego Cell 용량 조절 확인

```
# diego cell 접속
$ bosh -d cf-54c9c2f906b6aba996cd ssh diego_cell/0

# /dev/sdb1의 용량이 63G로 변경됨을 확인하고 Swap의 용량이 0임을 확인
diego_cell/b2b02abd-3321-4d9e-a9d3-5ae3fc3a9faf:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         16G     0   16G   0% /dev
tmpfs            16G     0   16G   0% /dev/shm
tmpfs            16G   63M   16G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            16G     0   16G   0% /sys/fs/cgroup
/dev/sda1       2.9G  1.4G  1.4G  49% /
/dev/sdb1        63G   15G   45G  25% /var/vcap/data 
tmpfs           1.0M   52K  972K   6% /var/vcap/data/sys/run
tmpfs            16G     0   16G   0% /var/vcap/data/bpm/tmpworkaround
/dev/loop0       63G  5.0G   58G   8% /var/vcap/data/grootfs/store/unprivileged
/dev/loop1       63G   97M   63G   1% /var/vcap/data/grootfs/store/privileged
diego_cell/b2b02abd-3321-4d9e-a9d3-5ae3fc3a9faf:~# free -m
              total        used        free      shared  buff/cache   available
Mem:          32168        4760        5748         131       21659       26474
Swap:             0           0           0
```
