1. Git 리포지토리로 푸시/병합 발생
GitLab CI/CD가 자동으로 파이프라인을 트리거한다.
.gitlab-ci.yml 파일에 정의된 단계와 규칙에 따라 실행됨

2. build(빌드)

3. pulish(도커이미지 파일 생성하여 ECR에 올림)

4. deploy (manifest 파일 교체)
edit set image로 이미지태그 변경, kustomize build 를 통해 
kustomization.yaml 파일이 갱신됨

5. ArgoCD가 동기화해줌





쿠버네티스가 관리하는 컨테이너 안에 ArgoCd가 올라가있는 컨테이너가 있는거면
어디서 볼 수 있나요

*쿠버네티스*
컨테이너 오케스트레이션 도구 

*쿠버네티스 클러스터*
컨트롤 플레인 + 데이터 플레인(노드들의 집합)
- 컨트롤 플레인(마스터 노드 그룹) : 데이터 플레인이 어떻게 동작할지 설정함
- 데이터 플레인 : 여러개의 노드 그룹
- 노드 : kublet과 파드(N개), kube-proxy 로 이루어짐. 각 노드는 컨트롤 플레인에 의해 관리됨
- kublet : ControlPlane과 통신을 담당
- 파드 : 쿠버네티스 최소 단위. 파드 하나 안에서 1개 이상의 컨테이너를 동작시킬 수 있음
컨테이너를 파드 내에 배치하고 노드에서 실행함


*쿠버네티스 리소스*
클러스터 내에서 애플리케이션을 실행하고 관리하기 위한 오브젝트들

1. Pod
쿠버네티스 최소 단위. 파드 하나 안에서 1개 이상의 컨테이너를 동작시킬 수 있음

2. ReplicaSet
파드를 얼마나 동작시킬지 관리하는 오브젝트
클러스터에서 pod가 정확한 종류와 수로 동작하는지 관리
디플로이먼트가 직접 파드를 배포하지 않고 레플리카셋이라는 오브젝트를 생성해 배포한다.
노드의 장애나 네트워크 분리 같은 장애 상황에서 pod를 자동으로 다시 스케줄링함

3. Service
배포한 파드를 외부에 공개하기 위한 구조를 제공
클러스터 내에 파드 여러개를 동작시킨 경우 그 앞단에 로드밸런서를 배치하여 특정 파드를 클러스터 외부로 공개할 수 있다
파드 여러개를 묶어 하나의 DNS 이름으로 접속할 수 있음
해당 서비스를 구성하는 파드 중 정상적으로 동작하는 파드에만 요청을 할당할 수 있음


4. 네임스페이스
쿠버네티스에서는 클러스터 하나를 네임스페이스라는 논리적 구획으로 구분하여 관리
컨테이너를 동작시키기 위한 리소스는 아니지만 컨테이너가 동작하는 클러스터를 논리적으로 사용하기 위한 리소스


5. Deployment
배포 이력 관리, 레플리카셋 수 변경, 롤백 등의 역할을 함
디플로이먼트는 레플리카셋을 관리한다.
배포전략속성
- RollingUpdate : 새 버전을 배포하면서 하나씩 늘려가고 예전걸 하나씩 줄여나감. (무중단 배포)
- Recreate : 기존파드 모두 삭제 후 새 파드 생성(다운타임 발생)
롤아웃 : 모든 배포 및 업데이트

6. DaemonSet
클러스터 내부에서 공통으로 사용하는 기능을 구현하기 위한 리소스
전체 노드에 특정 파드를 실행할 때 사용함 
모니터링, 로그 수집용 애플리케이션과 같은 클러스터 전체에 항상 실행해두어야 하는 파드에 사용한다.

7. StatefulSet
쿠버네티스는 파드 외부에 볼륨 형태로 데이터를 저장한 후 파드가 재시작되더라도
그때까지 사용했던 볼륨을 계속 사용할 수 있도록 함.
스테이트풀셋이 이 동작을 지원한다.
Stateful : 파드별로 고유의 상태를 가짐(독자성 유지)
Stateless : 고유의 상태를 갖지 않고, 모든 파드가 같은 상태를 가짐

