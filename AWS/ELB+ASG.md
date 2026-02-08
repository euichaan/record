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
  

  
