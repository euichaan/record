# DNS란?
DNS(Domain Name System)는 인터넷 전화번호부이다. 사람들은 espn.com과 같은 도메인 이름을 통해 온라인으로 정보에 액세스한다. 웹 브라우저는 인터넷 프로토콜(IP) 주소를 통해 상호작용한다.  
  
**DNS는 브라우저가 인터넷 자원을 로드할 수 있도록 도메인 이름을 IP 주소로 변환한다.**  
  
# DNS는 어떻게 작동하나요?
DNS 확인 프로세스에는 호스트 이름(예: www.example.com)을 컴퓨터 친화적인 IP 주소(예: 192.168.1.1)로 변환하는 과정이 포함된다. 사용자가 어떤 웹페이지를 로드하려고 할 때에는 사용자가 웹브라우저에 입력한 내용(예: www.example.com)을 웹페이지를 찾는 데 필요한 컴퓨터 친화적 주소로 변환해야 한다.    
  
웹브라우저 입장에서는 DNS 확인이 백그라운드에서 발생하며 최초의 사용자 요청 외에 사용자 컴퓨터와의 추가적인 대화는 필요하지 않다.  
  
웹 페이지 로드와 관련된 4개의 DNS 서버가 있다.  
**DNS 리커서** - 리커서는 도서관의 어딘가에서 특정한 책을 찾아달라고 요청받는 사서로 생각할 수 있다. DNS 리커서는 웹 브라우저 등의 애플리케이션을 통해 클라이언트 컴퓨터로부터 쿼리를 받도록 고안된 서버이다. 일반적으로, 리커서는 클라이언트의 DNS 쿼리를 충족시키기 위해 추가 요청을 수행한다.  
  
**루트 네임 서버** - 루트 네임 서버는 사람이 읽을 수 있는 호스트 이름을 IP 주소로 변환(확인)하는 첫 번째 단계이다. 도서관에서 책장 위치를 가리키는 색인으로 생각할 수 있으며, 일반적으로 다른 더욱 특정한 위치에 대한 참조로 사용된다.  
  
**TLD 네임 서버** - TLD 서버는 도서관의 특정 책장으로 생각할 수 있다. 이 이름 서버는 특정 IP 주소 검색의 다음 단계이며 호스트 이름의 마지막 부분을 호스팅한다.(example.com에서 TLD 서버는 "com"이다).  
  
**권한 있는 네임 서버** - 최종 네임 서버로서, 책장에 있는 사전처럼 특정 이름을 해당 정의로 변환한다. 권한 있는 네임 서버(Authoritative Name Server)는 네임 서버 쿼리의 종착점이다. 권한있는 네임 서버가 요청한 레코드에 대한 액세스 권한이 있다면, 요청한 호스트 이름의 IP 주소를 초기 요청을 한 DNS 리커서에게 돌려 보낸다.  
  
# 권한 있는 DNS 서버와 재귀 DNS 확인자의 차이점은 무엇인가?
### 재귀 DNS 확인자
재귀 확인자는 클라이언트의 재귀 요청에 응답하고 DNS 레코드를 추적하는 데 시간을 투자하는 컴퓨터이다. 요청한 레코드에 대해, 권한있는 DNS 네임 서버에 도달할 때까지 일련의 요청을 하는 방식으로 이를 수행한다(또는 레코드가 없으면 시간 초과되거나 오류를 반환). 다행히 재귀 DNS 확인자가 클라이언트에 응답하는 데 필요한 레코드를 추적하기 위해 항상 다수의 요청을 해야 하는 것은 아니다. (캐싱이 있기 때문)  
  
### 권한 있는 DNS 서버
권한 있는 DNS 서버는 실제로 DNS 리소스 레코드를 보유하고 담당하는 서버이다. 이 서버는 쿼리한 자원 레코드로 응답하는 DNS 조회 체인의 맨 아래에 있는 서버로, 궁극적으로 웹 브라우저가 웹사이트 또는 다른 웹 자원에 액세스하는 데 필요한 IP 주소에 도달하도록 요청할 수 있게 한다. 권한 있는 네임 서버는 특정 DNS 레코드의 최종 원천이므로 다른 원천을 쿼리할 필요 없이 자체 데이터의 쿼리를 충족시킬 수 있다.  
  
# DNS 조회는 어떤 단계를 거칩니까?
## DNS 조회의 8단계
대부분의 경우, DNS는 도메인 이름을 적절한 IP 주소로 변환하는 일에 관여한다.  
```text
참고: DNS 조회 정보는 쿼리 컴퓨터 내부에서 로컬로 또는 DNS 인프라에서 원격으로 캐시되는 경우가 많다.
DNS 조회에는 일반적으로 8단계가 있지만, DNS 정보가 캐시되어 있으면 DNS 조회 프로세스에서 몇 단계를 건너 뛸 수 있으므로, 더 빨라집니다.
아래 예시는 캐시되지 않은 8단계를 모두 보여준다.
```
  
1. 사용자가 웹 브라우저에 'example.com'을 입력하면, 쿼리가 인터넷으로 이동하고 DNS 재귀 확인자(DNS 리커서)가 이를 수신한다.  
2. 이어서 확인자가 DNS 루트 네임 서버(.)를 쿼리한다.  
3. 다음으로, 루트 서버가, 도메인에 대한 정보를 저장하는 최상위 도메인(TLD) DNS 서버(예: .com 또는 .net)의 주소로 확인자에 응답한다.  
4. 이제, 확인자가 .com TLD에 요청한다.  
5. 이어서, TLD 서버가 도메인 네임 서버(example.com)의 IP 주소로 응답한다.  
6. 마지막으로, 재귀 확인자가 도메인의 네임 서버로 쿼리를 보낸다.  
7. 이제, example.com의 IP 주소가 네임 서버에서 확인자에게 반환된다.  
8. 이어서, DNS 확인자가, 처음 요청한 도메인의 IP 주소로 웹 브라우저에게 응답한다.  
  
