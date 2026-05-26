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
## 사용 기술

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
1. 개발자가 `index.html`을 수정합니다.
2. 변경사항을 GitHub Repository에 commit / push합니다.
3. GitHub Actions Workflow가 실행됩니다.
4. Dockerfile을 기준으로 Docker 이미지를 빌드합니다.
5. 빌드된 이미지를 Azure Container Registry에 `demo-app:latest` 태그로 Push합니다.
6. GitHub Actions에서 `kubectl rollout restart deployment demo-app -n demo-app` 명령을 실행합니다.
7. Kubernetes Deployment의 Rolling Update가 트리거됩니다.
8. 새 Pod가 생성되고 ACR에서 최신 `demo-app:latest` 이미지를 Pull합니다.
9. 새 Pod가 Running / Ready 상태가 되면 기존 Pod가 점진적으로 교체됩니다.
10. Service type LoadBalancer를 통해 외부 Public IP에서 변경된 웹 페이지를 확인합니다.
11. Argo CD에서 Application의 `Synced` 및 `Healthy` 상태를 확인합니다.

## Repository 구조

| File                        | Description                                  |
| --------------------------- | -------------------------------------------- |
| `index.html`                | 실제 웹 페이지 내용                                  |
| `Dockerfile`                | Nginx 기반 Docker 이미지 빌드 정의                    |
| `.github/workflows/ci.yaml` | GitHub Actions CI/CD Workflow                |
| `deployment.yaml`           | AKS에 배포할 Kubernetes Deployment 정의            |
| `service-lb.yaml`           | 외부 접속을 위한 Kubernetes LoadBalancer Service 정의 |
| `README.md`                 | 프로젝트 설명 문서                                   |

## 배포 전략

현재 `deployment.yaml`은 다음과 같이 `latest` 태그를 참조합니다.

```yaml
image: jhacr01.azurecr.io/demo-app:latest
```

`latest` 태그는 ACR에서 가장 최근에 Push된 이미지를 가리킬 수 있지만, Kubernetes manifest 파일의 값 자체는 계속 동일하게 유지됩니다.

즉, 새 Docker 이미지가 ACR에 Push되어도 `deployment.yaml`의 내용은 다음과 같이 변하지 않습니다.

```yaml
image: jhacr01.azurecr.io/demo-app:latest
```

이 경우 Kubernetes 또는 Argo CD 입장에서는 manifest의 image 값이 변경되지 않았기 때문에, 기존 Pod가 자동으로 새 이미지를 Pull하지 않을 수 있습니다.

이를 보완하기 위해 GitHub Actions Workflow에서 다음 명령을 실행하도록 구성했습니다.

```bash
kubectl rollout restart deployment demo-app -n demo-app
```

이 명령은 Deployment의 Rolling Update를 트리거합니다.  
새 Pod는 ACR에서 최신 `demo-app:latest` 이미지를 Pull하고, Running / Ready 상태가 된 후 기존 Pod가 점진적으로 교체됩니다.

따라서 현재 구조는 다음과 같이 정리할 수 있습니다.

```text
GitHub Push
-> GitHub Action sWorkflow 실행
-> Docker 이미지 빌드
-> ACR에 latest 태그로 이미지 Push
-> kubectl rollout restart 실행
-> 새 Pod가 최신 latest 이미지 Pull
-> 새 Pod가 정상 상태가 된 후 기존 Pod가 점진적으로 교체됨
```

이 방식은 실습 환경에서 단순하고 빠르게 CI/CD 흐름을 확인하기에 적합합니다.  
다만 GitOps 관점에서는 manifest의 image tag가 항상 `latest`로 유지되기 때문에, 어떤 이미지 버전이 실제로 배포되었는지 추적하기 어렵다는 한계가 있습니다.

## 검증 결과

본 프로젝트에서는 GitHub Push 이후 CI/CD 파이프라인이 정상적으로 동작하는지 다음 항목을 기준으로 검증했습니다.

| 검증 항목 | 확인 내용 | 결과 |
|---|---|---|
| GitHub Actions 실행 | Push 후 Workflow가 자동 실행되는지 확인 | 정상 |
| Docker 이미지 빌드 | GitHub Actions에서 Dockerfile 기반 이미지 빌드가 성공하는지 확인 | 정상 |
| ACR Push | 빌드된 `latest` 이미지가 Azure Container Registry에 Push되는지 확인 | 정상 |
| AKS Deployment 반영 | `kubectl rollout restart` 이후 새 Pod가 생성되고 Running 상태가 되는지 확인 | 정상 |
| LoadBalancer 접속 | `demo-app-lb`의 External IP로 접속하여 수정된 `index.html` 내용이 반영되는지 확인 | 정상 |
| Argo CD 상태 | Argo CD Application이 `Synced` 및 `Healthy` 상태인지 확인 | 정상 |


## 한계 및 향후 개선 사항

현재 방식은 `latest` 태그와 `kubectl rollout restart`를 사용하여 최신 이미지를 AKS에 반영합니다.

| 현재 구조 | 한계 | 개선 방향 |
|---|---|---|
| `latest` 태그 사용 | manifest 값이 변경되지 않아 배포 버전 추적이 어려움 | commit SHA 기반 이미지 태그 사용 |
| `kubectl rollout restart` 사용 | GitHub Actions가 AKS에 직접 명령을 실행함 | Argo CD가 Git 변경사항을 기준으로 Sync하도록 개선 |
| LoadBalancer Service 사용 | 단순 외부 노출 구조 | Ingress Controller 또는 Application Gateway 연동 가능 |

향후에는 Docker 이미지를 commit SHA 기반으로 태깅하고, GitHub Actions가 `deployment.yaml`의 image 값을 자동으로 업데이트하도록 개선할 수 있습니다.

