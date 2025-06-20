# Model
## azure-aks-istio-servicemesh-monitoring-lab-20250620
https://labs.msaez.io/#/courses/cna-full/2c7ffd60-3a9c-11f0-833f-b38345d437ae/deploy-my-app-2024
- 이 실습은 Azure Kubernetes Service (AKS) 환경에 Istio 서비스 메시를 설치하고, Kiali, Grafana, Prometheus, Loki와 같은 모니터링 대시보드를 연동하여 서비스 메시의 가시성을 확보하는 방법을 다룹니다.
- 또한, NGINX Ingress Controller를 통해 이들 대시보드를 외부로 노출하고, 애플리케이션 Pod에 Istio 사이드카를 주입하는 다양한 방법을 실습합니다.

## 사전 준비
Azure 계정 및 구독, Gitpod 워크스페이스, Spring Boot 애플리케이션 코드

![스크린샷 2025-06-20 151048](https://github.com/user-attachments/assets/6b9da586-6ab5-46ba-8b62-a8d3534a6fa4)
![스크린샷 2025-06-20 151135](https://github.com/user-attachments/assets/108ab9f8-569c-47b2-a5f1-c8e966a8c9e6)
![스크린샷 2025-06-20 153734](https://github.com/user-attachments/assets/a311c7bb-7ab1-4bfc-b399-0b20fae931b0)
![스크린샷 2025-06-20 153905](https://github.com/user-attachments/assets/4a6be501-26af-4afa-a454-e96f48fff5a3)
![스크린샷 2025-06-20 154627](https://github.com/user-attachments/assets/be1a3690-8220-409c-9c56-22849d5307cf)
![스크린샷 2025-06-20 155526](https://github.com/user-attachments/assets/3384355c-7f51-425e-b63e-fb189a6b49b6)
![스크린샷 2025-06-20 160801](https://github.com/user-attachments/assets/0fc02dd6-3260-4ad8-84ac-6cd3c681b365)
![스크린샷 2025-06-20 160921](https://github.com/user-attachments/assets/b498f470-6194-49e9-b372-eaa3113270cb)
![스크린샷 2025-06-20 160931](https://github.com/user-attachments/assets/a189d315-aa7d-4ee7-9ce7-09de045fccc9)
![스크린샷 2025-06-20 161111](https://github.com/user-attachments/assets/ea63982c-ae35-47f9-8298-8bc3822fd9b7)
![스크린샷 2025-06-20 161628](https://github.com/user-attachments/assets/0d475b74-dee5-4296-80eb-4e5b493bb2f9)
![스크린샷 2025-06-20 163931](https://github.com/user-attachments/assets/80f5a72b-3121-4268-9821-0ce878c38272)

---

## 실습 단계별 상세 설명
- 이 섹션은 Istio 서비스 메시 설치 및 설정에 필요한 터미널 명령어를 순서대로 제공합니다.

0. 초기 환경 설정
```
# Gitpod 환경 초기화 스크립트 실행 (init.sh 스크립트가 존재한다고 가정)
./init.sh
```
```
# Kubernetes 클러스터 버전 확인 (Istio 호환성 확인용)
kubectl version

# Istio 특정 버전 다운로드 및 압축 해제
export ISTIO_VERSION=1.24.6
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=$ISTIO_VERSION TARGET_ARCH=x86_64 sh -

# 다운로드된 Istio 패키지 디렉토리로 이동
cd istio-$ISTIO_VERSION

# istioctl 클라이언트 툴을 PATH 환경 변수에 추가
export PATH=$PWD/bin:$PATH
```
1. Istio 서비스 메시 설치
```
# Istio를 'demo' 프로파일로 클러스터에 설치
# 'demo' 프로파일은 모든 주요 기능과 모니터링 도구가 포함된 학습 및 테스트용 구성입니다.
istioctl install --set profile=demo --set hub=gcr.io/istio-release

# Istio 관련 네임스페이스 확인
kubectl get namespace

# istio-system 네임스페이스 내의 모든 Istio 컴포넌트 상태 확인
kubectl get all -n istio-system
```
2. Istio 애드온 대시보드 설치
```
# 기존 loki.yaml 파일 백업 (옵션)
mv samples/addons/loki.yaml samples/addons/loki.yaml.old

# 외부에서 loki.yaml 파일 다운로드 (msa-school에서 제공하는 커스텀 loki 설정)
curl -o samples/addons/loki.yaml https://raw.githubusercontent.com/msa-school/Lab-required-Materials/main/Ops/loki.yaml

# Istio 애드온 대시보드들 (Kiali, Grafana, Prometheus, Loki 등) 배포
# 이 명령은 samples/addons 디렉토리 내의 모든 YAML 파일을 적용합니다.
kubectl apply -f samples/addons
```
3. Istio 설치 및 애드온 최종 확인
```
# istio-system 네임스페이스 내의 모든 서비스 상태 확인
kubectl get svc -n istio-system

# istio-system 네임스페이스 내의 모든 리소스 상태 확인
kubectl get all -n istio-system
```
4. NGINX Ingress Controller 설치
```
# Helm 3.x 설치 (이미 설치되어 있다면 건너뛰기)
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh

# Helm Repository 추가 및 업데이트
helm repo add stable https://charts.helm.sh/stable
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# NGINX Ingress Controller를 위한 네임스페이스 생성
kubectl create namespace ingress-basic

# Nginx Ingress Controller를 Azure 환경에 맞게 설치
# controller.service.annotations를 통해 Azure Load Balancer 헬스 체크 경로를 설정합니다.
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-basic \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz
```
5. Istio 대시보드 외부 라우팅 룰 (Ingress) 설정
```
# Kiali, Grafana, Prometheus, Loki 대시보드를 위한 Ingress 리소스 정의 및 배포
# 이 Ingress는 NGINX Ingress Controller를 통해 서비스들을 외부로 노출합니다.
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: "Ingress"
metadata:
  name: "shopping-ingress"
  namespace: istio-system # Istio 애드온들이 있는 네임스페이스
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  ingressClassName: nginx # NGINX Ingress Controller 사용 명시
  rules:
    - host: "" # 모든 호스트명 허용 (IP로 접속 시)
      http:
        paths:
          - path: /kiali
            pathType: Prefix
            backend:
              service:
                name: kiali
                port:
                  number: 20001
          - path: /grafana
            pathType: Prefix
            backend:
              service:
                name: grafana
                port:
                  number: 3000
          - path: /prometheus
            pathType: Prefix
            backend:
              service:
                name: prometheus
                port:
                  number: 9090
          - path: /loki
            pathType: Prefix
            backend:
              service:
                name: loki
                port:
                  number: 3100
EOF
```
6. 서비스 메시 모니터링 대시보드 접속
```
# Istio 대시보드들을 노출하는 Ingress (shopping-ingress)의 외부 IP 주소 확인
kubectl get ingress -n istio-system

# 웹 브라우저에서 Kiali 대시보드에 접속
# (위 kubectl get ingress 결과에서 ADDRESS를 확인하여 [Ingress-LoadBalancer-IP]를 대체)
# 예시: http://20.249.190.246/kiali
# Kiali 로그인 계정이 필요한 경우, 일반적으로 admin / admin 입력
```
7. 분산 추적 시스템(Tracing) 접속 준비
```
# Jaeger (Tracing 시스템) 서비스를 ClusterIP에서 LoadBalancer 타입으로 변경하여 외부 노출
# (이렇게 하면 Jaeger 대시보드에도 외부 IP가 할당됩니다.)
kubectl patch svc tracing -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'

# 변경된 서비스 상태 확인 및 Jaeger 대시보드의 EXTERNAL-IP 확인
kubectl get service -n istio-system
```
8. Istio Sidecar Injection 방법 학습 (수동 주입)
```
# NGINX Deployment YAML 파일 생성 (이 파일을 현재 Istio 디렉토리 내에 저장)
# 예시 파일명: deployment.yaml
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-nginx
  template:
    metadata:
      labels:
        app: hello-nginx
    spec:
      containers:
        - name: hello-nginx
          image: nginx:latest
          ports:
            - containerPort: 80
EOF
```
```
# istioctl kube-inject 명령어를 사용하여 Sidecar가 주입된 YAML 내용 확인 (파일로 저장)
# 이 output.yaml 파일에는 NGINX 컨테이너 외에 Istio 사이드카 컨테이너가 추가되어 있습니다.
istioctl kube-inject -f deployment.yaml > output.yaml

# Sidecar가 주입된 Deployment를 클러스터에 바로 적용
kubectl apply -f <(istioctl kube-inject -f deployment.yaml)
```
9. Istio Sidecar Injection 방법 학습 (네임스페이스 자동 주입)
```
# Istio 사이드카 주입을 테스트할 새로운 네임스페이스 'tutorial' 생성
kubectl create ns tutorial

# 생성된 네임스페이스 목록 확인
kubectl get ns

# 'tutorial' 네임스페이스에 Istio 자동 사이드카 주입을 활성화하는 레이블 추가
kubectl label namespace tutorial istio-injection=enabled

# Sidecar가 주입되지 않은 'deployment.yaml'을 'tutorial' 네임스페이스에 배포
# (이 네임스페이스에 레이블이 있으므로 자동으로 사이드카가 주입됩니다.)
kubectl apply -f deployment.yaml -n tutorial

# 'tutorial' 네임스페이스의 모든 리소스 상태 확인 (Pod에 2/2 Ready 상태 확인 - nginx 컨테이너 + istio-proxy 컨테이너)
kubectl get all -n tutorial

# 특정 Pod의 상세 정보 확인 (Sidecar 컨테이너가 주입되었는지 확인)
# $(kubectl get pod -l app=hello-nginx -n tutorial -o jsonpath='{.items[0].metadata.name}') 는 동적으로 Pod 이름을 가져옵니다.
kubectl describe pod/$(kubectl get pod -l app=hello-nginx -n tutorial -o jsonpath='{.items[0].metadata.name}') -n tutorial
```
10. Istio Sidecar Injection 방법 학습 (Pod 레이블을 통한 개별 주입)
```
# Deployment YAML 파일에 직접 'sidecar.istio.io/inject: "true"' 레이블 추가 예시
# (이 방식은 네임스페이스 레벨의 자동 인젝션 설정과 무관하게 Pod에 Sidecar를 주입합니다.)
#
# apiVersion: apps/v1
# kind: Deployment
# metadata:
#   name: hello-nginx
# spec:
#   replicas: 1
#   selector:
#     matchLabels:
#       app: hello-nginx
#   template:
#     metadata:
#       labels:
#         app: hello-nginx
#         sidecar.istio.io/inject: "true" # 이 Pod 레이블을 추가
#     spec:
#       containers:
#         - name: hello-nginx
#           image: nginx:latest
#           ports:
#             - containerPort: 80
```
11. Istio 서비스 메시 및 애드온 제거 (선택 사항)
```
# Istio 애드온 대시보드들 삭제
# (Istio 패키지 폴더 (istio-$ISTIO_VERSION) 내에서 실행)
kubectl delete -f samples/addons

# Istio 컨트롤 플레인 및 게이트웨이 등 핵심 컴포넌트 제거
# (Istio 패키지 폴더 (istio-$ISTIO_VERSION) 내에서 실행)
istioctl uninstall --purge

# istio-system 네임스페이스 삭제 (선택 사항, 필요 없으면 제거)
kubectl delete namespace istio-system

# 'tutorial' 네임스페이스 삭제 (Sidecar Injection 테스트용 네임스페이스)
kubectl delete namespace tutorial
```
