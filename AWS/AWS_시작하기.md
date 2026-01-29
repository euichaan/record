# AWS 시작하기
Region 에는 고유한 이름이 있다.  
**Region is a cluster of data centers** 데이터 센터가 모여있는 곳.  
대부분의 서비스는 특정 Region 범위로 연결된다.  
한 Region에서 서비스를 사용하고 다른 Region에서 다시 사용하려면 새로 서비스를 이용하는 것과 같다.  
  
## Region 선택에 영향을 줄 수 있는 요소  
- Compliance(규정 준수): 정부가 데이터를 해당 국가에 두길 원한다.
- Proximity(근접성) to customers: reduced latency  
- Available services within a Region: new services and new features aren't available in every Region   
- Pricing: Region마다 가격이 다를 수 있다.  
  
## AWS Availability Zones (AZ)
- Each region has many availability zones(ap-southeast-2a, ap-southeast-2b...)  
- Each AZ is one or more discrete data centers(개별 데이터 센터) with redundant power, networking, and connectivity    
- **They're separate from each other, so that they're isolated from disasters**  
- They're connected with high bandwith, ultra-low latency networking  
- 물리적으로 분리된 데이터 센터  
  
## AWS Points of Presence (Edge Locations)  
Points of Presence (PoP) / Edge Location은 AWS의 CDN인 CloudFront가 사용하는 전 세계에 분산된 캐시 서버 위치.  
Edge Location에 콘텐츠를 캐싱해두면 미국 유저는 가까운 Edge에서 바로 받아서 빠르다.  
Regional Edge Cache: Edge Location과 Origin 사이의 중간 캐시 계층. Edge에서 캐시 미스 나면 Origin 까지 안 가고 Regional Cache에서 먼저 찾아봄.  
유저 -> Edge Location (400+ 개) -> Regional Edge Cache (10+ 개) -> Origin 서버  
  
서비스가 Region에서 가능한지 확인하려면 Region 테이블을 확인하면 된다.  
  
AWS의 모든 서비스가 모든 리전에 있는 것은 아니다.  

```  
아시아 태평양의 AWS 클라우드에는 23 엣지 네트워크 로케이션 및 4 엣지 캐시 로케이션와 함께 13 지리적 리전 내에 있는 41 가용 영역이 포함되어 있습니다.  
```  

[How CloudFront delivers content](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HowCloudFrontWorks.html#CloudFrontRegionaledgecaches)  
  
