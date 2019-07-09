# Pivotal CLoud Foundry Application Backup & Upload

- Pivotal CLoud Foundry에 Push한 Application의 Metadata Droplet을 Backup하고 다시 Backup 받은 파일을 Upload 하는 방법에 대해 기술 하였다.
- 사용 가이드는 $ cf Curl(CF API)를 사용한 스크립트로 구성 하였다.
- 전제 조건
  - cf CLI, jq CLI가 존재 해야 한다.
  - Pivotal Cloud Foundry에 Target/Login이 되어 있어야 한다.

## 1. Application 명 추출
### 1.1. Backup 할 Application 명을 특정 txt 파일에 지정
- 본 가이드에서는 app_names라는 파일에 Application 명을 넣었다.
```
$ cat app_namse
test-1
test-2
test-3
test-4
test-5
test-6
```

## 2. Application Droplet Backup
#### 2.1. app_namse에서 불러온 Application 명을 Loop를 실행하여 바탕으로 Application Guid를 추출 한다.
#### 2.2. Application Guid를 바탕으로 다시 Loop를 실행하여 Droplet을 다운로드 한다.

```
$ cat application_backup.sh

#!/bin/bash

filename=app_names
declare -a myArray
myArray=(`cat "$filename"`) # 파일을 통해 Application 명을 배열로 받아 온다.

for (( i = 0 ; i < ${#myArray[@]} ; i++)) # 배열의 크기 만큼 Loop 실행
do
  echo "app_name [$i]: ${myArray[$i]}"
  app_name=${myArray[$i]}
  app_guid=$(cf curl "/v2/apps" -X GET -H "Content-Type: application/x-www-form-urlencoded" -d 'q=name:'$app_name | jq -r ".resources[].metadata.guid") # Application의 GUID를 추출
  echo "app guid [$i]: $app_guid"

  if [ -n "$app_guid" ]; then
    echo "app_guid is not empty & back-up $app_name.tgz"
        cf curl "/v2/apps/$app_guid/droplet/download" --output $app_name.tgz # Application Droplet Download
  fi

  file=$app_name.tgz
  if [ -f "$file" ]; then
    echo "backup file found & delete cf app"
    cf delete $app_name -r -f # Backup 받은 Application 삭제
  fi
done
```

## 3. Application Upload
### 3.1. Backup 받은 Application Droplet을 통해 Pivotal Cloud Foundry에 Application을 재 배포 한다.
- example
```
$ cf curl "/v2/apps/796238a7-00f1-45e1-b030-59432f4648dc/droplet/download" --output authcode-sample.tgz
```

