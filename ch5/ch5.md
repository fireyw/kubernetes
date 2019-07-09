## 쿠버네티스
* 쿠버네티스란
    * 컨테이너 운영 자동화를 위한 컨테이너 오케스트레이션 도구
    * 가장 큰 특징은 다양한 리소스를 조합해 유연한 어플리케이션 구축
    * 도커의 부성과 쿠버네티스 탄생
        * 빠른 속도로 컨테이너를 실행하고 파기 할 수 있었지만 배포 및 컨테이너 배치 전략, 스케일 인 아웃 등 몇몇 문제발생
        * 사용이 확산되면서 여러가지 오케스트레이션 도구 발표
        * 구글이 오픈소스로 공개한 쿠버네티스가 제일 부각됨
    * 구글 클라우드플랫폼과 MS AZURE, AWS 까지 쿠버네티스 서비스 제공
    * 쿠버네티스 역할
        * 스웜: 여러대의 호스트를 묶어 기초적인 오케스트레이션 기능 제공
        * 쿠버네티스: 스웜보다 충실한 기능을 갖춘 컨테이너 오케스트레이션 시스템이자 도커를 비롯한 여러가지 컨테이너 런타임을 다룰 수 있다
        
* 컨테이너 오케스트레이션
    * 일반적으로 애플리케이션을 실행하기 위해서는 네트워킹 수준에서 정리가 필요한 개별적인 컨테이너로 구성이 되는데 이 컨테이너를 정리하는 과정이 컨테이너 오케스트레이션
    * 필요한 이유
        * 변수를 통제하고 안정적인 서비스를 목표로 하는 서비스 운영단계에서 단순한 방법이 아닌 체계적으로 관리  
          좀더 쉽게 설명하면 host 머신 한대라면 그냥 docker run 을 하던 docker-compose 를 하던 그냥 container 를 실행시켜도 크게 상관이 없지만 사용자가 점점 많아지면 하나의 host에서 모든 container 를 실행할수 없습니다.  
          host machine의 리소스를 생각하여 container 분배 등을 처리 해야하는데 이런 문제를 해결해주는게 바로 컨테이너 오케스트레이션 툴입니다
    * 주 업무
        1. 컨테이너형 애플리케이션의 배포
        2. 컨테이너 그룹에 대한 로드밸런싱
        3. 스케일링/오토스케일링
        4. 컨테이너 장애 복구
        5. 컨테이너 그룹간 격리/연결        
* 쿠버네티스의 주요 개념
    * 리소스: app를 구성하는 부품과 같은 것으로 노드, 네임스페이드, 파드 등을 가르킴
        * 노드: 컨네이너가 배치되는 서버
        * 네임스페이스: 쿠버네티스 클러스터 안의 가상 클러스터
        * 파드: 컨테이너 집합 중 가장 작은 단위
        * 레플리카세트: 같은 스펙을 갖는 파드를 여러개 생성하고 관리
        * 디플로이먼트: 레플리카 세트의 리버전을 관리
        * 서비스: 파드의 집합에 접근하기 위한 경로 설정
        * 인그레스: 서비스를 쿠버네티스 클러스 외부로 노출
        * 컨피그맵: 설정정보를 정의하고 파드에 전달
        
* 쿠버네티스 클러스와 노드
    * 클러스터는 구버네티스 여러 리소스를 관리하기 위한 집합체
    * 노드
        * 쿠버네티스 리소스중 가장 큰 개념으로 등록된 도커 호스트를 말하며 컨테이너가 배치되는 대상
        * 노드의 수와 사양에 따라 배치할 수 있는 컨테이너 수가 결정된다
        ~~~
        root@parallels-Parallels-Virtual-Platform:~# kubectl get nodes
        NAME                                   STATUS     ROLES    AGE    VERSION
        parallels-parallels-virtual-platform   NotReady   master   123m   v1.15.0
        ~~~
    * 마스터 노드 : 아래 컴포넌트를 통해 클러스터가 동작한다
        * kube-apiserver: kubectl로 부터 리소스 조작 지시를 받는다
        * etcd: 고가용성을 갖춘 분산 키-값 스토어로 클러스터의 백킹 스토어로 쓰임
        * kube-scheduler: 노드를 모니터링하고 컨테이너가 배치할 적절한 노드 선택
        * kube-controller-manager: 리소스를 제어하는 컨트롤러 실행
        
* 네임스페이스
    * 클러스터 안에 가상 클러스터
    ~~~
    root@parallels-Parallels-Virtual-Platform:~# kubectl get nodes
    NAME                                   STATUS     ROLES    AGE    VERSION
    parallels-parallels-virtual-platform   NotReady   master   123m   v1.15.0
    ~~~
    * 개발팀이 일정 규모 이상일때 유용
    
