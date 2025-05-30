# DNS 레코드
DNS 레코드(또는 영역 파일)는 권한 있는 DNS 서버에 있는 명령으로서, **도메인에 연계된 IP 주소 및 해당 도메인에 대한 요청의 처리 방법에 대한 정보를 제공한다.** 이 레코드는 DNS 구문이라고 하는 일련의 텍스트 파일로 구성된다. DNS 구문은 DNS 서버에게 수행할 작업을 알려주는 명령으로 사용되는 문자열이다. 또한, 모든 DNS 레코드에는 TTL이 있는데, 이는 DNS 서버가 해당 레코드를 새로 고치는 빈도를 나타낸다.  
  
# DNS 레코드 유형
### A 레코드
A는 주어진 도메인의 IP 주소를 나타낸다. A 레코드는 IPv4 주소만 보유한다. 웹 사이트에 IPv6 주소가 있는 경우 웹 사이트는 대신 "AAAA" 레코드를 사용한다.  
다음은 A 레코드의 예를 나타낸다.  
| example.com | 레코드 유형 | 값 | TTL |
|----------|----------|----------|----------|
|  @  |   A  |   192.0.2.1  |  14400  |  
  
  
이 예제의 "@" 기호는 루트 도메인에 대한 레코드임을 나타낸다. A 레코드의 기본 TTL은 14,400초이며, 이는 A 레코드가 업데이트되면 적용되는 데 240분(14,400초)이 걸림을 의미한다.  
  
A 레코드의 가장 일반적인 용도는 IP 주소 조회, 즉 도메인 이름(예: "example.com")을 IPv4 주소와 일치시키는 것이다. 이를 통해 사용자의 장치는 사용자가 실제 IP 주소를 기억해서 입력하지 않아도 웹 사이트에 연결하고 로드할 수 있다. 사용자의 웹 브라우저에서는 DNS 확인자(DNS 재귀 확인자)에 쿼리를 전송하여 이를 자동으로 수행한다.  
  
### AAAA 레코드
DNS AAAA 레코드는 도메인 이름을 IPv6 주소와 일치시킨다. DNS AAAA 레코드는 IPv4 주소 대신 도메인의 IPv6 주소를 저장한다는 점을 빼고는 DNS A 레코드와 정확히 같다.  
  
AAAA 레코드는 도메인에 IPv4 주소 외에 IPv6 주소가 있고 해당 클라이언트 주소가 IPv6을 사용하도록 구성된 경우에만 사용된다.  
  
### CNAME 레코드
CNAME 레코드는 별칭 도메인에서 정식 도메인을 가리킨다. 도메인 또는 하위 도메인이 다른 도메인의 별칭인 경우 A 레코드 대신 CNAME 레코드가 사용된다. 모든 CNAME 레코드는 도메인을 가리켜야 하며 IP 주소를 가리켜서는 안 된다. CNAME 레코드가 있는 도메인은 다른 단서(CNAME 레코드가 있는 다른 도메인) 또는 보물(A 레코드가 있는 도메인)을 가리킬 수 있는 단서와 같다.  
  
예를 들어 blog.example.com 에 'example.com' 값이 있는 CNAME 레코드가 있다고 가정하자. 이는 DNS 서버가 blog.example.com에 대한 DNS 레코드에 도달하면, 실제로 example.com에 다른 DNS 조회를 트리거하여 example.com의 IP 주소를 A 레코드를 통하여 반환함을 의미한다. 이 경우 example.com은 blog.example.com의 정식 이름이라고 말할 수 있다.  
  
종종, 사이트에 blog.example.com 또는 shop.example.com 와 같은 하위 도메인이 있는 경우, 이러한 하위 도메인에는 루트 도메인(example.com)을 가리키는 CNAME 레코드가 있다. 이런 방식으로, 호스트의 IP 주소가 변경되면 루트 도메인의 DNS A 레코드만 업데이트하면 되며, 모든 CNAME 레코드는 루트에 대한 변경 사항을 따른다.  
  
CNAME 레코드는 항상 CNAME 레코드가 가리키는 도메인과 동일한 웹 사이트로 확인되어야 한다는 오해가 자주 생기지만, 그렇지 않다. **CNAME 레코드는 클라이언트를 루트 도메인과 동일한 IP 주소로만 가리킨다.** 클라이언트가 해당 IP 주소에 도달하면 웹 서버는 그에 따라 URL을 처리한다.  
  
예를 들어 blog.example.com은 example.com을 가리키는 CNAME을 가질 수 있으며, 클라이언트에게 example.com의 IP 주소로 향하게 할 수 있다. 그러나 클라이언트가 실제로 해당 IP 주소에 연결하면 웹 서버는 URL을 보고 그 URL이 blog.example.com임을 확인하며, 홈 페이지가 아닌 블로그 페이지를 전달한다.  
  
