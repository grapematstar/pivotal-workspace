https://bosh.io/docs/runtime-config
https://github.com/cloudfoundry/os-conf-release
timezone변경하는 bosh-release를 개발해야함.(https://github.com/cloudfoundry/os-conf-release)

반영 자동화: install opsman director concourse pipeline에 자동화하여 반영

참고:https://github.com/myminseok/pivotal-docs/blob/master/bosh-add-on.md?fbclid=IwAR2yLXdmX8qm2ulrbGQUjo2J40SS9YKMqk0KWmpIrn23b5a2Xz25b99GgbY

-- os conf release를 사용하여 반영하는 방법 --
releases:
 - name: os-conf
   version: 20.0.0

addons:
  - name: os-configuration
    jobs:
    - name: pre-start-script
      release: os-conf
      properties:
        script: |-
          #!/bin/bash
          ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
    include:
      deployments:
      - zookeeper-dongsik




timezon.yml 파일 예제  ( Deployment _name 값 입력)
==================

```
releases:
- name: timezone
  version: 1.0.0

addons:
- name: timezone-addon
  jobs:
  - name: timezone
    release: timezone
  include:
    deployments:
    - Pivotal_Single_Sign-On_Service-4bc48f80c0ec40bb7c35
    - apmPostgres-6f17013fecb4976e5fb0
    - cf-3f05e4a809c17445d857
    - harbor-container-registry-c6595a3eaf148982f07a
    - p-cloudcache-65071dc5c93fc78eea77
    - p-healthwatch-e829a0905eca9c194972
    - p-rabbitmq-5f29c209694d38b20e45
    - p-redis-940c17c94d42338affc7
    - p-spring-cloud-services-ac35eb3b78a4c473f09d
    - pivotal-mysql-dbdcefc9d41cefebae34
    - service-instance_2fd17eb8-6a70-43a9-8afb-c1e80af8260a
    - service-instance_5875ae92-4072-4254-90af-91b50bc355a8

```


app 배포시 Manifest.yml 파일 옵션추가 (TZ: Asia/Seoul)
====================================
```
---
applications:
- name: testconf2
  memory: 1024M
  instances: 1
  buildpack: oracle_java_buildpack
  host: testconf
  env:
   TZ: Asia/Seoul
   JBP_CONFIG_DEBUG: '{enabled: true}'
