# ELB + ASG
## 고가용성 및 확장성
고가용성은 애플리케이션 시스템이 조정을 통해 더 많은 양을 처리할 수 있다는 의미.  
- Vertical Scalability  
- Horizontal Scalability (= elasticity)  
  
Vertical Scalability means increasing the size of the instance  
Horizontal Scalability means increasing the number of instances / systems for your application  
  
Horizontal scaling implies distributed systems.  
  
High Availability means running your application / system in at least 2 data centers (== AZ)  
  
The goal of high availability is to survive a data center loss  
  
The high availability can be passive (for RDS Multi AZ for example)  
The high availability can be active (for horizontal scaling)  
```
Passive HA
평소에는 하나만 일하고, 나머지는 대기 상태로 있다가 장애가 나면 자동으로 교체되는 방식.

Active HA
여러 인스턴스가 동시에 트래픽을 처리하는 방식. ELB + Auto Scaling Group으로 여러 AZ에 EC2를 띄우면 모든 인스턴스가 실제로 요청을 받아서 일한다.
```

High Availability: Run instances for the same application across multi AZ(동일한 애플리케이션의 인스턴스를 여러 AZ에 걸쳐 실행하는 것)  
  
## Why use load balancing
- 부하를 다수의 다운스트림 인스턴스로 분산하기 위해서  
- 애플리케이션에 단일 액세스 지점(DNS)을 노출  
- 다운스트림 인스턴스의 장애를 원활히 처리할 수 있음  
- 주기적으로 인스턴스의 health check  
- SSL termination  
- Enforce stickiness with cookies  
- HA across zones  
- Seperate public traffic from private traffic  
  
Elastic Load Balancer는 관리형 로드 밸런서이기도 하다.  
- AWS가 관리하며, 어떤 경우에도 작동할 것을 보장한다.  
- AWS가 업그레이드와 유지 관리 및 고가용성을 책임진다.  
- AWS provides only a few configuration knobs  
  
헬스 체크는 port와 route에서 이뤄진다.  
  
Application Load Balancer: HTTP, HTTPS, WebSocket  
Network Load Balancer: TCP, TLS(secure TCP), UDP  
Gateway Load Balancer: Operates at layer 3 (Network layer) - IP Protocol  
  
users <-> Load Balancer <-> EC2  
에서의 보안 그룹은 EC2 가 LB를 통과한 트래픽만 받을 수 있도록 포트 80에서 HTTP 트래픽을 허용하며 소스는 IP 범위가 아니라 보안 그룹이 된다. **EC2 인스턴스의 보안 그룹을 로드 밸런서의 보안 그룹으로 연결하는 것이다.**  
  
## Application Load Balancing
Layer 7, HTTP 전용 로드 밸런서  
리다이렉트 지원 (from HTTP to HTTPS for example)  
HTTP/2, WebSocket 지원  
  
포트 매핑 기능이 있어 ECS 인스턴스의 동적 포트로의 리다이렉션을 가능하게 해준다.  
  
대상 그룹  
- EC2 인스턴스  
- ECS 태스크  
- 람다 함수  
- IP 주소(must be private)  
  
ALB는 여러 대상 그룹으로 라우팅할 수 있으며 상태 확인은 대상 그룹 레벨에서 이루어진다.  
```
타겟 그룹: ALB가 트래픽을 보낼 대상들의 묶음.

클라이언트 -> ALB(리스너) -> 타겟 그룹 (EC2 인스턴스들)
```
  
로드 밸런서를 사용하는 경우 고정된 hostname이 부여된다. (XXX.region.elb.amazonaws.com)  
application server는 클라이언트의 IP를 직접적으로 볼 수 없다.  
- 클라이언트의 실제 IP는 X-Forwarded-For라고 불리는 헤더에 삽입된다.  
- X-Forwarded-Proto에 의해 사용하는 프로토콜도 얻게 된다.  
  
백엔드 인스턴스의 입장에서 보면 요청의 출발지 IP가 클라이언트가 아니라 ALB의 private IP로 찍힌다.  

### Application Load Balancer의 작동 방식
1. 클라이언트가 애플리케이션에 요청을 보냅니다.  
2. **로드 밸런서의 리스너**는 구성한 프로토콜 및 포트와 일치하는 요청을 수신합니다.  
3. 수신 리스너는 지정된 규칙에 따라 수신 요청을 평가하고, 해당되는 경우 요청을 적절한 대상 그룹으로 라우팅합니다. HTTPS 리스너를 사용하여 TLS 암호화 및 복호화 작업을 로드 밸런서로 오프로드할 수 있습니다.  
4. 하나 이상의 대상 그룹에 있는 정상 대상은 로드 밸런싱 알고리즘과 리스너에서 지정한 라우팅 규칙을 기반으로 트래픽을 수신합니다.  
  
EC2 인스턴스의 보안 그룹에서 인바운드 규칙을 로드밸런서의 보안 그룹으로 연결하면 EC2 public ip로의 트래픽을 차단할 수 있다. 클라이언트는 로드 밸런서를 통해서만 요청을 할 수 있다.  
  