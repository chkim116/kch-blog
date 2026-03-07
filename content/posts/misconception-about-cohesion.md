---
title: "응집도에 대한 오해"
date: 2026-03-07T12:00:00+09:00
updated: 2026-03-07T12:00:00+09:00
summary: "응집도의 진짜 기준은 유사성이 아니라 변화축이다."
cover:
  image: ""
categories:
  - 개발
tags:
  - 응집도
  - 변화축
  - SRP
  - 폴더구조
slug: "misconception-about-cohesion"
draft: false
---

개발을 배우면서 응집도에 대해 이런 말을 들어본 적이 있을 것이다.

"관련된 코드는 같은 곳에 두어라."

틀린 말은 아니다. 그런데 이 말에는 중요한 것이 빠져 있다. **무엇을 기준으로 관련이 있다고 판단하는가.** 이 기준이 없으면 응집도를 높이려다가 오히려 유지보수하기 어려운 코드를 만들게 된다.

이런 코드를 본 적이 있을 것이다.

```ts
// guardUtils.ts
export function authGuard() { ... }       // 인증 Guard
export function permissionGuard() { ... } // 권한 Guard
export function validationGuard() { ... } // 데이터 유효성 Guard
export function routeGuard() { ... }      // 라우팅 Guard
```

응집도가 높아 보인다. 같은 역할을 하는 코드들이 한곳에 있으니까.

그런데 이 파일을 실제로 유지보수해보면 이상하다. 인증 로직이 바뀌면 이 파일을 열어야 하고, 권한 정책이 바뀌어도 이 파일을 열어야 한다. 데이터 유효성 기준이 바뀌어도, 라우팅 규칙이 바뀌어도 마찬가지다. 변경 이유가 네 개다.

이게 문제의 핵심이다.

### 응집도의 진짜 기준

응집도는 **변화축이 같은 것들을 모으는 것**이다.

변화축이란 "이 코드가 변경되는 이유"다. Robert Martin이 SRP(단일 책임 원칙)에서 말한 것과 같은 개념이다. "클래스를 변경해야 하는 이유는 오직 하나뿐이어야 한다."

변경 이유가 같다 = 변화축이 같다 = 함께 두어야 한다.
변경 이유가 다르다 = 변화축이 다르다 = 분리해야 한다.

`authGuard`와 `permissionGuard`는 둘 다 Guard지만 변경 이유가 다르다. 인증 정책이 바뀌면 `authGuard`가 바뀌고, 권한 정책이 바뀌면 `permissionGuard`가 바뀐다. 변화축이 다르다. 분리해야 한다.

```ts
// auth/guard.ts — 인증 관련만
export function authGuard({ children }: Props) {
  const token = localStorage.getItem('auth_token');
  if (!token) { return <LoginPage />; }
  return <>{children}</>;
}

// permission/guard.ts — 권한 관련만
export function permissionGuard({ role, children }: Props) {
  const { userRole } = useUserRole();
  if (userRole !== role) { return <ForbiddenPage />; }
  return <>{children}</>;
}
```

### 폴더 구조에도 적용된다

이 개념은 함수 단위에만 적용되는 게 아니다. 폴더 구조에도 그대로 적용된다.

기능별로 폴더를 나누는 방식(`components/`, `hooks/`, `utils/`)은 코드의 타입으로 묶은 것이다. 타입이 같다고 변화축이 같은 건 아니다. `hooks/` 폴더 안에 있는 `useAuth`, `useCart`, `useTheme`는 모두 hook이지만, 변경 이유는 전혀 다르다.

```
// 타입 기준 구조 — 변화축이 섞임
hooks/
  useAuth.ts
  useCart.ts
  useTheme.ts

// 도메인 기준 구조 — 변화축이 모임
auth/
  useAuth.ts
cart/
  useCart.ts
```

물론 도메인별 구조가 항상 정답은 아니다. 중요한 건 구조 자체가 아니라, **이 코드가 변경되는 이유가 하나인가**이다. 파일 크기나 줄 수는 기준이 될 수 없다. 크기는 변화축을 제대로 나눴을 때 나오는 결과일 뿐이다.

### 변화축으로 판단하기

응집도를 판단할 때 이 질문 하나를 던져보면 된다.

> **"이 코드가 변경되는 이유는 몇 개인가?"**

하나면 응집도가 높은 것이다. 여러 개라면 분리 신호다.

이름이 같다고 묶는 게 아니다. 같은 이유로 바뀌는 것들을 묶는 것이다.
