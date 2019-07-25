# Pivotal Cloud Foundry Application Instance_id 추출 방법
- Needs: Pivotal Cloud Foundry에 Push한 Application의 guid는 유일하지만 Instance 수가 복수이거나 증설되면 instance_id는 각 각 다름. Application의 instance_id를 추출하는 방법에 대해 기술

## 1. Pivotal Cloud Foundry의 Mysql에 직접 접근하여 query로 결과 값을 추출하는 방법
```
$ bosh -e {alias-env} -d {cf-deployment} ssh {mysql-vm} \
 -c "/var/vcap/packages/pxc/bin/mysql \
 --defaults-file=/var/vcap/jobs/pxc-mysql/config/mylogin.cnf \
 -e 'use diego; select SUBSTR(process_guid, 1, 36), instance_guid, cell_id, instance_index  from actual_lrps;'" | awk '{print $5, $7, $9, $11}'

# 결과 값
02ecab9f-695c-4c9e-9f69-e7fcbb461910 94f265f4-0b2b-4ecf-4995-d1a7 bc15969b-8522-42fb-9cf5-9277c63588dd 0
02ecab9f-695c-4c9e-9f69-e7fcbb461910 a1fbed38-08e5-45fb-6c61-c540 cec919ad-f7b8-4fb9-af99-24a08837ef20 1
02ecab9f-695c-4c9e-9f69-e7fcbb461910 0b24c1fa-ef13-49ea-758c-2325 5244b492-3477-42bd-a6b0-d4d4cba5bfb9 2
03f962a8-7956-4dd8-9b31-fdbb8237cb00 327b0284-a4e7-493e-4bc5-4df5 bc15969b-8522-42fb-9cf5-9277c63588dd 0
```
## 2. Pivotal Cloud Foundry의 Diego Cell에 있는 cfdot 명령어를 통해추출하는 방법
```
 $ bosh -e {alias-env} -d {cf-deployment} ssh {diego-cell-vm} -c '/var/vcap/packages/cfdot/bin/cfdot actual-lrp-groups  --bbsURL https://bbs.service.cf.internal:8889 --caCertFile /var/vcap/jobs/cfdot/config/certs/cfdot/ca.crt \
 --clientCertFile /var/vcap/jobs/cfdot/config/certs/cfdot/client.crt \
 --clientKeyFile /var/vcap/jobs/cfdot/config/certs/cfdot/client.key \
 | /var/vcap/packages/cfdot/bin/jq -r ".instance | \"\(.process_guid[0:36]) \(.index) \(.instance_guid) \(.cell_id) \(.address)\" "' | awk '{print $4, $5, $6, $7, $8}'

# 결과 값
02ecab9f-695c-4c9e-9f69-e7fcbb461910 0 94f265f4-0b2b-4ecf-4995-d1a7 bc15969b-8522-42fb-9cf5-9277c63588dd 172.28.86.31
02ecab9f-695c-4c9e-9f69-e7fcbb461910 1 a1fbed38-08e5-45fb-6c61-c540 cec919ad-f7b8-4fb9-af99-24a08837ef20 172.28.86.29
02ecab9f-695c-4c9e-9f69-e7fcbb461910 2 0b24c1fa-ef13-49ea-758c-2325 5244b492-3477-42bd-a6b0-d4d4cba5bfb9 172.28.86.30
03f962a8-7956-4dd8-9b31-fdbb8237cb00 0 327b0284-a4e7-493e-4bc5-4df5 bc15969b-8522-42fb-9cf5-9277c63588dd 172.28.86.31
03f962a8-7956-4dd8-9b31-fdbb8237cb00 1 8d63cbb3-bcc0-45be-6ed8-15a4 cec919ad-f7b8-4fb9-af99-24a08837ef20 172.28.86.29

```

## 3. Mysql/Cell에서의 결과 값 차이점 & 결과 값 내용 공유
- mysql에서 추출한 결과 값은 신뢰성을 보장하지만,  cell의 ip를 추출하지 못함(컬럼이 존재하지 않음)
- app_guid는 존재하지만 instance_id가 존재 하지 않을 경우 해당 Application은 Crash 상태임을 확인해야 한다.
