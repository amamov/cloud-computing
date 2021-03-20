# IT 인프라

<br>

## VPS

### AWS EC2

### AWS Lightsail

### SSNODES

- [ssdnodes](https://www.ssdnodes.com/)

<br>

## Storage

### Azure Storage

### Amazone S3

### Google Cloud Storage

<br>

## Database

### Azure Database for PostgreSQL

### Google Cloud SQL

<br>

## Cache/Redis server

### Azure Cache for Redis

### Amazone Elastic Cache

### Google Cloud Memorystore

<br>

## Docker Container Orchestation

### Azure

#### Azure Container Registry

Docker Registry 관리형 서비스

#### Azure Container Instances

> 요새 잘 쓰이지 않음

VM을 직접 프로비저닝 하지 않고, VM 기반의 컨테이너

#### Azure Kubernetes Service

> 진입 장벽이 매우 높다.

K8S 관리형 서비스

#### Azure Web App for Containers

**Docker도 지원하는 PaaS 플랫폼**

#### Azure Functions

Serverless 플랫폼.

Docker를 통한 운영도 지원 (AWS, GCP는 아직까지 지원 X)

<br>

### GCP

#### Google Container Registry

Docker Registry 관리형 서비스

#### Google Compute Engine

서비스 차원에서 VM으로의 Docker 지원

#### Google Kubernetes Engine

본진의 K8S 관리형 서비스

#### Google AppEngine Flexible

> Azure Web App for Containers와 비슷한 서비스

**PaaS 플랫폼.**

Docker를 통한 운영도 지원

<br>

### AWS

#### Elastic Container Registry

Docker Registry 관리형 서비스

#### Fargate

ECS/EKS에서 구동. 리소스 제어권을 AWS에 위임

#### Elastic Container Service

AWS만의 EC2 컨테이너 서비스

#### Elastic Kubernetes Service

K8S 관리형 서비스

#### Elastic Beanstalk

PaaS 플랫폼이라고 하지만, 자동화된 IaaS에 가깝다.

#### AWS Lambda

Serverless 플랫폼 AWS에서 직접 운영하는 VM에서 구동. Docker 지원 X