8. Job 
일정한 처리를 수행하고 완료하는 태스크를 실행하기 위한 것
예) job을 하나 생성 -> 파드 하나를 실행 -> 처리 완료
 - > 실패 또는 삭제된 경우 -> 새로운 파드 기동

9. CronJob
시작 간격 또는 시간 간격을 설정하여 프로그램을 동작시키기 위한 구조

10. Configmap
파드가 동작할 때 configmap에서 값을 읽어 들여 환경변수 등에 설정함

11. Secret
파드가 동작할 때 시크릿에서 값을 읽어 들여 환경변수 등에 설정함
설정된 값은 시크릿으로 등록될 때 base62로 인코딩되어 등록된다.

12. Ingress
쿠버네티스 클러스터로 접근하는 입구를 만들기 위한 리소스




*manifest 파일*
쿠버네티스 오브젝트 설정 파일.
쿠버네티스는 manifest 파일에 기재된 내용에 따라 파드를 생성함
manifest 파일은 리소스 단위로 작성하며 한 파일에 합쳐서 작성해도 됨
선언형 설정 : 원하는 상태를 설정에 작성해 서비스로 제출하면 원하는 상태의 서비스로 실제 상태가 될 수 있도록 조치하는 설정 형태
 => 관리가 용이하고 자가 치유 조치의 기본
명령형 설정 : 일련의 행동을 통해 변경사항을 만듦.

<pre><code class="yaml">
apiVersion: v1
kind: Service
metadata:
  name: backend-app-svc
  namespace: dominos-service-backend
spec:
  type: NodePort       -------서비스의 리소스 타입 중 하나. 각 노드에서 해당 서비스에 접속하기 위한 포트를 열고 클러스터 외부에서 접속 가능하도록 한다. 
  ports:
    - port: 80
      targetPort: 8081  ---------- 파드쪽 포트 번호
  selector:
    app: backend-app    ---------- 이 서비스가 대상으로 하는 파드를 선택하기 위한 셀렉터 정의. 이거는 patch로 따로 정의 안 하나?
</code></pre>


같은 설정을 여러 환경에 배포할 때 
일부 설정만 조금씩 다르게 배포 하고 싶은 경우
1. 템플릿 방식
Helm이라는 도구가 있음
Deployment YAML 파일을 만들때, 환경에 따라 변경되는 부분을 변수로 처리한 템플릿을 만든 후에, 환경별로 변수 값을 채워 넣는 방식

2. 상속 방식(Kustomize)
공통적으로 적용할 base yaml을 생성하고, 변경할 부분만 상속받은 yaml에서 구현하는 방식


kustomize 
yaml파일을 변경하지 않고 리소스를 생성하는 도구
kustomization.yaml : kustomize가 실행될 때 어떤 필드를 재정의 것인가

<pre><code class="yaml">
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:  ----------------------------kustomize를 적용할 쿠버네티스 yaml 파일
- 04.Deployment.yaml
- 05.Service.yaml
- 06.Ingress.yaml
</code></pre>


디렉토리 구조
<pre>
~/someApp
├── base -----------------------공통 yaml 리소스 파일
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays
    ├── dev
    │   ├── deployment-patch.json
    │   ├── service-patch.json
    │   └── kustomization.yaml   ---------base를 참조하고 수정하는 내용을 patch 항목으로 정의하고 있음
    └── prod
        ├── deployment-patch.json
        ├── service-patch.json
        └── kustomization.yaml
</pre>
kustomize build를 하면 patch를 적용하여 yaml이 생성됨
어디에 ?



*ArgoCD*
Application : manifest가 저장된 Git과 동기화시킬 쿠버네티스 클러스터 정보를 저장


- 배포?
1. ArgoCD가 repository의 manifest 변경 감지, 변경된 리소스 내용 파악
2. Kubenetest API 호출 => 리소스 조회, 생성, 수정 등 

- 배포는 deployment를 이용
Deployment 매니페스트 파일 적용 => Deployment 오브젝트 생성 => 레플리카셋 생성 => 파드 생성
디플로이먼트 설정변경(컨테이너 이미지 업데이트 포함) => 레플리카셋 생성 => 파드 생성


API Gateway?
메서드 요청 -> 통합 요청 -> HTTP 통합 -> 응답 -> 메서드 응답 

