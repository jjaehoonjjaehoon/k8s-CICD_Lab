# Improvement Notes

## 현재 구조

현재 프로젝트는 `demo-app:latest` 태그와 `kubectl rollout restart`를 사용하여 AKS에 최신 이미지를 반영하는 구조입니다.

```text
GitHub Push
→ GitHub Actions Workflow 실행
→ Docker 이미지 빌드
→ Azure Container Registry에 latest 태그로 이미지 Push
→ kubectl rollout restart로 Deployment Rolling Update 트리거
→ 새 Pod가 최신 latest 이미지 Pull
→ 새 Pod가 Running / Ready 상태가 된 후 기존 Pod가 점진적으로 교체됨
```

## 현재 구조의 한계

현재 `deployment.yaml`은 다음과 같이 `latest` 태그를 참조합니다.

```yaml
image: jhacr01.azurecr.io/demo-app:latest
```
GitHub Actions가 새 이미지를 ACR에 `latest` 태그로 Push하더라도, Git에 저장된 `deployment.yaml`의 image 값은 변경되지 않습니다.  
따라서 Argo CD 입장에서는 Git manifest 변경사항이 발생하지 않아, 새 이미지 Push만으로는 배포 변경을 명확하게 감지하기 어렵습니다.

이를 보완하기 위해 현재 Workflow에서는 이미지 Push 이후 `kubectl rollout restart deployment demo-app -n demo-app` 명령을 실행합니다.  
이 명령을 통해 AKS Deployment의 Rolling Update를 트리거하고, 새로 생성된 Pod가 ACR에서 최신 `demo-app:latest` 이미지를 Pull하도록 구성했습니다.

즉, 현재 구조는 `latest` 태그와 `rollout restart`를 조합해 최신 이미지를 반영하는 방식입니다.  
다만 GitOps 관점에서는 Git 변경사항을 Argo CD가 감지하여 배포하는 구조라기보다는, GitHub Actions가 AKS에 직접 재시작 명령을 실행해 배포를 트리거하는 구조입니다.


## 개선 방향

향후에는 commit SHA 기반 이미지 태그를 사용하고, GitHub Actions가 `deployment.yaml`의 image 값을 자동으로 업데이트하도록 개선할 수 있습니다.

개선 후 흐름은 다음과 같습니다.

```text
GitHub Push
→ GitHub Actions Workflow 실행
→ Docker 이미지 빌드 with commit SHA tag
→ Azure Container Registry에 commit SHA 태그로 이미지 Push
→ deployment.yaml image tag 자동 업데이트
→ GitHub에 manifest 변경사항 commit / push
→ Argo CD가 Git 변경 감지
→ AKS Sync
```

예시:

```yaml
image: jhacr01.azurecr.io/demo-app:53e26a0
```

## 기대 효과

commit SHA 기반 이미지 태그를 사용하면 다음과 같은 장점이 있습니다.

- 어떤 Git commit이 어떤 Docker 이미지로 배포되었는지 추적 가능
- Git manifest만 보고 현재 배포 버전 확인 가능
- Argo CD가 Git 변경사항을 기준으로 명확하게 Sync 가능
- Rollback 대상 이미지 태그를 명확하게 지정 가능
- GitHub Actions가 AKS에 직접 재시작 명령을 실행하는 구조를 줄일 수 있음

## 정리

운영형 구조로 고도화하려면 `latest` 태그 대신 commit SHA 기반 이미지 태그를 사용하고, GitHub Actions가 Kubernetes manifest를 자동 업데이트하도록 개선하는 것이 더 적합합니다.
