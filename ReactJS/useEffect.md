# useEffect

```JavaScript
useEffect(() => {
  // Callback 함수
}, []); // 의존성 배열(Dependency Array)
```

의존성 배열 내에 들어있는 값이 변화하면 콜백 함수가 수행된다.

# 함수 컴포넌트 Life Cycle useEffect

리액트의 컴포넌트에는 라이프사이클이 존재하고, 각각 작업을 수행시킬 수 있다는 것을 **라이프 사이클을 제어한다**고 말한다.

- 컴포넌트 탄생

  - 화면에 나타나는 것(Mount)
  - ex) 초기화 작업

- 컴포넌트 변화

  - 업데이트(Update)
  - ex) 예외 처리 작업

- 컴포넌트 죽음

  - 화면에서 사라짐(UnMount)
  - ex) 메모리 정리 작업

클래스 컴포넌트에서는 `componentWillMount`, `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`를 컴포넌트 당 한 번씩만 사용했다.
<br/>
함수형 컴포넌트에서는 저런 메서드들을 사용할 수 없다.

함수 컴포넌트를 사용하면 useEffect를 빼놓을 수 없다.
함수 컴포넌트는 특정 데이터에 대해서 라이프 사이클이 진행된다.
<br/>

```javascript
useEffect(() => {}); // 컴포넌트 리렌더링마다 코드 실행
useEffect(() => {}, []); // mount시 첫 1회 코드 실행
useEffect(() => {}, [의존성1, 의존성2, ..]); // 조건부 effect 발생 : 의존성 배열 바뀔 때
useEffect(() => { return () => {}}); // unmount시 1회 실행
```

# eslint "react-hooks/exhaustive-deps" 규칙

useEffect 내부에서 실행된 함수에서 사용되는 변수를 useEffect의 의존성 배열 안에 넣어주지 않으면 경고가 뜬다.

### 번거롭게 왜 있는걸까?

1. 함수의 예측 가능성 보장

   React 컴포넌트는 렌더링될 때마다 다른 함수 스코프를 가진다. 'useEffect'내에서 사용된 변수나 함수가 외부 스코프에 있으면, 이 변수나 함수가 변경될 때 'useEffect'가 재실행되지 않을 수 있다. 이 규칙은 모든 의존성이 명시적으로 나열되게 만들어서 useEffect가 항상 최신 상태의 데이터를 사용하도록 한다.

2. 버그 방지

   의존성 배열을 올바르게 관리하지 않으면 useEffect 내부의 로직이 오래된 데이터나 상태에 기반하여 실행될 수 있다. 이는 버그로 이어질 수 있으며, 이 규칙은 이러한 종류의 버그를 미연에 방지한다.

3. 성능 최적화

   의존성 배열에 변수를 정확히 명시함으로 useEffect는 해당 변수가 변경될 떄만 재실행된다. 불필요한 재실행을 방지함으로 앱의 성능을 향상시킬 수 있다.

# useEffect Definition from 모던 리액트 딥다이브

애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 매커니즘이다.
이 부수 효과가 '언제' 일어나는지보다 어떤 상태값과 함께 실행되는지 살펴보는 것이 중요하다.

### useEffect는 어떻게 의존성 배열이 변경된 것을 알고 실행할까?

useEffect는 자바스크립트의 proxy나 데이터 바인딩, 옵저버 같은 특별한 기능을 통해 값의 변화를 관찰하는 것이 아니고 렌더링할 때마다 의존성에 있는 값을 보면서 이 의존성의 값이 이전과 다른 게 하나라도 있으면 부수 효과를 실행하는 평범한 함수라 볼 수 있다.
따라서 useEffect는 state와 props의 변화 속에서 일어나는 렌더링 과정에서 실행되는 부수 효과 함수라고 볼 수 있다.

### 클린업 함수의 목적

일반적으로 이벤트를 등록하고 지울 때 사용해야 한다고 알려져 있다.

```jsx
import { useState, useEffect } from 'react';

export default function App() {
  const [counter, setCounter] = useState(0);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  // 최초 실행
  useEffect(() => {
    function addMouseEvent() {
      console.log(counter);
    }

    window.addEventListener('click', addMouseEvent);

    // 클린업 함수
    // 그리고 이 클린업 함수는 다음 렌더링이 끝난 뒤에 실행된다.
    return () => {
      console.log('클린업 함수 실행!', counter);
      window.removeEventListener('click', addMouseEvent);
    };
  }, [counter]);

  return (
    <>
      <h1>{counter}</h1>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

```
클린업 함수 실행! 0
1

