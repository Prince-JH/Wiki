# Load Balancer

## ALB vs NLB
Application Load Balancer (ALB):

- ALB는 7계층(HTTP/HTTPS 계층)에서 동작한다. HTTP, HTTPS, HTTP/2 트래픽에 최적화되어 있으며, 이를 통해 content-based routing, host-based routing, path-based routing 등의 고급 라우팅 기능을 제공한다.
- ALB는 WebSocket과 같은 특정 프로토콜 지원을 포함하여 다양한 추가 기능을 제공한다.
- ALB는 SSL/TLS 종단 간 연결 및 SSL/TLS 인증서의 관리와 업데이트를 자동화할 수 있다.
- ALB는 애플리케이션의 세션 상태를 기반으로 사용자를 특정 인스턴스로 라우팅하는 스티키 세션 기능을 지원합니다.
- ALB는 인스턴스에 대한 연결이 로드 밸런서에서 설정되므로 웹 서버 액세스 로그에는 로드 밸런서의 IP 주소가 캡처된다. Client의 IP를 얻기 위해서는 X-Forwarded-For 헤더를 사용한다.

Network Load Balancer (NLB):

- NLB는 4계층(TCP/UDP 계층)에서 동작한다. 이로 인해 NLB는 초당 수백만 개의 요청을 처리할 수 있는 높은 처리량과 낮은 지연 시간을 가지며, 특히 TCP/UDP 트래픽에 대해 최적화되어 있다.
- NLB는 소스 IP를 유지하므로 클라이언트의 실제 IP 주소를 백엔드 인스턴스가 알 수 있다.
- NLB는 각 연결에 대해 최대 350 초까지 연결 시간 초과 값을 설정할 수 있다.
