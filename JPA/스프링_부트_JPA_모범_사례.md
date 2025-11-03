# 항목 1: @OneToMany 연관관계를 효과적으로 구성하는 방법
양방향 지연 @OneToMany 연관관계와 관련해 Author와 Book이라는 두 엔티티를 생각해보자. author의 행은 여러 book 행에 의해 참조될 수 있으며, author_id 열은 author 테이블의 기본키를 ㅊ마조하는 외래키를 통해 이 관계를 매핑한다. 저자 없이 도서는 존재할 수 없기 때문에 author가 부모 측이 되고 book은 자식 측이 된다. 이 @ManyToOne 연관관계는 외래키 열을 영속성 컨텍스트(1차 캐시)와 동기화하는 역할을 한다.  
  
일반적인 규칙으로 단방향 연관관계보다는 양방향을 사용하자.  
  
## 항상 부모 측에서 자식 측으로 전이를 사용
자식 측에서 부모 측으로의 전이는 코드 스멜이자 잘못된 관행이며 도메인 모델과 애플리케이션 설계를 다시 살펴봐야 할 명확한 신호다. 통상적으로 부모 측에서 자식 측으로만 전이를 지정한다.  
`@OneToMany(cascade = CascadeType.ALL)`  
  
부모 측에 설정되는 mappedBy 속성은 양방향 연관관계의 특성을 부여한다. 다시 말해 양방향 @OneToMany 연관관계에서 부모 측 @OneToMany에 mappedBy가 지정되고 mappedBy에 의해 참조되는 자식 측에 @ManyToOne이 지정된다. mappedBy를 통해 양방향 @OneToMany 연관관계가 @ManyToOne 자식 측 매핑을 미러링한다는 신호를 보내는 것이다. 이 경우 Author 엔티티에 다음과 같이 추가한다.  
`@OneToMany(cascade = CascadeType.ALL, mappedBy = "author")`  
