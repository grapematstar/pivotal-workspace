


# 1. Pivotal Cloud Foundry & Bosh Prometheus Metric

- 전제 조건
	- BOSH/PAS가 설치되어 있어야 한다.
	- 외부와 통신이 가능한 Jumpbox(workspace server)가 존재 해야 한다.
 
## 1.1 Bosh Prometheus 아키텍처

![prometheus][prometheus-image-1]

- Bosh Prometheus Release 내부에는 각 Deployment(BOSH, Pivotal Cloud Foundy, Mysql, Redis, RabbitMq 등) 별 Metric을 수집하는 Exporter들이 존재 한다.
- Prometheus Exporter를 통해  수집 한 각 Deployment의 Metric을 Grafana에서 가공하여 시각화 한다.

## 1.2 Bosh Prometheus Deploy

### 1.2.1. Bosh Prometheus Deployment Download git

```
# git clone prometheus
$ git clone https://github.com/bosh-prometheus/prometheus-boshrelease.git

# 사용 할 prometheus git manifest directory 구조
manifests/prometheus.yml # prometheus 설치 main manifest 파일 
manifests/operators/monitor-bosh.yml  # bosh metric 관련 manifest 파일
manifests/operators/monitor-cf.yml # cf metric 관련 manifest 파일

# 릴리즈 다운로드 주소 & 릴리즈 업로드
$ wget https://bosh.io/d/github.com/cloudfoundry/postgres-release?v=36
$ wget https://github.com/bosh-prometheus/prometheus-boshrelease/releases/download/v25.0.0/prometheus-25.0.0.tgz

$ bosh upload-release prometheus-25.0.0.tgz
$ bosh upload-release postgres-release?v=36

# metric을 수집하기 위해 doppler와 cloud controller, bosh의 read 권한을 갖고 있는 uaac client 추가
$ uaac target ${UAA 주소}

$ uaac token client get admin -s ${UAA ADMIN CLINET PASSWORD}

$ uaac client add firehose_exporter \
  --name firehose_exporter \
  --secret "password" \
  --authorized_grant_types client_credentials,refresh_token \
  --authorities doppler.firehose

$ uaac client add cf_exporter \
  --name cf_exporter \
  --secret "password" \
  --authorized_grant_types client_credentials,refresh_token \
  --authorities cloud_controller.admin_read_only
  
$ uaac client add bosh_exporter \
  --name bosh_exporter \
  --secret "password" \
  --authorized_grant_types client_credentials,refresh_token \
  --authorities bosh.read \
  --scope bosh.read  || true #ignore errors

# prometheus deploy script
bosh -e b -d prometheus deploy manifests/prometheus.yml \
  --vars-store tmp/deployment-vars.yml \
  -o manifests/operators/monitor-bosh.yml \
  -v bosh_url=${BOST_TARGET_IP} \
  -v bosh_username=${BOST_USER_NAME} \
  -v bosh_password=${BOST_USER_PASSWORD} \
  --var-file bosh_ca_cert=${BOST_CA_CERT} \
  -v metrics_environment=leedh-pcf \ # 실제 grafana에 보여질 metrics 주소
  -o manifests/operators/monitor-cf.yml \ 
  -v metron_deployment_name=leedh-pcf \ # 
  -v system_domain=${CF_DOMAIN} \
  -v uaa_clients_cf_exporter_secret="password \ # cf_exporter uaa client password
  -v uaa_clients_firehose_exporter_secret=passworkd \ # firehose_exporter uaa client password
  -v traffic_controller_external_port=443 \ # cf 컴포넌트 loggregator port
  -v skip_ssl_verify=true

# 설치 완료 후 web url
-   alertmanager:  http://<nginx-ip-address>:9093
-   grafana:  http://<nginx-ip-address>:3000
-   prometheus:  http://<nginx-ip-address>:9090

# password 확인 파일
$ cat /home/ubuntu/prometheus-boshrelease/tmp/deployment-vars.yml
alertmanager_password: ncguup2wt6do2wommnyy
grafana_password: 1r1jr1oxxt873r7j21yt
grafana_secret_key: dxlse2wlchfafr4ldb3n
postgres_grafana_password: sxsfajchrv1ec7nhak86
prometheus_password: jhdmvpjowqfgphpo1q0p

# 설치가 완료되면 http://<nginx-ip-address>:3000를 통하여 들어온 Bosh/Pivotal Cloud Foundry의 시각화한 Metric 정보를 확인 한다.
```

## 2.1 Prometheus Customizing