* 파드
    * 컨테이너가 모인 집합체로 최소 하나이상의 컨테이너로 이루어짐
    * 컨테이너간에 강한 결함이 필요할 경우 파드로 묶어 일괄 배포한다
    * 같은 파드를 여러 노드에 배치할 수 도 있고 한 노드에 여러개 배치할 수 있다.
    * 파드 내 컨테이너를 분리해서 노드에 넣을 순 없다.
    * 마스터 노드는 관리용 컴포넌트만 담긴 파드만 배포된 노드다. app에서 사용되는 파드는 배포 할 수 없다
    
    ~~~
    # simple-pop.yaml
    apiVersion: v1
    kind: Pod            # 쿠버네티스 리소스의 유형을 지정
    metadata:
      name: simple-echo  # 이 리소스의 이름
    spec:                # 리소스를 정의하기 위한 속성
      containers:
      - name: nginx      # 컨테이너의 이름
        image: gihyodocker/nginx:latest # 도커허브에 저장된 이미지 태그 
        env:             * 환경 변수
        - name: BACKEND_HOST
          value: localhost:8080
        ports:
        - containerPort: 80 
      - name: echo
        image: gihyodocker/echo:latest
        ports:
        - containerPort: 8080
    ~~~                   
    
    ~~~
    root@parallels-Parallels-Virtual-Platform:~/test# kubectl apply -f simple-pod.yaml
    pod/simple-echo created
    ~~~
    * 파드와 파드안에 든 컨테이너 주소
        * 파드는 각각 고유 ip주소가 할당된다 
        * 할달된 ip는 해당파드에 속하는 모든 컨테이너가 공유함으로 컨테이너간의 통신이 가능하다
        * 파드는 사실 컨테이너를 담은 가상머신과 마찬가지

* 쿠버네티스 명령어 관련 내용
    * kubeadm 
        * the command to bootstrap the cluster  
        * 쿠버네티스 클러스터 생성을 위한 도구 
        * Pod별로 ip 1개를 제공하는 네트워크를 생성
        * 첫노드에는 kubeadm init, 나머지 노드에는 kubeadm join 하여 클러스투 구성
    * kubelet 
        * the component that runs on all of the machines in your cluster and does things like starting pods and containers
    	* 각 노드마다 구동되는 에이전트로 쿠버네티스 마스터와 통신
        * Pod들과 Node의 상태를 클러스터에 보고한다.
    * kubectl 
        * the command line util to talk to your cluster
        * 쿠버네티스 클러스터에 명령을 내리는 CLI
        * 쿠버네티스 API서버와 통신하는 커맨드 도구
    	* 쿠버네티스 객체를 생성 검사 업데이트 할 수 있다  
    		  ex)apply : 쿠버네티스 리소스를 정의하는 파일을 통해 app 관리. 실행하여 클러스터 리소스를 생성하고 업데이트        
              
* 레플리카세트
    * 규모있는 애플리케이션 구축의 경우 같은 파드를 여러개 생성해야 하는데 이때 사용하는게 레플리카세트다
    * 똑같은 정의를 갖는 파드를 여러개 생성하고 관리하는 리소스
    
    ~~~
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
        name: echo
        labels:
            app:echo
    spec:
        replicas:3
        selector:
            machLabels:
                app: echo
    ~~~                 
    * 레플리카세트를 조작해 파드의 수를 줄이면 그만큼 삭제된다
    * 생성한 레플리카는 매니페스트 파일을 사용해 삭제가능하다
    ~~~
    kubectl apply -f simple-replicaset.yaml
    kubectl delete -f simple-replicaset.yaml
    ~~~
    
* 디플로이먼트
    * 레플리카세트보다 상위에 있는 리소스로 애플리케이션 배포의 기본단위 리소스
    * 레플리카세트는 똑같은 파드의 레플리케이션 개수를 관리 및 제어하는 리소스인데, 디플로이먼트는  
    레플리카세트를 관리하고 다루기 위한 리소스
    ~~~
    Deployment  -->  ReplicaSet --> pod1, pod2, pod3
              (create)          (create)
    ~~~
    ~~~
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: echo
        labels:
            app:echo
    spec:
        replicas:3
        selector:
            machLabels:
                app: echo
    ~~~         
    * 정의는 레플리카와 크게 다르진 않지만 디플로이먼트는 레플리카의 리비전관리를 할 수 있다
    ~~~
    kubectl apply -f simple-deployment.yaml --record 
    
    kubectl get pod,replicaset,deployment --selector app=echo //생성된 내역 확인
    
    kubectl rollout history deployment echo //리비전 확인
    ~~~
    * 쿠버네티스는 디플로이먼트를 단위로 애플리케이션을 배포한다 실제 운영에서는 레플리카를 직접 다루기보다  
    디플리먼트 매니페스트 파일을 통해 다룬다
    * 디플로이먼트를 수정하면 레플리카세트가 새로 생성되고 기본 레플리카세트와 교체된다
        * 파드개수만 수정하면 레플리카세트가 새로 생성되지 않음
        * 컨테이너의 정의(simple-deployment.yaml)를 수정하면 새로운 파드가 생기고 기존 파드는 단계적으로 정지 
    * 롤백 실행
        * 리비전 번호를 통해 롤백할 수 있다
        ~~~
        kubectl rollout history deployment echo --revision=1
        ~~~
        * undo 를 하면 바로 직전 리비전으로 롤백된다
        ~~~
        kubectl rollout undo deployment echo
        ~~~
        * 삭제는 매니페스트 파일을 이용해 삭제한다
        ~~~
        kubectl delete -f simple-deployment.yaml
        ~~~
        
