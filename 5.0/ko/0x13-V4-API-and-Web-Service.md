# V4 API 와 WEB 서비스

## 제어할 목표

웹 브라우저나 다른 클리어언트에서 사용하기 위해서 API를 노출하는 애플리케이션 (일반적으로 JSON, XML, GraphQL 사용) 에는 몇 가지 특별히 고려해야 할 사항이 적용됩니다. 이 챕터에서는 관련 보안 설정과 적용해야할 매커니즘에 대해 다룹니다.

다른 챕터에서 다루는 인증, 세션 관리, 입력값 검증과 관련된 문제들은 API에도 동일하게 적용되므로, 이 챕터의 내용을 전체 맥락에서 벗어나 개별적으로 테스트해서는 안 된다는 점에 유의해야 합니다.

## V4.1 일반적인 웹 서비스 보안

이 섹션은 일반적인 웹 서비스 보안 고려사항과 기본적인 웹 서비스 보안 수칙에 대해서 다룹니다.

| # | 설명 | 수준 |
| :---: | :--- | :---: |
| **4.1.1** | 메세지 본문이 포함된 모든 HTTP 응답에는 실제 내용과 일치하는 Content-Type 헤더 필드가 포함되어 있는지 검증합니다. 이때 IANA 미디어 타입(text, /+xml, /xml 등)에 따라 안전한 문자 인코딩 (ex. UTF-8, ISO-8859-1)을 지정하는 charset 파라미터도 포함되어야 합니다. | 1 |
| **4.1.2** | 사용자가 대면하는 엔드포인트(사람이 직접 웹 브라우저에 접속하는 경우)만 HTTP에서 HTTPS로 자동 리디렉션하고, 그 외에 서비스나 API 엔드포인트는 명백한 리디렉션을 구현하지 않았는지 검증합니다. 이는 클라이언트가 실수로 암호화되지 않은 HTTP 요청을 보내고 있음에도, 요청이 자동으로 HTTPS로 리디렉션되어 민감한 데이터의 유출 사실을 발견하지 못하게 되는 상황을 방지하기 위함입니다. | 2 |
| **4.1.3** | 로드 밸런서, 웹 프록시,  | 2 |
| **4.1.4** | Verify that only HTTP methods that are explicitly supported by the application or its API (including OPTIONS during preflight requests) can be used and that unused methods are blocked. | 3 |
| **4.1.5** | Verify that per-message digital signatures are used to provide additional assurance on top of transport protections for requests or transactions which are highly sensitive or which traverse a number of systems. | 3 |

## V4.2 HTTP Message Structure Validation

This section explains how the structure and header fields of an HTTP message should be validated to prevent attacks such as request smuggling, response splitting, header injection, and denial of service via overly long HTTP messages.

These requirements are relevant for general HTTP message processing and generation, but are especially important when converting HTTP messages between different HTTP versions.

| # | Description | Level |
| :---: | :--- | :---: |
| **4.2.1** | Verify that all application components (including load balancers, firewalls, and application servers) determine boundaries of incoming HTTP messages using the appropriate mechanism for the HTTP version to prevent HTTP request smuggling. In HTTP/1.x, if a Transfer-Encoding header field is present, the Content-Length header must be ignored per RFC 2616. When using HTTP/2 or HTTP/3, if a Content-Length header field is present, the receiver must ensure that it is consistent with the length of the DATA frames. | 2 |
| **4.2.2** | Verify that when generating HTTP messages, the Content-Length header field does not conflict with the length of the content as determined by the framing of the HTTP protocol, in order to prevent request smuggling attacks. | 3 |
| **4.2.3** | Verify that the application does not send nor accept HTTP/2 or HTTP/3 messages with connection-specific header fields such as Transfer-Encoding to prevent response splitting and header injection attacks. | 3 |
| **4.2.4** | Verify that the application only accepts HTTP/2 and HTTP/3 requests where the header fields and values do not contain any CR (\r), LF (\n), or CRLF (\r\n) sequences, to prevent header injection attacks. | 3 |
| **4.2.5** | Verify that, if the application (backend or frontend) builds and sends requests, it uses validation, sanitization, or other mechanisms to avoid creating URIs (such as for API calls) or HTTP request header fields (such as Authorization or Cookie), which are too long to be accepted by the receiving component. This could cause a denial of service, such as when sending an overly long request (e.g., a long cookie header field), which results in the server always responding with an error status. | 3 |

## V4.3 GraphQL

GraphQL is becoming more common as a way of creating data-rich clients that are not tightly coupled to a variety of backend services. This section covers security considerations for GraphQL.

| # | Description | Level |
| :---: | :--- | :---: |
| **4.3.1** | Verify that a query allowlist, depth limiting, amount limiting, or query cost analysis is used to prevent GraphQL or data layer expression Denial of Service (DoS) as a result of expensive, nested queries. | 2 |
| **4.3.2** | Verify that GraphQL introspection queries are disabled in the production environment unless the GraphQL API is meant to be used by other parties. | 2 |

## V4.4 WebSocket

WebSocket is a communications protocol that provides a simultaneous two-way communication channel over a single TCP connection. It was standardized by the IETF as RFC 6455 in 2011 and is distinct from HTTP, even though it is designed to work over HTTP ports 443 and 80.

This section provides key security requirements to prevent attacks related to communication security and session management that specifically exploit this real-time communication channel.

| # | Description | Level |
| :---: | :--- | :---: |
| **4.4.1** | Verify that WebSocket over TLS (WSS) is used for all WebSocket connections. | 1 |
| **4.4.2** | Verify that, during the initial HTTP WebSocket handshake, the Origin header field is checked against a list of origins allowed for the application. | 2 |
| **4.4.3** | Verify that, if the application's standard session management cannot be used, dedicated tokens are being used for this, which comply with the relevant Session Management security requirements. | 2 |
| **4.4.4** | Verify that dedicated WebSocket session management tokens are initially obtained or validated through the previously authenticated HTTPS session when transitioning an existing HTTPS session to a WebSocket channel. | 2 |

## References

For more information, see also:

* [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
* Resources on GraphQL Authorization from [graphql.org](https://graphql.org/learn/authorization/) and [Apollo](https://www.apollographql.com/docs/apollo-server/security/authentication/#authorization-methods).
* [OWASP Web Security Testing Guide: GraphQL Testing](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/12-API_Testing/01-Testing_GraphQL)
* [OWASP Web Security Testing Guide: Testing WebSockets](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/11-Client-side_Testing/10-Testing_WebSockets)
