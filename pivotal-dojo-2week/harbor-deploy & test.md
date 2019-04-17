###1. harbor bosh release (private docker hub)
  - https://github.com/vmware/harbor-boshrelease
  
    #### harbor.yml 수정
    release : delete url, hash
    stemcell: version -> latest
    deployment-network -> your network in cloud-config
    az -> your az in cloud config
    필요시 deployment명 수정:
    smoke test 실패 수정:

    instance_groups:
    name: harbor-apㅔ

    jobs:
    name: harbor
    release: harbor-container-registry
    properties:

    admin_password_for_smoketest: ((harbor_admin_password_for_smoketest))

    variables:

    name: harbor_admin_password_for_smoketest
    type: password
    
    harbor는 ha 구성이 되지 않을수도있고 vmware에서만 올라간다.
    bosh deploy 시 dns 관련 runtime config를 적용시켜줘야함
 
  - harbor docker push inscure (IP에 대한 SANS x509 예외 방지)
  
     ubuntu@TLKPCFJB1:/etc/docker$ cat /etc/docker/daemon.json
     {
        "insecure-registries" : ["172.28.83.197"] # harbor 주소
     }
     $ docker service restart
 
  - etc/hosts에서 bosh-deploy시 내가 설정한 harbor host name을 명시
  
    ubuntu@TLKPCFJB1:/etc/docker$ cat /etc/hosts
    127.0.0.1       localhost
    172.28.83.54    TLKPCFJB1
    172.28.83.197 harbor.local
    
    $ docker login {ex)harbor.local} -u {id} -p {pwd}
    
    $ docker push {dokcer image}
    
    $ docker pull {docker image}
    
  
