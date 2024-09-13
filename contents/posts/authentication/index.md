---
title: "인증과 인가"
description: "세션유지방식과 이에 대한 활용을 정리"
date: 2024-07-20
update: 2024-07-20
tags:
  - session
  - token
  - cookie
series: "network"
---

## 인증과 인가

인증 : 사용자의 신원을 확인하는 것
인가 : 신원이 확인된 사용자에게 접근 권한을 부여하는 것.

Http는 비연결성, 무상태성을 가진다. 이전 요청을 기억하지 못하기 때문에 한 번 인증, 인가가 되었다고 해서 사용자의 세션(사용자의 이전 상태를 기억하는 지속된 연결 경험)이 마법처럼 유지되지 못한다.

이를 위해 다양한 방식으로 세션을 유지하는 방법들이 있다.

## 세션 유지 방식

- url

`hiyen.com/홍길동/mypage...` 와 같이 url을 기준으로 유저에게 맞는 웹 페이지를 제공하는 방식이다.  당연하게도 url만 알면 모두가 유저 개인페이지에 접근할 수 있고 url기반이기 때문에 정보를 가로채는 것도 아주 쉽다.

url은 브라우저 히스토리, 북마크 등에 저장되어 쉽게 노출되며 referer header등에 남아 서버 간 쉽게 공유되기 때문에 거의 사용되지 않는다.

- 쿠키

홍길동이 id와 password로 로그인을 하면 사용자가 인증되었다는 정보 서버에서 쿠키로 만들어 응답한다. 브라우저는 해당 쿠키를 이후 요청마다 http요청에 포함하고 서버에서는 쿠키를 보고 개인에 맞는 서비스를 실행한다.

쿠키는 클라이언트 쪽에 저장되기 때문에 탈취, 조작 당하기 쉽고,  이를 당해도 서버에서 이를 구분할 방법이 없어 보안상 문제가 생길 수 있다.

예를 들어 크롬에서 관리자 도구만 켜도 웹 사이트에서 받은 쿠키를 볼 수 있다. 쿠키의 값이 11이라면, 이를 12로만 바꾸어도 다른 사용자의 정보를 볼 수 있을 지도 모른다. 따라서 쿠키를 인증과 인가에서 사용한다면 예측할 수 없는 암호화, 주요 정보는 쿠키에 저장하지 않아야 한다.

- 헤더

헤더에 사용자의 개인정보를 넣는 방식은 쿠키보다 안전하다.

헤더는 기본적으로 클라이언트 자바 스크립트에서 접근할 수 없고(쿠키는 documents.cookies로 바로 접근 가능) 명시적으로 헤더를 사용할 때만 포함할 수 있기 때문에 모든 요청에 자동으로 포함되는 쿠키보다 노출되는 위험이 줄어든다.

사용자의 세션을 유지하기 위해서는 위의 세가지 방법 중 하나가 필요하다.

## 세션 유지 방식의 활용

- 서버내에서 세션 관리 + 쿠키 or 헤더에 식별자를 주기

서버가 세션을 관리한다. 상태를 유지한다고 말한다. 클라이언트는 세션 식별자를 쿠키나 헤더에 포함시켜 서버에 요청을 보낸다.

1. 로그인 시 서버는 세션에 관련된 정보를 저장(대개 DB, 세션 스토리지라 불림)하고 이에 대한 식별자(대개 세션ID)를 쿠키나 헤더에 넣어 클라이언트에 응답한다.
2. 클라이언트는 이후 요청마다 이 세션ID를 쿠키or 헤더에 포함하여 보낸다.
3. 서버는 세션ID를 통해 세션 스토리지를 조회하고 유효성을 확인, 사용자에 맞는 응답을 제공한다.

이 방식은 사용자의 개인정보를 서버 내에서 관리하기 때문에 상대적으로 보안이 취약한 클라이언트에 저장할 필요가 없어 보안상 이점이 있다.

또한 세션ID가 갈취당하거나 조작된 경우 이에 대해 서버 내에서 해당 세션을 무효화하는 것이 쉽다.

이 방식에서 주의할 것은 클라이언트로 보내지는 세션 ID가 암호화되어야 한다는 점이고 거의 모든 요청에서 세션 스토리지 조회가 일어나 성능에 영향이 간다는 점, 또한 서버가 수평확장을 할 경우 세션 불일치 문제가 생길 수 있다는 점이다.

- 서버는 stateless + 토큰 사용

서버는 상태를 유지하지 않고 클라이언트가 요청마다 인증 정보를 포함시켜 서버에 보내는 형식이다. 주로 JWT(Json Web Token)가 사용된다.

1. 로그인 시 서버는 토큰을 생성해 클라이언트에 응답한다.
2. 이후 요청마다 클라이언트는 토큰을 헤더에 포함하여 서버에 보낸다.
3. 서버는 토큰을 검증하고 유효한 토큰이면 요청을 처리한다.

