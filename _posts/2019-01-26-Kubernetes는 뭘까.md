---
layout: post
title:  "Kubernetes는 뭘까"
date:   2019-01-26 14:38:20 +0900
categories: kubernetes k8s CNCF
---

## Kubernetes를 시작하기 앞서

> *최신 개발 트렌드는 ...*
>
> *어플리케이션의 구조를 <u>작고, 독립적인 단위</u>로 개발하고 (Microservices),*
>
> *이를 <u>경량화된 가상화 환경</u>에서 구동할 수 있는 단위 (Container)로 생성하여,*
>
> *이러한 <u>컨테이너들을 관리</u>할 수 있는 환경 (Cloud Native)을 구성하는 것이다*

<br>


### 1. Microservice Architecture (MSA)

과거에는 서비스를 하나의 애플리케이션으로 만들어 모든 시스템을 그 하나에 다 집어넣는 **모놀리식 아키텍처(Monorithic Architecture)** 로 만들었다. 이러한 일체식 구조는 개발/배포/확장을 단순하게 만드는 장점을 가지지만, 큰 규모일 수록 코드이해나 수정이 어렵다.

그래서 등장하게된 **마이크로서비스 아키텍처(Microservice Architecture)** 는 서비스를 <u>작고</u>, <u>독립적이고</u>, <u>느슨하게 결합</u>하는 방식의 서비스 지향 아키텍처이다. 각각의 요소를 독립적인 어플리케이션으로 만들고, API로 조합해 애플리케이션으로 만든다.

> **MSA 구성요소**
>
> - Service Discovery
> - Circuit Breaker
> - Sidecar (Service Discovery + Circuit Breaker) 
> - Service Mesh
> - Service Mesh's Control Plane


<br>

<br>

### 2. Virtualization

서버를 가상으로 분할하는 **가상화 (Virtualization)** 는, 분할된 가상의 서버 내부에서 서비스를 실행하여 리소스를 효율적으로 쓰고자하는 기술이다. 가상화는 **KVM, XEN, Hyper-V** 등의 하이퍼바이저 기반의 기술과, **Docker, LXC** 등의 컨테이너 기반의 기술이 발전하면서 상용화되고 있다.

<br>

### Container

> **VM** 은 하이퍼바이저를 통한 하드웨어의 가상화이고, 
>
> **Container** 는 OS레벨의 가상화 (User 공간의 추상화)를 제공한다.

**Container**는 host 시스템의 커널을 container들끼리 공유하기때문에 가볍고 빠른 속도를 가지며 편리하다.

모듈성(modularity)와 확장성(scalability)이 좋지만 보안성이 약하다 => VM과 공존 필요

​		*여러대의 서버에 여러대의 어플리케이션을 쓴다면 VM이,*

​		*하나의 서버에 여러대의 어플리케이션을 쓴다면 Container가 적합할 수 있다*

<br>

### Docker

리눅스 컨테이너를 기반으로 하는 오픈소스 프로젝트

namespace, control group(cgroup)과 같은 리눅스 커널 기능을 이용해서 OS 위에 컨테이너들을 생성하는 기술이다.


<br>
<br>


### 3. Cloud Native Computing Foundation (CNCF)

**Cloud Native Computing** 은 클라우드 컴퓨팅 모델의 장점을 모두 활용하는 애플리케이션을 개발하고 실행하기 위한 접근 방식이다.

microservice로 앱을 배포하고, 컨테이너 별로 패키징하고, 리소스 사용량을 최적화하는 동적 조절을위해 오픈소스 소프트웨어를 사용한다. **Cloud Native Computing Foundation (CNCF)** 는 이러한 클라우드 기술과 관련된 표준형을 개발하려는 단체이며, **<u>Kubernetes</u>**가 유일한 중심 프로젝트로 편성되었다.

>  kubernetes, prometheus, envoy, istio, ...


<br>

<br>

<br>


# Kubernetes (k8s) 란?

> *그래서 MSA형태로 개발된 서비스들을 Docker로 컨테이너화해서 띄우긴했는데..*
>
> * 여러대의 물리서버에서 각각 관리하기도 어렵고
>
> - <u>lifecycle management</u>도 필요하고 (문제 대응, 패치, 업데이트 등)
>
> - 컨테이너 배포, 스케일링, 오퍼레이팅등도 자동으로 되면 좋겠는데...
>
>   => ("해결사가 왔어!") **Kubernetes**

<br>


### Kubernetes

**kubernetes**는 "Docker container Orchestration tool" 

컨테이너화된 어플리케이션을 Automatic deployment / Scaling / Management

\- 그리스어로 '키잡이'라는 뜻으로, 줄여서 k8s라고 부른다. (k와 s사이에 8글자)

\- Google에서 최초 개발되었고 현재는 CNCF에 기증된 상태

> *service를 host os를 공유하는 container화해서 올리는  **docker***
>
> *docker를 관리하는 **k8s**  (kubernetes)* 
>
> *이러한 k8s application들을 chart화 시키고 관리하는 **helm***


<br>


### k8s Object

*  **Pod** (name + spec + containers)

\- k8s의 가장 기본단위이자 Container의 묶음

\- pod 단위로 network namespace와 ip 가질 수 있음 (!= namespace in k8s)

