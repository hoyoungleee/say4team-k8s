# Kubernetes MSA 배포 인프라 구성 (Helm + Ingress)

## 개요

본 프로젝트는 Kubernetes 기반의 마이크로서비스 아키텍처(MSA)를 Helm과 Ingress Controller를 통해 효율적으로 배포하고 관리하기 위한 인프라 템플릿입니다. 각 서비스의 배포 자동화, 도메인 기반 트래픽 라우팅, HTTPS 인증서 적용 등을 목표로 구성되어 있습니다.

---

## 사용 기술 및 도입 이유

| 기술            | 설명 | 도입 목적 |
|-----------------|------|-----------|
| **Kubernetes**  | 컨테이너 오케스트레이션 플랫폼 | MSA 구조에서 각 서비스의 배포, 스케일링, 복구 자동화를 위함 |
| **Helm**        | Kubernetes 패키지 매니저 | 반복적인 리소스 배포를 템플릿화하여 관리 효율성을 높이기 위함 |
| **Ingress**     | 클러스터 외부로부터 내부 서비스로의 HTTP/S 요청 라우팅 제어 | 하나의 진입점을 통해 서비스 접근을 제어하고 도메인 기반 트래픽 분배를 위해 |
| **TLS/HTTPS 인증서** | `cert-manager`와의 연동 가능성 고려 | 서비스 보안 강화를 위한 HTTPS 적용 준비 |

---

## 디렉토리 구조

```
📁 ingress/
    ├── ingress.yaml           # 기본 Ingress 리소스 (도메인 라우팅)
    └── ingress_cert.yaml      # HTTPS 인증서 적용용 Ingress

📁 msa-chart/
    ├── Chart.yaml             # Helm 차트 메타 정보
    ├── Chart.lock             # 의존성 잠금 파일
    ├── .helmignore            # Helm 빌드시 제외할 파일 목록
```

---

## 구성 요소 설명

### 1. Ingress 설정

- `ingress.yaml`: 도메인별 트래픽을 해당 서비스로 라우팅
- `ingress_cert.yaml`: TLS 비밀키를 기반으로 HTTPS 트래픽을 처리

Ingress Controller(NGINX, ALB 등)와 함께 사용되어 외부 트래픽을 적절히 분배합니다.

### 2. Helm Chart (msa-chart)

- `Chart.yaml`: Helm 패키지 정의 (name, version 등)
- 내부적으로 `Deployment`, `Service`, `ConfigMap` 등의 리소스를 템플릿화하여 통일된 배포 방식을 제공합니다.
- `.helmignore`: 빌드시 불필요한 파일 제외

---

## 배포 절차 예시

```bash
# 1. Ingress 설정 적용
kubectl apply -f ingress/ingress.yaml
kubectl apply -f ingress/ingress_cert.yaml

# 2. Helm Chart 배포
helm install msa-service ./msa-chart
```

> 서비스 환경에 따라 `values.yaml` 파일을 커스터마이징하여 배포 대상, 포트, 환경 설정을 분리할 수 있습니다.

---

## 인증서 적용

- `ingress_cert.yaml`은 사전 발급된 TLS 인증서(예: Let's Encrypt, AWS ACM 등)의 Secret을 참조합니다.
- 추후 `cert-manager`를 활용한 자동 인증서 발급 및 갱신 기능을 연동할 수 있습니다.

예시:

```yaml
tls:
  - hosts:
      - your-domain.com
    secretName: tls-secret
```

---

## 기대 효과

- 마이크로서비스별 독립 배포 및 관리 가능
- 템플릿화된 배포 구조를 통한 일관성 확보
- 도메인 기반 요청 제어로 외부 노출 서비스 제한 가능
- HTTPS 인증서 적용으로 보안성 강화

---

## 요구 사항

- Kubernetes 1.19 이상 클러스터
- Helm 3.x 이상
- Ingress Controller 설치 (예: nginx-ingress, AWS ALB Ingress Controller)
- 선택: cert-manager (HTTPS 인증서 자동화)

---