일견 보면 위의 방식과 별 차이가 없어 보이지만 서버에서 세션 스토리지를 운영, 조회하는 과정을 하지 않는다는 차이점이 있다.

JWT를 예로 들자면 JWT는 헤더,페이로드, 시그니처로 이루어져 있는데 헤더와 시그니처는 토큰의 무결성을 보장하고 페이로드는 Claims이라고 불리는 json 형식의 데이터를 지원하여 해당 부분에 세션 유지를 위한 사용자의 개인정보를 넣을 수 있다.

이 방식은 서버가 상태를 유지하지 않으므로 수평으로 확장하는데 아무런 문제가 없다. 또한 세션 스토리지를 운영할 필요가 없으므로 시스템 설계의 복잡도가 줄어들 수 있다. (토큰 발급, 유효성 검사 로직만 각 서버에 있으면 됨)

단점은 장점과 같이 stateless하다는 점, 즉, 서버가 토큰에 대한 제어권이 없다는 점이다. 토큰은 만료될 때까지 유효하기 때문에 이를 갈취당해도 서버 내에서 무효화할 방법이 없다.

이를 방지하기 위해서 만료시간을 짧게 하면 사용자의 잦은 재로그인이 필요하고 만료시간이 길면 보안에 취약해 진다.

- Access Token + Refresh Token

토큰 방식의 문제를 해결하기 위해 많이 사용하고 있는 방식이다. Access Token + Refresh Token으로 토큰을 두 개로 운영하며 Access Token은 만료 시간을 짧게 설정하여 탈취 당해도 피해를 최소화할 수 있게하고 Refresh Token은 Access Token과 함께 발행하는 비교적 만료 시간이 긴 토큰으로 이 토큰을 가지고 있다면 새로운 Access Token을 재발급 받을 수 있게 한다.

하지만 Refresh Token 자체가 탈취당할 가능성도 있다. 즉, stateless하다는 토큰의 장점을 계속 가지고 싶다면 보안을 일정 수준 포기해야 한다.

이에 따라 토큰 방식을 사용하되 토큰에 대한 관리를 서버에서 하는 방법이 사용되기도 한다. 토큰 블랙리스트를 운영하는 방법, Refresh Token을 발급 할때마다 이전 Refresh 토큰을 무효화 리스트에 추가하는 방법등이 있다. 이러한 방법들은 전부 서버내의 스토리지를 요구하기 때문에 수평적 확장을 할 시 고려해야 한다.

## Oauth

사용자의 다른 웹사이트에 있는 정보에 대해 접근 권한을 부여해 주는 프로토콜.

구글캘린더의 일정을 받아 온다고 치자. 사용자는 구글 id 패스워드를 우리의 hiyen닷컴에 제공하는 것을 신뢰할 수 없다. 우리 서버도 다른 서비스의 id 패스워드를 관리하는 것이 부담스럽다.

따라서 프로토콜을 통해 사용자는 구글에 인증을 하고 사용자의 구글 정보에 대한 접근 권한을 히엔닷컴에 주는 방식이다 우리의 서버는 이제 이 접근 권한으로 사용자의 구글 캘린더를 구글에서 받아올 수 있다.

용어 정리

1. Resource Owner(사용자) - 자원에 접근 권한을 가지고 있는 사용자. 홍길동
2. Clinet(클라이언트) - 홍길동 대신 구글 캘린더에 접근하려는 서버. 히앤닷컴
3. Authorization Server(인증 서버) - 클라이언트를 인증, 인가 코드 발급, 액세스 토큰 발급
4. Resource Server(자원 서버) - 홍길동의 구글 캘린더를 가지고 있는 서버.

Oauth과정

1. 홍길동이 로그인을 요청한다.
2. 히앤닷컴은 구글에 미리 등록했던 Client Id, Redirect URI, Scope를 가지고 인증서버에 사용자가 로그인요청을 했음을 알린다
3. 인증 서버는 홍길동에게 로그인 페이지를 제공한다.
4. 홍길동이 id, password로 로그인을 한다.
5. 인증서버는 홍길동에게 인증 코드(Authorization code)를 준다
6. 홍길동은 Redirect URI로 리다이렉트 된다 이때 인증코드가 같이 히앤닷컴에 전달된다.
7. 히앤닷컴은 이제 인증코드로 인증서버에 접근 권한(여기서는 Access Token)을 요구한다
8. 인증서버는 access token을 히앤닷컴에 준다
9. 히앤닷컴은 이 access token을 어딘가에 저장한다
10. 홍길동이 구글 캘린더를 사용하는 히앤닷컴 서비스를 요청한다.
11. 히앤닷컴은 이제 자원서버에 토큰을 가지고 요청을 보낸다
12. 자원서버는 토큰을 확인하고 자원을 준다.
13. 히앤닷컴은 이제 사용자에게 구글 캘린더를 사용하는 서비스를 제공할 수 있다!!