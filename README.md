# argoCD_basic

### 01 Argo 기본 설치

1. ArgoCD 로컬 설치 arco-cd 디렉토리 이동 후 make upgrade-local 실행

2. 초기 admin 비번 argo namespace secret
   argocd-initial-admin-secret 참조

### 02 Argo declarative-setup

1. argo-declarative-setup 디렉토리 이동
2. AppProject로 프로젝트 생성(k apply -f AppProject.yaml)
   Application 실행(k apply -f Application.yaml)
3. heml 사용하여 배포 시, k apply -f Application-helm.yaml
4. (auto prune이 아닐 시 코드 변경 후) api call sync 사용(계정 토큰 발급 후) - http://localhost:30080/api/v1/applications/sample-yaml/sync

### 03 Argo app-of-apps

- 여러 워크로드 들을 배포 시 기존에는 yaml 파일을 실행하였는데, app-of-apps를 통해서 여러 워크로드를 한번에 배포할 수 있다.

1. app-of-apps 디렉토리 이동 helm template -f values-local.yaml .
   원하는 워크로드의 path 설정
2. app-of-apps 프로젝트 생성
3. 워크로드 추가 후 refresh 진행

### 04 Argo authentication

1. account 생성 - argo-cd디렉토리 values-local.yaml에서 accounts.seobi: apiKey, login 설정 후 make upgrade-local 진행
2. brew install argocd
3. argocd login 127.0.0.1:30080 --username admin --grpc-web / y / secret 비번
4. account 조회 - argocd account list
5. 유저 패스워드 변경 -argocd account update-password --account 유저명
6. 권한 제공 (g, seobi, role:local-user-admin)

### 05 Argo SSO

- dex connector를 사용하여 upstream Identitiy Provider 사용

### 06 Argo CI/CD

- 깃허브에 코드 푸쉬되면 github actions가 트리거되고, 도커이미지 빌드 후 ECR에 push되고,
  cd작동하여 git push --prune으로 config repository 업데이트됨.
  argo는 config repository를 보게되고 config repository에 values 업데이트된 내용이 argocd에 배포됨

1. secret ecr 생성
   k create secret docker-registry ecr-cred \
    --docker-server=${clientId}.dkr.ecr.ap-northeast-2.amazonaws.com \
    --docker-username=AWS --docker-password=$(aws ecr get-login-password --region ap-northeast-2) \
    --namespace=default

2. git hub repo security에 AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY / GH_TOKEN 생성

3. app version 올린 후 app-of-apps 작동

4. 기본 pulling 3분 대기하므로 구동 app의 repo setting 에서 webhook 등록 시 바로 작용한다.
   (http://localhost/api/webhook)

### 06 Argo eks CI/CD

1. argocd를 eks에 배포후 alb를 통해 config repo 연결
2. alb 설치 - alb 디렉토리에서 alb 설치(make upgrade-eks)
3. argocd 배포 - argocd디렉토리에서 make upgrade-eks실행 되면서 aws alb컨트롤러 생성
4. app-of-apps eks 버전으로 프로젝트 생성 namespace(argo)
