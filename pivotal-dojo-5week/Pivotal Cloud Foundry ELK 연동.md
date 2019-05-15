## Pivotal Cloud Foundry Log ELK

### 1. Opensource Bosh ELK Stack Deployment Deploy
- 전제 조건: Opensource BOSH가 설치 되어 있어야한다.

#### 1.1.  PCF Loggregator 아키텍쳐
```
https://docs.pivotal.io/pivotalcf/2-5/loggregator/architecture.html
PCF 내부 VM안의 agent와 cf logs 명령어를 통해 Loggreagtor VM의 Traffic Controller가 Application, VM 로그를 수집 및 제공 한다.
외부 SYSLOG Server(ELK의 Logstash로 연동 시키면 ELK에서 VM의 로그를 확인할수있다.)
```

#### 1.2.  ELK Bosh Release Deploy

##### 1.2.1. 아래 git 주소를 통해 ELK Deployment를 clone한다.
```
$ git clone https://github.com/bosh-elastic-stack/elastic-stack-bosh-deployment
```
##### 1.2.2. 외부와 인터넷 통신이 되지 않을 경우 아래 2파일을 기준으로 인터넷이 되는 공간에서 release/stemcell을 다운로드 받고 해당 다운로드 URL을 로컬 file 주소로 변경 한다.
```
elastic-stack-bosh-deployment/elastic-stack.yml
elastic-stack-bosh-deployment/versions.yml
```

##### 1.2.3. Test 할 logstash.conf를 작성한다.
```
cat <<EOF > logstash.conf
input {
  tcp {
     port => 5514 # input port (pas vm syslog를 밀어줄 PORT 번호)
  }
}
output {
  stdout {
    codec => json_lines
  }
  elasticsearch {
    hosts => __ES_HOSTS__
    index => "logstash-%{+YYYY.MM.dd}" (실제 Kibana, Elastic Serarch의 인덱스)
  } 
}
EOF
```

##### 1.2.4. bosh deploy 명령어 실행
```
bosh -d elastic-stack deploy elastic-stack.yml \
     -l versions.yml \
     -o ops-files/vm_types.yml \
     -o ops-files/disk_types.yml \
     -o ops-files/instances.yml \
     -o ops-files/networks.yml \
     -o ops-files/azs.yml \
     -o ops-files/elasticsearch-add-lb.yml \
     -o ops-files/elasticsearch-add-data-nodes.yml \
     -o ops-files/elasticsearch-add-plugins-master.yml \
     -o ops-files/elasticsearch-add-plugins-data.yml \
     -o ops-files/logstash-add-lb.yml \
     -o ops-files/logstash-readiness-probe.yml \
     -o ops-files/kibana-https-and-basic-auth.yml \
     -o ops-files/kibana-add-lb.yml \
     --var-file logstash.conf=logstash.conf \
     -v elasticsearch_master_instances=3 \
     -v elasticsearch_master_vm_type=minimal \
     -v elasticsearch_master_disk_type=5120 \
     -v elasticsearch_master_network=default \
     -v elasticsearch_master_azs="[z1, z2, z3]" \
     -v elasticsearch_data_instances=2 \
     -v elasticsearch_data_vm_type=minimal \
     -v elasticsearch_data_disk_type=5120 \
     -v elasticsearch_data_network=default \
     -v elasticsearch_data_azs="[z1, z2, z3]" \
     -v logstash_instances=2 \
     -v logstash_vm_type=minimal \
     -v logstash_disk_type=5120 \
     -v logstash_network=default \
     -v logstash_azs="[z1, z2, z3]" \
     -v logstash_readiness_probe_http_port=0 \
     -v logstash_readiness_probe_tcp_port=5514 \
     -v kibana_instances=1 \
     -v kibana_vm_type=minimal \
     -v kibana_network=default \
     -v kibana_azs="[z1, z2, z3]" \
     --no-redact
```

### 2. PAS VM, Application 연동

#### 2.1. PAS VM SYSLOG 연동
##### 2.1.1. Director Tile과 PAS의 Syslog Config를 수정
```
Address: logstash 주소 ex) 172.16.100.32
Port: logstash input port ex)5514
Transport Protocol: logstash input Protocal ex) tcp

# Director와 PAS의 Syslog Config 설정이 완료되면 Apply Change 버튼을 클릭한다.
# Syslog Release가 Runtime Config 형식으로 붙으며 VM의 Syslog가 Logstash에 날라 간다.
```

#### 2.2. PAS App Logs 연동
##### 2.2.1. Cf Drain Plugin Install
```
CF Apps의 로그를 ELK로 전달하기 위해서 특정 App을 syslog drain을 걸어줘야하는데
CF Drain 기능을 사용하기 위해 특정 Plugin을 설치한다.

$ cf install-plugin -r CF-Community "drains"
# 내부망일 경우 외부망이 되는 PC에서 drains 바이너리 파일을 다운로드 받아 설치 한다.

# 설치가 완료 되면 아래의 cf 명령어가 생겨 있다.
$ cf drain -h

# syslog drain 사용자 서비스를 생성한다.
$ cf cups log-drain -l syslog://<logstash-serve>:<logstash-port>

# 만들어진 서비스를 test application에 bind 한다.
$ cf bind-service cf-spring log-drain

# test application을 재시작 한다.
$ cf restart cf-spring
```

### 3. Kibana 확인
#### 3.1. Kibana의  Web 화면에서 Index를 생성한다.
```
Index 명은 위 logstash config 파일의 logstash-%{+YYYY.MM.dd} 명과 같다.
logstash-*의 인덱스를 지정하여 생성하고 로그를 색인검색하여 확인한다.
```