![what_is_a_dns_server_dns_lookup](https://github.com/user-attachments/assets/dd14049d-aa02-4ab9-90fb-88048d30bf8e)  

DNS 조회의 8단계를 거쳐 example.com의 IP 주소가 반환되면, 이제 브라우저가 웹 페이지를 요청할 수 있다.      
9. 브라우저가 IP 주소로 HTTP 요청을 보낸다.  
10. 해당 IP의 서버가 브라우저에서 렌더링할 웹 페이지를 반환한다.  
  
# DNS 쿼리에는 어떤 유형이 있습니까?
일반적인 DNS 조회에서는 세 가지 유형의 쿼리가 발생한다.  
1. 재귀 쿼리 - 재귀 쿼리에서는, 확인자가 레코드를 찾을 수 없는 경우, DNS 클라이언트는 DNS 서버(일반적으로 DNS 재귀 확인자)가, 요청한 자원 레코드 또는 오류 메시지를 사용하여 클라이언트에 응답하도록 요구합니다.  
2. 반복 쿼리 - 이 경우, DNS 클라이언트는 DNS 서버가 가능한 최상의 응답을 반환하도록 한다. 쿼리한 DNS 서버가 쿼리 이름과 일치하는 이름을 갖고 있지 않은 경우, 하위 수준의 도메인 네임스페이스에 대해 권한 있는 DNS 서버에 대한 참조를 반환한다. 그러면 DNS 클라이언트가 참조 주소를 쿼리한다. 이 프로세스는 오류 또는 제한 시간 초과가 발생할 때까지 추가 DNS 서버가 쿼리 체인을 중단한 상태로 계속된다.  
3. 비재귀 쿼리 - 일반적으로, DNS 확인자 클라이언트의 쿼리를 받은 DNS 서버가 해당 레코드에 대한 권한이 있거나 캐시 내부에 해당 레코드를 갖고 있어, DNS 서버가 액세스 권한을 갖고 있는 레코드를 쿼리할 때 발생한다. 일반적으로, DNS 서버는 추가 대역폭 서버 및 업스트림 서버의 부하를 방지하기 위해 DNS 레코드를 캐시한다.  

# DNS 캐싱이란 무엇입니까? DNS 캐싱은 어디서 발생합니까?
DNS 캐싱은 요청하는 클라이언트와 가까운 곳에 데이터를 저장함으로써, DNS 쿼리를 조기에 확인할 수 있고 DNS 조회 체인의 추가 쿼리를 피할 수 있으므로, 로드 시간이 향상되고 대역폭/CPU 소비가 줄어든다.  
DNS 데이터는 다양한 위치에 캐시될 수 있으며, 각 위치는 TTL(Time-To-Live)에 의해 정의된 설정 시간 동안 DNS 레코드를 저장한다.  
  
## 브라우저 DNS 캐싱
최신 웹 브라우저는 기본적으로 정해진 시간 동안 DNS 레코드를 캐시하도록 설계되었다.  
**DNS 캐싱이 웹 브라우저와 가까울수록 캐시를 확인하고 IP 주소에 대한 올바른 요청을 하기 위해 처리해야 할 단계가 적어진다.**  
DNS 레코드를 요청할 때 브라우저 캐시에서 처음으로 요청한 레코드를 확인하는 것이다.  
  
Chrome 에서는 chrome://net-internals/#dns에서 DNS 캐시의 상태를 볼 수 있다.  
## 운영체제 (OS) 수준 DNS 캐싱
운영 체제 수준 DNS 확인자는 DNS 쿼리가 컴퓨터를 떠나기 전의 두 번째 중단점이며, 로컬에 있는 마지막 중단점이다. 이 쿼리를 처리하도록 설계된 운영 체제 내부의 프로세스를 일반적으로 "스텁 확인자" 또는 DNS 클라이언트라고 한다.  
스텁 확인자는 애플리케이션에서 요청을 받으면 먼저 자체 캐시를 검사하여 레코드가 있는지 확인한다. 레코드가 없으면 로컬 네트워크 외부의 (재귀 플래그가 설정된) DNS 쿼리를 인터넷 서비스 공급자(ISP) 내부의 DNS 재귀 확인자로 보낸다.  
  
ISP 내부의 재귀 확인자가 모든 이전 단계와 같이 DNS 쿼리를 수신하면, 요청한 호스트-IP-주소 변환이 로컬 지속성 계층 내에 이미 저장되어 있는지도 확인한다.  
  
재귀 확인자에는 캐시에 있는 레코드 유형에 따른 추가 기능도 있다.  
  
1. 확인자가 A 레코드는 갖고 있지 않지만, 권한있는 네임 서버에 대한 NS 레코드를 갖고 있는 경우에는, DNS 쿼리의 여러 단계를 거치지 않고 해당 이름 서버를 직접 쿼리한다. 이 바로가기는 루트 및 .com 이름 서버로부터의 조회를 방지하고 DNS 쿼리의 확인이 더 빨리 이루어지도록 도와준다.  
2. 확인자에 NS 레코드가 없는 경우, 루트 서버를 건너뛰고 TLD 서버(이 경우 .com)로 쿼리를 보낸다.  
3. 확인자에 TLD 서버를 가리키는 레코드가 없는 경우, 루트 서버를 쿼리한다. 이 이벤트는 일반적으로 DNS 캐시가 제거된 후에 발생한다.  
  
출처 - [https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns](https://www.cloudflare.com/ko-kr/learning/dns/what-is-dns)