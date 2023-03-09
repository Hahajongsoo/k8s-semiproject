# 개요
- 주제: k8s에서 wordpress 애플리케이션 구축
- 기간: 23-03-09 ~ 23-03-10

## 전체 구성

![구성도](image/project-architecture.svg)

Wordpress APP
- Deployment
- Probe, Resource
- HPA
- ConfigMap
- Secret
- PVC
- Service/Ingress

MySQL APP
- StatefulSet
- Probe, Resource
- Read Replica
- ConfigMap
- Secret
- PVC
- Service

# MySQL 구성
## Config Map
- Primary DB와 Read Replica 의 설정을 다르게 하기 위해서 사용
## Secret
- MySQL 이미지 실행시 필요한 환경변수들을 넘겨주기 위해서 사용
  - 루트 패스워드
  - 워드프레스용 DB와 사용자 관련 정보
## Service
- Stateful Set에 속한 파드들을 식별하기 위해서 헤드리스 서비스로 구성
## Stateful Set
- 파드마다 이름이 순서대로 정해지기 때문에 (mysql-0, mysql-1 ...) 이를 이용하여 쉘스크립트로 Primary 역할을 하는 파드와 Read Replica의 역할을 하는 파드들의 환경 구성을 다르게 함
- stateful set에 포함된 pvc에 자동으로 pv가 할당되게 하기 위해 동적 프로비저닝을 사용하고 이를 위한 default storage class로 nfs-subdir-external-provisioner를 사용함

# Wordpress 구성
## Config Map
- liveness probe를 위한 apache 구성과 인덱스 파일을 넣기 위해서 사용함
## Secret
- wordpress가 사용할 DB에 대한 정보를 파드의 환경변수로 넘겨주기 위해 사용함
## PVC
- 여러 파드들이 하나의 볼륨을 바라볼 수 있게하기 위해 pvc를 사용함
## HPA
- 파드 cpu 사용량을 기준으로 HPA 구성
## Deployment
- 각 파드가 stateful 할 필요는 없음
- 데이터는 PV와 연결한 DB에 저장함
## Ingress
- certbot을 이용하여 인증서를 발급받아 tls 적용