\- 같은 pod에서는 같은 volume 접근가능

<br>

* **ReplicaSet** (**Pod** + replicas)

\- <u>replica</u>는 복제라는 의미로 replicas 수만큼 pod가 유지되도록 관리된다

\- pod는 죽으면 다시 되살리지않지만, ReplicaSet으로 만들면 replicas 수 (=pod 수)에 맞게 계속 살린다

\- (단, 모니터링이 되어 autoscaler가 작동되고 있는 상황이어야 함)

<br>

* **Deployment** (**ReplicasSet** + History (revision))

\- deployment는 name + replicas + pod 내용으로 구성되고, 대부분은 deployment를 사용해서 배포한다

\- 버전별로 설치/롤백되고 배포 관리가 가능하다

\- apps/v1일때는 <u>selector</u>가 있어야 <u>labels</u>를 가져올 수 있다



> ​	`kubectl create deployment.yaml` *로 Deployment를 생성할 때 순서를 확인해보면 ,*
>
> ​	*" Deployment -> ReplicaSet -> Pod "  순으로 생성된다*


<br>

* **Service**

Load balancer를 이용하여 여러 pod들을 하나의 ip, port로 묶어서 제공하는 DNS이다

그 기준은 <u>label selector</u> 로, 특정 label을 가진 것들을 하나의 서비스로 묶는다.

> Service object 노출 방식 3가지
>
> 1. **ClusterIP** - default값으로, Service에 Cluster IP (내부 IP)를 할당한다. 클러스터 내부에서만 접근 가능하고 외부에서는 접근이 불가능
> 2. **NodePort** - 각각의 Node의 IP와 static 포트를 노출하여 접근가능하게 하고, 클러스터 외부에서도 접근가능
> 3. **Load Balancer** - Cloud provider(GCE/AWS)와 같은 외부IP를 가진 Load balancer에게 Service를 노출


<br>

* 그 외 고오급 오브젝트

\- **DaemonSet** : 맵핑된 label이 있는 node가 추가되면 자동으로 해당 node에 pod 생성을 보장 (scaling)

\- **StatefulSet** : 컨테이너가 제거/재시작되어도 상태의 영속성과 지속성을 보장	=> like DB
<br>

> \- **Affinity** : kube-schedular에게 정보 제공. 부하 분산 또는 버전관리 가능
>
> ( Session **Affinity** - sticky session 제공 (canary deployment) )


<br>
<br>


### 기타 Keyword

**Docker 배포**

> 특징 : 확장성, 표준성, 이미지 기반, 환경변수로 제어하는 설정, 공유자원..

배포툴 : <u>**kubernetes**</u>, docker swarm, coreos, fleet,...


<br>

**Kubernetes 배포 프로세스**

> *binary build -> containerizing(image) -> push image -> service define -> test deploy (canary test) -> prod deploy*  
>
> => 어렵고 복잡. 이런 배포 프로세스를 통합/자동화하는 CICD (배포툴) 필요!

배포툴 : <u>kubespray</u>, kubeadm, kops, ... (CaaS 지원)


<br>
<br>

\* **오케스트레이션** (Orchestration)

여러 서버를 운영할때, 이들을 관리하는 것

- IaC를 돕는 설정관련 도구는 chef puppet <u>Ansible</u> SaltStack…

- CI/CD 관리 도구는 Travis CI, <u>Jenkins</u>, Circle CI ..

- 컨테이너관리 도구는 Docker swarm, <u>Kubernetes</u> …

<br>

\* **Ansible**

구성관리 tool로, 인프라 관리과정을 코드로 기술한 IaC (Infra as Code)를 효율적이고 자동으로 관리할수있는 인프라 도구.

Python 기반의 개발 + YAML로 정의 + JSON으로 통신

초기설정이나 모니터링, 변경사항 추적이 불가능하다는 단점이 있지만, shell command를 제외하고는 모두 **Idempotency(멱등성)** 을 제공한다.

>  *kubespray는 ansible 기반의 배포툴이다.*

<br>


\* **Helm**

Chart라는 개념으로 kubernetes의 application을 정의, 배포하고 관리

- Chart: app 구성하는 Kubernetes 객체들을 정의한 manifest template파일 및 설정묶음
- Cient (helm client, CLI) - Server (**tiller**, pod형태로 배포됨) 구조
- Release: client 통해 kube 위에 배포된 app

=> helm client 설치 후 tiller server를 kubernetes cluster위에 설치해야함

```
helm init (—upgrade)	// tiller 설치 
helm install 			// repository에 등록된 chart를 client->tiller로 보냄
helm lint				// chart의 문법검사
```

<br>

\* **CI/CD**

- CI (Continuous Integration): 지속적 통합, 자주 Build & Packaging

- CD (Continous Delivery / Deployment): 지속적 배포, 자주 Deployment

<br>
<br>


### 참조

1. https://www.samsungsds.com/global/ko/support/insights/101917_RD_Cloudnative.html)
2. https://engineering.linecorp.com/ko/blog/infrastructure-trends-open-infra-days-korea-2018/
3. 갓승규님 블로그 https://ahnseungkyu.com/ 
4. Google Cloud - JAM k8s 입문반 QWIK LAB 진행
