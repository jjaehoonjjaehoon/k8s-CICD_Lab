# Technical Summary

## 프로젝트 목적

이 프로젝트는 GitHub Actions, Azure Container Registry, Azure Kubernetes Service, Argo CD를 활용하여 컨테이너 이미지 빌드부터 AKS 배포 및 외부 접속 검증까지의 기본 CI/CD 흐름을 실습하기 위해 진행했습니다.

정적 웹 페이지인 `index.html`을 Nginx 기반 Docker 이미지로 패키징하고, GitHub Actions를 통해 Azure Container Registry에 Push한 뒤, AKS에서 Deployment와 LoadBalancer Service를 통해 실행 및 외부 접속을 확인했습니다.

## 사용 기술 요약

| 구분 | 기술 | 경험 내용 |
|---|---|---|
| Source Control | GitHub | 애플리케이션 코드 및 Kubernetes manifest 관리 |
| CI / 배포 트리거 | GitHub Actions | Push 이벤트 기반 Workflow 실행, Docker 이미지 빌드, ACR Push 및 AKS Deployment Rolling Update 트리거 |
| Container | Docker | `index.html`을 포함한 Nginx 기반 컨테이너 이미지 생성 |
| Container Registry | Azure Container Registry | 빌드된 Docker 이미지 저장 |
| Kubernetes | Azure Kubernetes Service | 컨테이너 애플리케이션 실행 환경 구성 |
| Workload | Kubernetes Deployment | Pod 배포 및 Rolling Update 관리 |
| Networking | Kubernetes Service LoadBalancer | 외부 Public IP 기반 웹 페이지 접속 구성 |
| GitOps | Argo CD | Kubernetes manifest 기반 Sync / Health 상태 확인 |

## 경험한 핵심 개념

- Dockerfile 기반 컨테이너 이미지 빌드
- GitHub Actions Workflow 구성
- Azure Container Registry에 Docker 이미지 Push
- AKS에서 Deployment, ReplicaSet, Pod가 동작하는 구조
- `kubectl rollout restart`를 통한 Deployment Rolling Update 트리거
- `latest` 태그 기반 배포 방식의 장단점

## 프로젝트를 통해 확인한 내용

이 프로젝트를 통해 GitHub Repository의 코드 변경이 GitHub Actions를 통해 컨테이너 이미지로 빌드되고, ACR에 Push된 뒤, AKS에서 최신 이미지 기반 Pod로 교체되는 흐름을 확인했습니다.

또한 Argo CD를 통해 GitHub Repository의 Kubernetes manifest와 AKS 클러스터 리소스 상태를 비교하고, Application이 `Synced` 및 `Healthy` 상태인지 확인했습니다.

## 마무리

GitHub Actions는 코드 Push 이벤트를 기준으로 Docker 이미지를 빌드하고 Azure Container Registry에 Push하는 CI 역할을 수행했습니다. 또한 현재 구조에서는 `latest` 태그 기반 이미지 반영을 위해 `kubectl rollout restart`를 실행하여 AKS Deployment의 Rolling Update를 트리거했습니다.

Argo CD는 GitHub Repository의 Kubernetes manifest를 기준으로 AKS 리소스 상태를 관리하고, Application의 `Synced` 및 `Healthy` 상태를 확인하는 GitOps 도구로 활용했습니다.
