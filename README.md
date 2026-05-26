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