클린업 함수 실행! 1
2

클린업 함수 실행! 2
3

클린업 함수 실행! 2
4
```

초기 렌더링 시 counter는 0이다. useEffect가 처음 실행될 때, 클린업 함수는 counter의 현재값인 0을 기억한다.
counter가 1로 증가하면, useEffect는 다시 실행된다. 이때, 이전 useEffect실행에서 생성된 클린업 함수가 실행되고, 기억된 counter값 0을 사용하여 클린업 함수 실행 0을 콘솔에 출력한다. 그 후 새로운 useEffect가 실행되고 이번에는 counter 값 1을 기억하는 새로운 클린업 함수가 생성된다.<br/>

이 패턴은 counter가 증가할 때마다 반복된다. 각각의 useEffect 실행은 그 시점의 counter값을 기억하는 새로운 클린업 함수를 생성한다. 결국, 클린업 함수는 useEffect가 마지막으로 실행됐을 때의 상태값을 기반으로 작동한다. 이는 useEffect의 실행과 클리닝 사이에 상태가 변경되어도, 클린업 함수는 항상 그것이 생성됐을 때의 상태를 참조한다는 것을 의미한다.

> 클린업 함수는 이전 counter값, 즉 이전 state를 참조해 실행된다는 것을 알 수 있다. 여기서 중요한 것은 **클린업 함수는 비록 새로운 값을 기반으로 렌더링 뒤에 실행되지만 이 변경된 값을 읽는 것이 아니라 함수가 정의됐을 당시에 선언됐던 이전 값을 보고 실행된다는 것이다.**

이 사실을 종합해보면 함수형 컴포넌트의 useEffect는 그 콜백이 실행될 때마다 이전의 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤에 콜백을 실행한다. 따라서 이벤트를 추가하기 전에 이전에 등록했던 이벤트 핸들러를 삭제하는 코드를 클린업 함수에 추가하는 것이다. 이렇게 함으로써 특정 이벤트의 핸들러가 무한히 추가되는 것을 방지할 수 있다.

> 이처럼 클린업 함수는 생명주기 메서드의 언마운트 개념과는 조금 차이가 있는 것을 볼 수 있다. 언마운트는 특정 컴포넌트가 DOM에서 사라진다는 것을 의미하는 클래스형 컴포넌트의 용어다. 클린업 함수는 언마운트라기보다는 함수형 컴포넌트가 리렌더링됐을 때 의존성 변화가 있었을 당시 이전의 값을 기준으로 실행되는, 말 그대로 이전 상태를 청소해 주는 개념으로 보는 것이 옳다.

## 의존성 배열 없는 useEffect가 매 렌더링마다 실행된다면 useEffect없이 써도 되지 않을까요?

```jsx
// 1
function 컴포넌트1() {
  console.log('렌더링 됨');
}

