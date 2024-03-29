# import NextAuth from "next-auth"

Next.js 프레임워크에 최적화된 인증 라이브러리로, 사용자 인증과 관련된 기능을 쉽고 빠르게 구현할 수 있도록 도와준다.
주로 OAuth와 같은 인증 플로우를 간단한 설정과 몇 줄의 코드로 쉽게 추가할 수 있게 해주는 것이 주요 특징이다.

# 특징

- 간편한 설정: next-auth는 Google, Facebook, Twitter 등과 같은 여러 인기 있는 소셜 로그인 제공자들과의 통합을 쉽게 설정할 수 있도록 해준다.

- 세션 관리: 사용자 세션을 생성하고 관리하는 기능을 내장하고 있어, 개발자가 별도로 세션 관리 로직을 작성할 필요가 없다.

- JWT 토큰 지원: JSON Web Tokens (JWT)를 사용하여 보안적으로 안전한 인증 방식을 제공한다.

- 커스터마이징 가능: 기본 제공되는 기능 외에도, 커스텀 인증 공급자, 콜백, 데이터베이스 모델 등을 통해 필요에 맞게 확장하고 맞춤 설정할 수 있다.

- Next.js와의 통합: Next.js의 서버사이드 렌더링(SSR) 및 정적 생성(SSG) 기능과 잘 통합되어, 인증 상태를 이용한 페이지 렌더링 최적화가 가능하다.

# 직접 구현과 차이

1. 구현의 복잡성

- Next-Auth 사용 : OAuth인증을 쉽게 구현할 수 있도록 해주며, 대부분의 설정이 사전에 구성되어 있다. 세션 관리, 사용자 인증, 토큰 취급 등의 기능이 내장되어있다.
- 직접 구현 : OAuth 인증 플로우를 직접 구현하면 더 많은 코드를 작성해야 하며 인증 프로세스의 모든 단계를 직접 관리해야한다.

2. 커스터마이징과 확장성

- Next-Auth 사용 : 많은 OAuth 제공자와 통합을 지원, 커스텀 인증 로직 추가 가능, But 특정 로직 추가시 라이브러리의 기본 동작과 충돌할 수 있다.
- 직접 구현 : OAuth 플로우를 완전히 제어할 수 있으며, 기존 시스템이나 복잡한 요구 사항에 맞게 인증 시스템을 완전히 맞춤 설정할 수 있다.

3. 보안과 JWT

- Next-Auth 사용 : 세션 관리를 위해 JWT를 내부적으로 사용할 수 있으며, 이는 안전하고 효율적인 사용자 인증 방법을 제공한다.
- 직접 구현 : JWT를 포함한 인증 토큰의 생성, 검증, 취급 방법을 직접 설계하고 구현해야 한다.

# calendar App 프로젝트에서 next-auth를 사용하지 않았다. 왜?

<img width="928" alt="image" src="https://github.com/Hoodie-Project/frontend/assets/70371342/0afce513-f035-4930-911e-90686dd6cfa1">

우리는 이러한 REST API 전략을 세웠었고, 우리 프로젝트 자체 서버에서 Google 서버에 요청을 보내 유저 정보를 조회를 하여 JWT 인증 토큰을 생성하고 생성한 토큰을 클라이언트로 넘겨주기로 결정했었다.

사실 next-auth에 대해 몰랐을 때 이미 로그인 전략을 세웠었고, next-auth를 사용하지 않고 OAuth를 구현중에 있다. 우리 프로젝트의 서버에서 직접 JWT로그인 방식을 구현하는 것이 서로 공부가 될 것 같다 느껴서 로그인 전략을 뒤엎진 않을 것이다. 그런데 충분히 사용해 볼 가치가 있다 느껴져서 추후 사용 방법에 대해 공부해봐야겠다.

[Next-Auth를 사용하여 손쉽게 OAuth기반 권한관리하기 + RefreshToken + Private Route](https://jeongyunlog.netlify.app/develop/nextjs/next-auth/)
