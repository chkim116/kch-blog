---
title: "useEffect를 잘못 쓰는 흔한 패턴"
date: 2026-04-05T00:00:00+09:00
updated: 2026-04-05T00:00:00+09:00
summary: "useEffect는 시점 제어가 역할이다. 콜백 안에 로직이 쌓이기 시작하면 읽기도, 수정하기도 어려워진다."
cover:
  image: ""
categories:
  - 개발
tags:
  - React
  - useEffect
  - 리팩터링
  - 가독성
slug: "use-effect-common-misuse-pattern"
draft: false
---

useEffect는 의존성 배열의 값이 바뀌면 콜백을 실행하는 구조다. 이 말은 "언제 호출할지"를 담당하고 있다는 뜻이다. 그러므로 콜백 안은 간결해야 한다. 상세한 구현이 들어오는 순간, "언제"와 "무엇"을 동시에 담당하게 된다.

코드에서 이런 패턴을 자주 본다. useEffect 내에 즉시 실행 함수를 호출하거나, 함수를 선언하고 그대로 호출하는 패턴이다.

```tsx
// 즉시실행함수 패턴
useEffect(() => {
  (async () => {
    const data = await fetchData(id);
    const filtered = data.filter(item => item.active);
    const sorted = filtered.sort((a, b) => b.createdAt - a.createdAt);
    setList(sorted);
  })();
}, [id]);

// 함수 선언 + 호출 패턴
useEffect(() => {
  const fetchAndSet = async () => {
    const data = await fetchData(id);
    const filtered = data.filter(item => item.active);
    const sorted = filtered.sort((a, b) => b.createdAt - a.createdAt);
    setList(sorted);
  };

  fetchAndSet();
}, [id]);
```

이 둘의 형태는 다르지만 문제는 같다. 너무 많은 일을 하고 있다는 것이다. 뭘 하는 건지 알려면 콜백 전체를 읽어야 한다. 그리고 deps가 하나라도 추가되는 날엔, 내부 로직이 복잡할수록 어떤 사이드 이펙트가 터질지 몰라 수정을 주저하게 된다.

useEffect에서 기대하는 건, 어떤 시점에 콜백 함수가 호출되느냐이지 콜백 함수가 무슨 일을 하느냐가 아니다.

---

## "언제"와 "무엇"을 나누면

어떻게 하면 될까. 단순하다. 구분하면 된다.

```tsx
async function fetchAndProcess(id: string) {
  const data = await fetchData(id);
  const filtered = data.filter(item => item.active);
  return filtered.sort((a, b) => b.createdAt - a.createdAt);
}

useEffect(() => {
  fetchAndProcess(id).then(setList);
}, [id]);
```

useEffect가 한 줄이 됐다. `id`가 바뀌면 `fetchAndProcess`를 실행한다. 그게 전부다. "무엇을 하는지"가 궁금하면 `fetchAndProcess`를 보면 된다. 굳이 useEffect를 읽을 필요가 없다.

수정할 때도 마찬가지다. 로직이 바뀌면 `fetchAndProcess`만, 타이밍이 바뀌면 useEffect만 건드리면 된다.

---

## 담당하는 일이 몇 가지인가

콜백 안에 계산 로직, 데이터 가공, 함수 선언이 들어오기 시작하면 useEffect는 시점 제어 이상의 일을 하게 된다. 하나만 하지 않는 순간, 읽기도 수정하기도 어려워진다.

다음에 useEffect를 작성할 때 한번 세어보자. 이 안에서 담당하는 일이 몇 가지인지.

별 거 아닌 것 같지만, 이런 것들이 쌓이면 코드는 조금씩 읽기 어려워진다.
