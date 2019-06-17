
# 1. VM Certificate 만료 인증서 교체
- 개요
	- Ops Manager, Bosh Director를 통해 배포 한 VM의 대한 Certificate의 만료 주기는 1~4년이며, 인증서가 만료 되면 모든 프로세스가 동작하지 않는다.
	- Pivotal Product는 Opsmanager API를 바탕으로 Certificate를 교체 할 수 있다.

## 1.1. Pivotal Deployment 인증서 교체

### 1.1.1. Pivotal Deployment 인증서 만료 일 확인

- Ops Manager의 인증서 만료 기간은 4년이다.
- Ops Manager API를 사용하기 위해 Ops Manager UAA에 접속해 Token 정보를 받는다.
```
# 전체 작업은 Ops Manager Shell에서 실행 하였다.
# uaac 로그인
$ uaac target https://127.0.0.1/uaa --skip-ssl-validation

# uaac 토큰 권한 받기
$ ubuntu@opsmanager-2-4:~$  uaac token owner get
Client ID:  opsman
Client secret:
Unknown key: Max-Age = 86400
User name:  pcfadmin
Password:  *******

# 위 작업으로 등록된 토큰을 확인 한다.
$ cat ~/.uaac.yml # opsman clinet의 access_token 값 복사하기

# Token 값을 편하게 사용하기 위해 환경 변수로 삽입 한다.
$ export TOKEN="....access_token value"

# Opsmanager API를 호출하여 certificate의 만료일을 확인 한다.
$ curl "https://127.0.0.1/api/v0/certificate_authorities" \
    -H "Authorization: Bearer $TOKEN" -k | jq

####################### 실행 결과는 아래와 같다 ###########################
{
  "certificate_authorities": [
    {
      "guid": "4532adf8073308712db2",
      "issuer": "Pivotal",
      "created_on": "2019-04-24T04:17:05Z", # 인증서 생성 일자
      "expires_on": "2023-04-24T04:17:05Z", # 인증서 만료 일자
      "active": true,
      "cert_pem": "-----BEGIN CERTIFICATE----------END CERTIFICATE-----\n",
      "nats_cert_pem": "-----BEGIN CERTIFICATE----------END CERTIFICATE-----\n"
    }
  ]
}
#######################################################################

# 만료 할 인증서 확인 방법은 아래와 같다. expires_within=${y 년, m 월, d 일}
$ curl "https://127.0.0.1/api/v0/deployed/certificates?expires_within=1y" \ 
    -H "Authorization: Bearer $TOKEN" -k | jq

####################### 실행 결과는 아래와 같다 ###########################
{
  "certificates": [
    {
      "configurable": false,
      "property_reference": null,
      "property_type": null,
      "product_guid": null,
      "location": "credhub",
      "variable_path": "/bosh_dns_health_server_tls",
      "issuer": "/CN=opsmgr-bosh-dns-tls-ca",
      "valid_from": "2019-04-25T08:19:32Z",
      "valid_until": "2020-04-24T08:19:32Z"
    },
    ······ 생략
  ]
}
#######################################################################
```

### 1.1.2. Pivotal Deployment Root CA 재생성
```
$ curl "https://127.0.0.1/api/v0/certificate_authorities/generate" \
    -X POST \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{}' -k
{"guid":"18fff9e0cfd780653400","issuer":"Pivotal","created_on":"2019-06-05T08:35:03Z","expires_on":"2023-06-05T08:35:03Z","active":false,"cert_pem":"-----BEGIN CERTIFICATE----------END CERTIFICATE-----\n","nats_cert_pem":"-----BEGIN CERTIFICATE----------END CERTIFICATE-----\n"}
```
### 1.1.3. Root CA 재생성 후 Apply Change

- 인증서 재생성 결과 active: false인 인증서를 각 Deployment에 할당 하기 위해 Apply Change를 실행 한다.
- Opsmanager > Bosh Director Tile > Director Config의 설정을 Recreate All VMs을 설정하여 모든 Tile들에 반영 되도록 한다.
-  OnDemand Tile의 Errand도 전체 Deploy로 선택한다.

### 1.1.4. 새로 재생성한 Root CA를 active: True로 만들어준다.
```
# 재생성한 CA의 guid 18fff9e0cfd780653400 를 넣고 active: True로 변환 한다.
$ curl "https://127.0.0.1/api/v0/certificate_authorities/18fff9e0cfd780653400/activate" \
  -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{}' -k | jq
```

### 1.1.5. 인증서를 전체 생성
- 새로운 ROOT CA에 대한 나머지 인증서 정보를 전체 재 생성한다.
```
# active 상태인 Root CA에 대한 인증서를 전체 재 생성 한다.
$ curl "https://127.0.0.1/api/v0/certificate_authorities/active/regenerate" -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" -d '{}' -k
```

### 1.1.6. 인증서 전체 재생성 후 Apply Change

- 전체 재생성 한 인증서를 각 Deployment에 할당 하기 위해 Apply Change를 실행 한다.
- Opsmanager > Bosh Director Tile > Director Config의 설정을 Recreate All VMs을 설정하여 모든 Tile들에 반영 되도록 한다.
-  OnDemand Tile의 Errand도 전체 Deploy로 선택한다.

### 1.1.7.  active: false 한 교체 전 인증서 삭제
- 이전에 사용하던 만료 일자가 얼마 남지 않은 인증서를 삭제 한다.
```
# 인증서를 검색 한다.
$ curl "https://127.0.0.1/api/v0/certificate_authorities" -X GET -H "Authorization: Bearer $TOKEN" -k | jq

# 인증서를 삭제 한다.
$ curl "https://127.0.0.1/api/v0/certificate_authorities/5713c94a51bdcb097e03" -X DELETE -H "Authorization: Bearer $TOKEN" -k
```

### 1.1.8.  Apply Change
- 삭제한 인증서를 적용하기 위해 다시 전체 Apply Change를 실행 한다.