### 2.1.1.  서로 다른 PAS의 Metric Environment 연동
```
# prometheus-25.0.0.tgz를 압축 해제 하여 내부 cf_exporter를 cf_exporter-2로 복사하여 job/package를 수정, firehose vm을 추가하여 배포하는 방법이 있지만 작업 공수가 많음.

# bosh prometheus 배포 시 prometheus2 vm의 job cf_exporter를 vm으로 설치하여 각 각의 pas에 mapping 시킨다. 방법은 아래와 같다.

$ cp monitor-cf.yml monitor-cf-2.yml
# monitor-cf-2.yml을 수정
# monitor-cf-2.yml 수정 완료 파일


ubuntu@opsmanager-2-4:~/prometheus-boshrelease/manifests/operators$ cat monitor-cf-2.yml
# Apply ./cf/add-prometheus-uaa-clients.yml to your cf-deployment based Cloud Foundry
# This file assumes bosh_exporter based Service Discovery is being used: ./monitor-bosh.yml

# Exporter jobs
- type: replace
  path: /instance_groups/-
  value:
    name: cf_exporter-dkpcf
    azs:
      - az1
    instances: 1
    vm_type: medium
    stemcell: default
    networks:
      - name: service-network
    jobs:
      - name: cf_exporter
        release: prometheus
        properties:
          cf_exporter:
            cf:
              api_url: https://api.cf.domain.com
              client_id: cf_exporter
              client_secret: "password"
              deployment_name: leedh-pcf-2
            metrics:
              environment: "leedh-pcf-2"
            skip_ssl_verify: ((skip_ssl_verify))
            web:
              port: "9099"

- type: replace
  path: /instance_groups/-
  value:
    name: firehose-2
    azs:
      - az1
    instances: 1
    vm_type: medium
    stemcell: default
    networks:
      - name: service-network
    jobs:
      - name: firehose_exporter
        release: prometheus
        properties:
          firehose_exporter:
            doppler:
              subscription_id: "leedh-pcf-2"
              max_retry_count: 300
            logging:
              url: wss://doppler.cf.domain.com:((traffic_controller_external_port))
              use_legacy_firehose: true
            uaa:
              url: https://uaa.domain.com
              client_id: firehose_exporter
              client_secret: "password"
            metrics:
              environment: "leedh-pcf-2"
            skip_ssl_verify: ((skip_ssl_verify))

# Prometheus Alerts
- type: replace
  path: /instance_groups/name=prometheus2/jobs/name=cloudfoundry_alerts?/release
  value: prometheus

- type: replace
  path: /instance_groups/name=prometheus2/jobs/name=prometheus2/properties/prometheus/rule_files/-
  value: /var/vcap/jobs/cloudfoundry_alerts/*.alerts.yml

# Grafana Dashboards
- type: replace
  path: /instance_groups/name=grafana/jobs/name=cloudfoundry_dashboards?/release
  value: prometheus

- type: replace
  path: /instance_groups/name=grafana/jobs/name=grafana/properties/grafana/prometheus/dashboard_folders/name=Cloudfoundry?/files/-
  value: /var/vcap/jobs/cloudfoundry_dashboards/cf*.json

- type: replace
  path: /instance_groups/name=grafana/jobs/name=grafana/properties/grafana/prometheus/dashboard_folders/name=Prometheus?/files/-
  value: /var/vcap/jobs/cloudfoundry_dashboards/prometheus*.json

# 파일 완성이 끝나면 -o manifests/operators/monitor-cf-2.yml 파라미터를 추가 하여 재설치를 진행 한다.

# prometheus deploy script
bosh -e b -d prometheus deploy manifests/prometheus.yml \
  --vars-store tmp/deployment-vars.yml \
  -o manifests/operators/monitor-bosh.yml \
  -v bosh_url=${BOST_TARGET_IP} \
  -v bosh_username=${BOST_USER_NAME} \
  -v bosh_password=${BOST_USER_PASSWORD} \
  --var-file bosh_ca_cert=${BOST_CA_CERT} \
  -v metrics_environment=leedh-pcf \ # 실제 grafana에 보여질 metrics 주소
  -o manifests/operators/monitor-cf.yml \ 
  -o manifests/operators/monitor-cf-2.yml \
  -v metron_deployment_name=leedh-pcf \ # 
  -v system_domain=${CF_DOMAIN} \
  -v uaa_clients_cf_exporter_secret="password \ # cf_exporter uaa client

# grafana 화면에 접속하여 Metric가 제대로 출력 되는지 확인한다. 적용 되기 까지 시간이 조금 걸릴수 있다.
```
[prometheus-image-1]:./images/prometheus-1.PNG
