---
title: "API 호출 시 JSON 대신 index.html이 반환되는 문제"
date: 2026-03-29T16:12:12+09:00
description: "Nginx 리버스 프록시 환경에서 API 호출 시 JSON 대신 SPA index.html이 반환되는 문제를 분석하고 원인과 해결 방법을 정리한 트러블슈팅 기록"
categories:
  - Troubleshooting
tags:
  - nginx
  - reverse-proxy
  - api
  - spa
  - troubleshooting
  - backend
draft: false
---

개발서버에 반영 후 API 테스트 중 이상한 현상을 경험했습니다.

API 엔드포인트를 호출했지만 JSON 응답이 아니라 다음과 같은 HTML이 반환되었습니다.

```html
<!DOCTYPE html>
<html>
  <head>
  <title>Web-site</title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but this web-site doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
  </body>
</html>
```

HTTP 상태 코드는 `200 OK`였습니다.  
정상 응답처럼 보였지만 실제로는 API 요청에 대해 **완전히 잘못된 응답**이었습니다.

더 이상했던 점은 이 서비스가 **웹 페이지를 제공하지 않는 REST API 서버**라는 점이었습니다.

## 문제 파악

에러 처리 로직에서 요청 정보를 로그로 확인했습니다.

```java
request.getRequestURI()
```

로그 결과

```
http://domain/api
```

하지만 실제 요청은 다음과 같았습니다.

```
https://domain:8007/api
```

즉 다음 정보가 바뀌어 있었습니다.

| 항목     | 실제 요청 | 서버 인식 |
| ------ | ----- | ----- |
| scheme | https | http  |
| port   | 8443  | 없음    |

또한 응답으로 반환된 HTML을 확인해 보니,  
우리 서비스가 아닌 같은 인프라에 있는 **다른 웹 서비스의 SPA 페이지**였습니다.

이로 인해 요청이 프록시 환경에서 다른 서비스로 라우팅되고 있을 가능성을 의심하게 되었습니다.

## 원인

서비스 앞단에는 리버스 프록시가 존재했고 동일한 프록시에서 여러 서비스가 운영되고 있었습니다.

대략적인 구조는 다음과 같습니다.

```
Client
  ▼
Nginx
  │
  ├── API 서비스
  └── SPA 웹 서비스
```

프록시를 거치면서 요청의 **scheme**과 **port** 정보가 변경되었고, 이로 인해 라우팅이 예상과 다르게 동작하면서 다른 서비스로 전달되는 상황이 발생했습니다.

## 해결 방법

프록시 환경에서는 원래 요청 정보를 백엔드 서버에 전달하기 위해
다음과 같은 헤더를 전달하는 것이 중요합니다.

```nginx
location /api/ {

    proxy_pass http://backend;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;

    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Port $server_port;

}
```

이와 같은 설정을 통해 백엔드 서버가 클라이언트의 실제 요청 정보를 정확하게 인식할 수 있도록 설정을 검토했습니다.

---

이번 문제는 Reverse Proxy 환경에서 요청 정보(`scheme`, `port`)가 변경되면서 발생했습니다.

그 결과 API 요청이 동일 프록시 내부의 다른 SPA 서비스로 라우팅되었고, API 응답 대신 `index.html`이 반환되는 현상이 나타났습니다.

리버스 프록시 환경에서 API 서버를 운영할 때는 다음 사항을 반드시 확인해야 합니다.

- `X-Forwarded-*` 헤더 전달 여부
- 프록시 라우팅 설정
- 백엔드 서버의 프록시 환경 처리 방식

프록시 환경에서는 요청 경로뿐만 아니라 요청 정보(scheme, host, port)가 어떻게 전달되는지까지 함께 확인하는 것이 중요하다는 점을 다시 한 번 느낄 수 있었습니다.