// 2
function 컴포넌트2() {
  useEffect(() => {
    console.log('렌더링 됨');
  });
}
```

1. 서버 사이드 렌더링 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해 준다. useEffect 내부에서는 window 객체의 접근에 의존하는 코드를 사용해도 된다.
2. useEffect는 컴포넌트 렌더링의 부수 효과, 즉 컴포넌트 렌더링이 완료된 이후에 실행된다. 반면 직접 실행은 컴포넌트가 렌더링되는 도중에 실행된다. 따라서 1번과는 달리 서버 사이드 렌더링의 경우에 서버에서도 실행된다. 그리고 이 작업은 함수형 컴포넌트의 반환을 지연시키는 행위다. 즉, 무거운 작업일 경우 렌더링ㅇ르 방해하므로 성능에 악영향을 미칠 수 있다.

## useEffect의 구현

```jsx
const MyReact = (function () {
  const global = {};
  let index = 0;

  function useEffect(callback, dependencies) {
    const hooks = global.hooks;

    // 이전 훅 정보가 있는지 확인한다.
    let previousDependencies = hooks[index];

    // 변경됐는지 확인
    // 이전 값이 있다면 이전 값을 얕은 비교로 비교해 변경이 일어났는지 확인한다.
    // 이전 값이 없다면 최초 실행이므로 변경이 일어난 것으로 간주해 실행을 유도한다.
    let isDependenciesChanges = previousDependencies
      ? dependencies.some(
          (value, idx) => !Object.is(value, previousDependencies[idx]),
        )
      : true;

    // 변경이 일어났다면 첫 번째 인수인 콜백 함수를 실행한다.
    if (isDependenciesChanged) {
      callback();
    }

    // 현재 의존성을 훅에 다시 저장한다.
    hooks[index] = dependencies;

    // 다음 훅이 일어날 때를 대비하기 위해 index를 추가한다.
    index++;
  }

  return { useEffect };
})();
```

핵심은 의존성 배열의 이전 값과 현재 값의 얕은 비교다. 리액트는 값을 비교할 때 `Object.is`를 기반으로 하는 얕은 비교를 수행한다. 이전 의존성 배열과 현재 의존성 배열의 값에 하나라도 변경 사항이 있다면 callback으로 선언한 부수 효과를 실행한다.

## 왜 useEffect의 콜백 인수로 비동기 함수를 바로 넣을 수 없을까?

만약 useEffect의 인수로 비동기 함수가 사용 가능하다면 비동기 함수의 응답 속도에 따라 결과가 이상하게 나타날 수 있다. 극단적 예제로 이전 state기반의 응답이 10초가 걸렸고, 이후 바뀐 state기반의 응답이 1초 뒤에 왔다면 이전 state 기반으로 결과가 나와버리는 불상사가 생길 수 있다. 이러한 문제를 useEffect의 경쟁 상태(race condition)라고 한다.

```tsx
useEffect(async () => {
  // useEffect에 async 함수를 넘겨주면 다음과 같은 에러가 발생한다.
  // Effect callbacks are synchronous to prevent race conditions.
  // Put the async function inside:
  const response = await fetch('http://some.data.com');
  const result = await response.json();
  setData(result);
}, []);
```

useEffect의 인수로 비동기 함수를 지정할 수 없는 것이지, 비동기 함수 실행 자체가 문제가 되는 것은 아니다. useEffect 내부에서 비동기 함수를 선언해 실행하거나, 즉시 실행 비동기 함수를 만들어서 사용하는 것은 가능하다.

```tsx
useEffect(() => {
  let shouldIgnore = false;

  async function fetchData() {
    const response = await fetch('http://some.data.com');
    const result = await response.json();
    if (!shouldIgnore) setData(result);
  }

  fetchData();

  return () => {
    // shouldIgnore를 이용해 useState의 두 번째 인수를 실행을 막는 것뿐만 아니라
    // AbortController를 활용해 직전 요청 자체를 취소하는 것도 좋은 방법이 될 수 있다.
    shouldIgnore = true;
  };
}, []);
```

다만 비동기 함수가 내부에 존재하게 되면 useEffect 내부에서 비동기 함수가 생성되고 실행되는 것을 반복하므로 클린업 함수에서 이전 비동기 함수에 대한 처리를 추가하는 것이 좋다. fetch의 경우 abortController 등으로 이전 요청을 취소하는 것이 좋다.
즉, 비동기 useEffect는 state의 경쟁 상태를 야기할 수 있고 cleanup 함수의 실행 순서도 보장할 수 없기 때문에 개발자의 편의를 위해 useEffect에서 비동기 함수를 인수로 받지 않는다고 볼 수 있다.

## useEffect를 사용할 때 주의할 점

### 1. eslint-disavle-line react-hooks/exhaustive-deps 주석은 최대한 자제해라

위에서 언급한 ESLint의 react-hooks/exhaustive-deps룰이다.이 룰은 useEffect 인수 내부에서 사용하는 값 중 의존성 배열에 포함돼 있지 않은 값이 있을 때 경고를 발생시킨다.
정말로 필요한 때에는 사용할 수도 있지만 대부분의 경우에는 의도치 못한 버그를 만들 가능성이 크다.

이 코드를 사용하는 대부분의 예제가 빈 배열 []을 의존성으로 할 때, 즉 컴포넌트를 마운트하는 시점에만 무언가를 하고 싶다라는 의도로 작성하곤 한다. 그러나 이는 클래스형 컴포넌트의 생명주기 메서드인 componentDidMount에 기반한 접근법으로, 가급적이면 사용해선 안 된다.

useEffect는 반드시 의존성 배열로 전달한 값의 변경에 의해 실행돼야 하는 훅이다. 그러나 의존성 배열을 넘기지 않은 채 콜백 함수 내부에서 특정 값을 사용한다는 것은, 이 부수 효과가 실제로 관찰해서 실행돼야 하는 값과는 별개로 작동한다는 것을 의미한다. 즉, 컴포넌트의 state, props와 같은 어떤 값의 변경과 useEffect의 부수 효과가 별개로 작동하게 된다는 것이다. useEffect에서 사용한 콜백 함수의 실행과 내부에서 사용한 값의 실제 변경 사이에 연결 고리가 끊어져 있는 것이다.

- 예제

```jsx
function Component({ log }: { log: string }) {
  useEffect(() => {
    logging(log);
  }, []); // eslint-disable-line react-hooks/exhaustive-deps
}
```

위 코드는 log가 최초로 props로 넘어와서 컴포넌트가 최초로 렌더링된 시점에만 실행된다. 코드를 작성한 의도는 아마도 해당 컴포넌트가 최초로 렌더링 됐을 때만 logging을 실행하고 싶어서다.

위 코드의 위험성은 log가 아무리 변하더라도 useEffect의 부수 효과는 실행되지 않고, useEffect의 흐름과 컴포넌트의 props.log의 흐름이 맞지 않게 된다. 따라서 logging이라는 작업은 log를 props로 전달하는 부모 컴포넌트에서 실행되는 것이 옳을지도 모른다. 부모 컴포넌트에서 Component가 렌더링되는 시점을 결정하고 이에 맞게 log 값을 넘겨준다면 useEffect의 해당 주석을 제거해도 위 예제 코드와 동일한 결과를 만들 수 있고 컴포넌트의 부수 효과 흐름을 거스르지 않을 수 있다.

> useEffect에 빈 배열을 넘기기 전에는 정말로 useEffect의 부수 효과가 컴포넌트의 상태와 별개로 작동해야만 하는지, 혹은 여기서 호출하는 게 최선인지 한 번 더 검토해 봐야 한다.

### 2. useEffect의 첫 번째 인수에 함수명을 부여하라

useEffect를 사용하는 많은 코드에서 useEffect의 첫 번째 인수로 익명 함수를 넘겨준다.

```jsx
useEffect(() => {
  logging(user.id);
}, [user.id]);
```

useEffect의 수가 적거나 복잡성이 낮다면 이러한 익명 함수를 사용해도 큰 문제는 없다. 그러나 useEffect의 코드가 복잡하고 많아질수록 무슨 일을 하는 useEffect코드인지 파악하기 어려우니 기명 함수로 바꾸는 것이 좋다.

```jsx
useEffect(
  function logActiveUser() {
    logging(user.id);
  },
  [user.id],
);
```

### 3. 거대한 useEffect를 만들지 마라

가능한 한 useEffect는 간결하고 가볍게 유지해라. 만약 부득이하게 큰 useEffect를 만들어야 한다면 적은 의존성 배열을 사용하는 여러 개의 useEffect로 분리하는 것이 좋다. 만약 의존성 배열에 불가피하게 여러 변수가 들어가야 하는 상황이라면 최대한 useCallback과 useMemo등으로 사전에 정제한 내용들만 useEffect에 담아두는 것이 좋다.

### 4. 불필요한 외부 함수를 만들지 마라

useEffect가 실행하는 콜백 또한 불필요하게 존재해서는 안된다.

```tsx
function Component({ id }: { id: string }) {
  const [info, setInfo] = useState<number | null>(null);
  const controllerRef = useRef<AbortController | null>(null);
  const fetchInformation = useCallback(async (fetchId: string) => {
    controllerRef.current?.abort();
    controllerRef.current = new AbortController();

    const result = await fetchInfo(fetchId, { signal: controllerRef.signal });
    setInfo(await result.json());
  }, []);

  useEffect(() => {
    fetchInformation(id);
    return () => controllerRef.current?.abort();
  }, [id, fetchInformation]);
  return <div>{/* 렌더링 */}</div>;
}
```

이 컴포넌트는 props를 받아서 그 정보를 바탕으로 API호출을 하는 useEffect를 가지고 있다. 그러나 useEffect 밖에서 함수를 선언하다 보니 불필요한 코드가 많아지고 가독성이 떨어졌다.

```tsx
function Component({ id }: { id: string }) {
  const [info, setInfo] = useState<number | null>(null);

  useEffect(() => {
    const controller = new AbortController()(async () => {
      const result = await fetchInfo(id, { signal: controller.signal });
      setInfo(await result.json());
    })();

    return () => controller.abort();
  }, [id]);
  return <div>{/* 렌더링 */}</div>;
}
```

useEffect 외부에 있던 관련 함수를 내부로 가져왔더니 훨씬 간결한 모습이다. 불필요한 의존성 배열도 줄일 수 있었고, 무한 루프에 빠지기 위해 넣었던 코드인 useCallback도 삭제할 수 있었다.
