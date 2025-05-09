# VPC 
## VPC보다 VPN(Virtual Private Network) 먼저
보안상의 이유로 직원간 네트워크를 분리하고 싶다면 건물의 내부선을 다 뜯어고쳐야 한다. 이를 위해 가상의 망 VPN을 사용한다.  
VPN은 네트워크 A와 네트워크 B가 실제로 같은 네트워크상에 있지만 논리적으로 다른 네트워크인 것처럼 동작한다. 이를 우리는 '가상사설망'이라고 한다.  

## VPC(Virtual Private Cloud)  
![Image](https://github.com/user-attachments/assets/30d2fabf-d5df-42f4-a0bb-8e6709a7035b)
VPC가 없다면 EC2 인스턴스들이 서로 거미줄처럼 연결되고 인터넷과 연결된다.  
이런 구조는 인스턴스만 추가되도 모든 인스턴스를 수정해야하는 불편함이 생긴다. 마치 인터넷 전용선을 다시 까는것과 같다.  
  
![Image](https://github.com/user-attachments/assets/eab71d2a-1f85-44c0-ad88-fcbb838dd311)  
VPC를 적용하면 위 그림과 같이 VPC별로 네트워크를 구성할 수 있고 각각의 VPC에 따라 다르게 네트워크 설정을 줄 수 있다. 또한 각각의 VPC는 완전히 독립된 네트워크처럼 작동하게 된다.  
  
ncloud의 VPC 설명을 참고하자.  
> VPC(Virtual Private Cloud)는 퍼블릭 클라우드 환경에서 사용할 수 있는 고객 전용 사설 네트워크입니다. 다른 네트워크와 논리적으로 분리되어 있어 IT 인프라를 안전하게 구축하고 간편하게 관리할 수 있습니다.
  
## VPC를 구축하는 과정
VPC를 구축하기 위해서는 VPC의 아이피 범위를 RFC1918이라는 private ip 대역에 맞추어 구축해야 한다.  
> 예를 들어 누군가 "안방에서 리모컨 좀 가져다 달라"라고 하면 옆집을 가는게 아닌 우리집에 있는 "안방"으로 찾아간다. 안방이 private ip(사설 ip), 집 주소가 public ip이다. 옆집도 안방이 있고 우리집도 안방이 있지만 서로 안방을 들었을 때 헷갈리지 않는다.  

안방, 작은방, 큰방처럼 내부에서 쓰는 주소를 private ip 대역이라고 부르며 내부 네트워크 내에서 위치를 찾아갈 때 사용한다. 물론 내 친구나 동생의 친구가 찾아올 때는 도로명 주소(public ip)를 알려주면 되고 우리집에 사는(동일한 네트워크 상 존재하는) 내가 동생에게 갈 때는 동생 방(private ip)로 찾아간다.  
  
VPC에서 사용하는 사설 아이피 대역은 아래와 같다.  
10.0.0.0 ~ 10.255.255.255 (10/8 prefix)  
172.16.0.0 ~ 172.31.255.255 (172.16/12 prefix)  
192.168.0.0 ~ 192.168.255.255 (192.168/16 prefix)  

한번 설정된 ip 대역은 수정할 수 없으며 각 VPC는 하나의 리전에 종속된다. 각각의 VPC는 완전히 독립적이기 떄문에 만약 VPC간 통신을 원한다면 VPC 피어링 서비스를 고려해볼 수 있다.  
  
### 서브넷
![Image](https://github.com/user-attachments/assets/f8d096e1-3960-4084-989b-65b57a1ffe9d)
서브넷은 VPC를 잘게 쪼개는 과정이다. 서브넷은 VPC보다 더 작은 단위이기 때문에 ip 범위가 더 작은 값을 갖게 된다. 서브넷을 나누는 이유는 더 많은 네트워크망을 만들기 위해서이다.  
  
각각의 서브넷은 가용영역 안에 존재하며 서브넷 안에 RDS, EC2와 같은 리소스들을 위치시킬 수 있다.  
  
### 라우팅 테이블과 라우터
![Image](https://github.com/user-attachments/assets/95884419-6b46-42dc-975e-eb9e5506e32a)
네트워크 요청이 발생하면 데이터는 우선 라우터로 향하게 된다. 라우터는 목적지이고 라우팅 테이블은 각 목적지에 대한 이정표이다. 데이터는 라우터로 향하게 되며 네트워크 요청은 각각 정의된 라우팅 테이블에 따라 작동한다.  
서브넷 A의 라우팅 테이블은 172.31.0.0/16, 즉 VPC 안의 네트워크 범위를 갖는 네트워크 요청은 로컬에서 찾도록 되어있다. 하지만 그 이외 외부로 통하는 트래픽을 처리할 수 없다. 이때 인터넷 게이트웨이를 사용한다.  
  
### 인터넷 게이트웨이
![Image](https://github.com/user-attachments/assets/95e66f02-96d4-4cb0-9f0d-09554adf6cdc)
인터넷 게이트웨이는 VPC와 인터넷을 연결해주는 하나의 관문이다. 서브넷 B의 라우팅 테이블을 보면 0.0.0.0/0 으로 정의되어 있다. 이 뜻은 모든 트래픽에 대하여 IGA A로 향하라는 뜻이다. 라우팅 테이블은 가장 먼저 목적지의 주소가 172.31.0.0/16에 매칭되는지를 확인한 후 매칭되지 않는다면 IGA A로 보낸다.  
  
### NAT 게이트웨이
NAT 게이트웨이는 private 서브넷이 인터넷과 통신하기 위한 아웃바운드 인스턴스이다.  
퍼블릭 서브넷상에서 동작하는 NAT 게이트웨이는 private 서브넷에서 외부로 요청하는 아웃바운드 트래픽을 받아 인터넷 게이트웨이와 연결한다.  
  
출처  
- https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098  