* 서비스
    * 쿠버네티스 클러스터 안에서 파드의 집합(레플리카세트)에 대한 경로나 서비스 디스커버리를 제공하는 리소스  
    (디스커버리: api주소가 동적으로 바뀌어도 클라이언트가 접속대상을 바꾸지 않고 하나의 이름으로 접근 가능)
    * 즉 서비스의 대상이 되는 파드는 서비스에서 정의하는 레이블 셀렉터로 정해짐 
    * Pod의 경우에 지정되는 Ip가 랜덤하게 지정이 되고 리스타트 때마다 변하기 때문에 고정된 엔드포인트로 호출이 
    어렵다, 또한 여러 Pod에 같은 애플리케이션을 운용할 경우 이 Pod 간의 로드밸런싱을 지원해줘야 하는데, 
    서비스가 이러한 역할을 한다.  
    서비스는 지정된 IP로 생성이 가능하고, 여러 Pod를 묶어서 로드 밸런싱이 가능하며, 고유한 DNS 이름을 가질 수 있다.
                        
    ~~~
    kubectl apply -f simple-replicaset-with-label.yaml
    
    kubectl get pod -l app=echo -l release=spring
    
    kubectl get pod -l app=echo -l release=summer
    ~~~
    * release=summer의 파드만 접근할 수 있는 서비스생성
    ~~~
    apiVersion: v1
    kind: Service
    metadata:
        name:echo
    spec:
        selector:
            app:echo
            release:summer
        ports:
        - name:http
          port:80
    ~~~
    ~~~
    http://echo/ --> service(app=echo          -->pod(app=echo, release=summer)
                             release=summer)   -->pod(app=echo, release=summer)
                                               -->X pod(app=echo, release=spring) 
    ~~~                      

    * ClusterIP 서비스
        * 클러스터란 각기 다른 서버들을 하나로 묶어서 하나의 시스템으로 동작하게함으로써 클라이언트들에게 고가용성 서비스  
        를 제공 하는것을 말한다. 클러스터에 묶임 한시스템에 장애가 발생하면 다른 정상적인 서버로 이동한다.  
        서버클러스토로 하여금 서버 기반 정보를 지속적이고 끊기지않게 제공받을 수 있게 한다
        * 서비스에도 여러 종류가 있지만 종류의 기본값은 ClusterIP 서비스로 내부 ip 할
        * ClusterIP를 사용하면 쿠버네티스 클러스터 내부 IP 주소에 서비스를 공개 할 수 있으며 이를 이용해 
        어떤 파드에서 다른 파드 그룹으로 접근할때 서비스를 거쳐 가도록 할 수 있다 
        * 외부에서는 접근 못함
    * NodePort 서비스
        * 클러스터 ip 로만 접근가능하는것이 아니라 외부에서 접근할 수 있는 서비스
        * 각 노드에서 서비스 포트로 접속하기 위해 글로벌 포트를 개방한다
    * LoadBalancer 서비스          
        * 로컬 쿠버네티스 환경에서는 사용할 수 없다. 주로 클라우드 플랫폼에서 사용
    * ExternalName 서비스
        * 외부서비스를 쿠버네티스 내부에서 호출하고자 할때 사
* 인그레스
    * 외부에서 서비스를 공개하려면 서비스를 NodePort로 노출 시키지만 이방법은 L4레벨까지만 가능하다
    * http/https 처럼 경로 기반으로 서비스를 전환하는 L7 레벨 제어는 불가능
    * 이를 해결하려는 리소스가 인그레이스다. http/https 서비스를 노출하는 경우 항상 사       
    * 클러스터 외부에서 내부로 접근하는 요청들을 어떻게 처리할지 정의해둔 규칙들의 모음
        1. 외부에서 접근가능한 URL을 사용할 수 있게 하고
        2. 트래픽 로드밸런싱도 해주고, SSL 인증서 처리도 해주고, 
        3. 도메인 기반으로 가상 호스팅을 제공하기도 합니다. 
        4. 인그레스 자체는 이런 규칙들을 정의해둔 자원이고 이런 규칙들을 실제로 동작하게 해주는게 인그레스 컨트롤러
    ~~~
    개념을 도식화하면 Ingress가 서비스 앞에서 L7 로드밸런서 역할을 하고 URL에 따라 라우팅함
            --> /users     -->Service --> pod1, pod2 ,pod3
    Ingress    
            --> /products  -->Service --> pod1, pod2 ,pod3
    ~~~        
                   