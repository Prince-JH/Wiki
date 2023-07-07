# WebFLux
Spring 5에서 추가된 모듈로, non-blocking reactive stream 을 지원한다.

## timeout vs readTimeout
timeout 옵션과 readTimeout 옵션은 WebClient에서 요청을 처리하는 데 사용되는 서로 다른 타입의 타임아웃이다.

- timeout: timeout 옵션은 WebClient에서 요청을 시작한 이후로부터 결과를 받을 때까지의 전체 시간에 대한 제한. 이것은 연결 설정, 요청 전송, 서버 처리, 그리고 응답 수신에 걸리는 전체 시간을 포함한다. 코드에서 `.timeout(Duration.ofSeconds(5))`와 같이 설정하여 5초 이내에 요청이 완료되도록 할 수 있다. 이 시간이 초과되면 TimeoutException이 발생한다.

- readTimeout: readTimeout은 서버로부터 데이터를 읽는데 걸리는 시간에 대한 제한이다. 이것은 응답 헤더를 수신한 후부터 응답 본문을 모두 수신할 때까지의 시간을 의미한다. 응답이 시작되면, 각 데이터 블록 사이에 너무 많은 시간이 소요되지 않도록 하는 것이 중요하다. 이는 대량의 데이터를 전송하는 요청에 특히 중요하다.

두 타입의 타임아웃은 서로 보완적인데, timeout은 요청의 전체 수명에 제한을 두고, readTimeout은 서버로부터 데이터를 읽는 데 걸리는 시간에 초점을 맞츤디. 따라서 일반적으로 timeout 값은 readTimeout 값보다 크거나 같아야 한다.