| blog.example.com | 레코드 유형 | 값 | TTL |
|----------|----------|----------|----------|
|  @  |   CNAME  |   이는 example.com의 별칭입니다  |  32600  |  
  
MX 및 NS 레코드는 CNAME 레코드를 가리킬 수 없다. 이들 레코드는 A 레코드 또는 AAAA 레코드를 가리켜야 한다.  
  
### NS 레코드
NS 레코드는 권한있는 DNS 서버를 나타낸다. NS는 '네임 서버'를 의미하며, 네임 서버 레코드는 어떤 DNS가 해당 도메인의 권한 있는 네임 서버(실제 DNS 레코드를 갖고 있는 서버)인지 지시한다. 기본적으로 NS 레코드는 인터넷에서 해당 도메인의 IP 주소를 찾기 위해 가야 할 곳을 알려준다. 도메인에는 해당 도메인의 주요 및 보조 이름 서버를 나타낼 수 있는 NS 레코드가 다수 있는 경우가 많다. NS 레코드가 적절히 구성되어 있지 않으면 사용자는 웹 사이트나 애플리케이션을 로딩할 수 없게 된다.  
  
| example.com | 레코드 유형 | 값 | TTL |
|----------|----------|----------|----------|
|  @  |   NS  |   ns1.exampleserver.com  |  21600  |  
  
네임 서버는 DNS 서버의 유형 중 한 가지이다. 이는 A 레코드, MX 레코드, CNAME 레코드 등 특정 도메인에 관한 모든 DNS 레코드를 저장하는 서버이다.  
  
거의 모든 도메인이 안정성을 높이기 위해 다수의 네임 서버에 의존한다. 이렇게 하면, 네임서버 하나가 중단되거나 사용할 수 없게 되어도 DNS 쿼리는 다른 네임서버를 이용할 수 있다. 대부분의 경우, 기본 네임서버가 하나 있으며, 이 기본 서버의 DNS 레코드를 그대로 복사한 내용을 갖고 있는 보조 네임서버가 다수 존재하게 된다. 기본 네임서버를 수정하면 보조 서버도 마찬가지로 수정된다.  
  
### SOA 레코드
DNS SOA(Start Of Authority) 레코드는 관리자의 이메일 주소, 도메인이 마지막으로 업데이트된 시간, 새로 고침 사이에 서버가 대기해야 하는 시간 등 도메인 또는 영역에 대한 중요한 정보를 저장한다.  
모든 DNS 영역에는 IETF 표준을 준수하기 위해 SOA 레코드가 필요하다.
  
### MX 레코드
DNS 메일 교환(MX) 레코드는 이메일을 메일 서버로 보낸다. MX 레코드는 단순 전자우편 전송 프로토콜(SMTP)에 따라 이메일 메시지를 라우팅하는 방법을 나타낸다.  
| example.com | 레코드 유형 | 우선 순위 | 값 | TTL |
|----------|----------|----------|----------|----------|
|  @  |   MX  |  10  |  mailhost1.example.com  | 45000 |  
|  @  |   MX  |  20  |  mailhost2.example.com  | 45000 |  
  
우선 순위 값이 낮은 쪽이 선호된다. 10은 20보다 작기 때문에 항상 mailhost1을 먼저 시도한다. 메시지 전송이 실패하면 서버는 기본적으로 mailhost2로 연결한다.  
이메일 서비스는 두 서버의 우선 순위가 같고 동일한 양의 메일을 수신하도록 이 MX 레코드를 구성할 수도 있다.  
| example.com | 레코드 유형 | 우선 순위 | 값 | TTL |
|----------|----------|----------|----------|----------|
|  @  |   MX  |  10  |  mailhost1.example.com  | 45000 |  
|  @  |   MX  |  10  |  mailhost2.example.com  | 45000 |  
  
이렇게 하면 이메일 공급자가 두 서버 간에 부하를 균등하게 분산시킬 수 있다.  
### TXT 레코드
도메인 관리자는 DNS '텍스트'(TXT) 레코드를 사용하여 도메인 이름 시스템(DNS)에 텍스트를 입력할 수 있다. 텍스트는 따옴표로 묶인 하나 이상의 문자열 형태로 저장된다.  
오늘날 DNS TXT 레코드의 가장 중요한 두 가지 용도는 이메일 스팸 방지와 도메인 소유권 확인이지만, TXT 레코드는 원래 이러한 용도로 설계되지 않았다.  