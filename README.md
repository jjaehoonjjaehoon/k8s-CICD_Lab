# AKS CI/CD Lab with GitHub Actions and Argo CD

## Overview

이 프로젝트는  Azure Container Registry, Azure Kubernetes Service, GitHub Actions(CI), Argo CD를 활용하여 Kubernetes 기반 CI/CD 흐름을 실습한 개인 프로젝트입니다.

정적 웹 페이지인 `index.html`을 Nginx 기반 Docker 이미지로 빌드하고, Azure Container Registry에 Push한 뒤, AKS 클러스터의 Deployment에 반영하는 구조로 구성했습니다.

GitHub Actions는 Docker 이미지 빌드, ACR Push, AKS Deployment 재시작을 담당하며, Argo CD는 GitHub Repository의 Kubernetes manifest를 기준으로 애플리케이션 리소스 상태를 관리하고 Sync / Health 상태를 확인하는 데 활용했습니다.

## Architecture

```text
Developer
  ↓
GitHub Repository
  ↓
GitHub Actions
  ↓
Docker Build
  ↓
Azure Container Registry
  ↓
AKS Deployment Restart
  ↓
AKS Pod
  ↓
Kubernetes LoadBalancer Service
  ↓
External User Access

```
## Tech Stack

| 구분                    | 사용 기술                    | 활용 내용                                               |
| --------------------- | ------------------------ | --------------------------------------------------- |
| Source Control        | GitHub                   | 애플리케이션 코드 및 Kubernetes manifest 관리                  |
| CI                    | GitHub Actions           | Docker 이미지 빌드, ACR Push, AKS Deployment 재시작 자동화     |
| Container Registry    | Azure Container Registry | 빌드된 Docker 이미지 저장                                   |
| Kubernetes            | Azure Kubernetes Service | 컨테이너 애플리케이션 실행 환경                                   |
| GitOps / CD Tool      | Argo CD                  | Kubernetes manifest 기반 리소스 상태 관리 및 Sync / Health 확인 |
| Container             | Docker                   | Nginx 기반 정적 웹 페이지 이미지 생성                            |
| Web Server            | Nginx                    | `index.html` 정적 페이지 제공                              |
| Kubernetes Workload   | Deployment               | Pod 배포 및 Rolling Update 관리                          |
| Kubernetes Networking | Service LoadBalancer     | 외부 접속을 위한 Public IP 제공                              |


## CI/CD Flow
1. 개발자가 index.html을 수정합니다.
2. 변경사항을 GitHub Repository에 commit / push합니다.
3. GitHub Actions Workflow가 실행됩니다.
4. Dockerfile을 기준으로 Docker 이미지를 빌드합니다.
5. 빌드된 이미지를 Azure Container Registry에 latest 태그로 Push합니다.
6. GitHub Actions가 AKS Deployment에 대해 kubectl rollout restart를 수행합니다.
7. 새 Pod는 ACR에서 `latest` 이미지를 Pull한 뒤, 이후 기존 Pod가 점진적으로 교체됩니다.
8. Service type LoadBalancer를 통해 외부 Public IP에서 변경된 웹 페이지를 확인합니다.
9. Argo CD에서 Application의 Synced 및 Healthy 상태를 확인합니다.
