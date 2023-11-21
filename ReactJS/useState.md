# useState

state : 컴포넌트가 가질 수 있는 상태이다.
useState : 컴포넌트 상태를 간편하게 생성, 업데이트하게 해준다.

### 왜 state를 직접 바꾸지 않고 useState를 사용해야 하나요?

state는 컴포넌트 렌더링에 영향을 주는 데이터를 갖고 있는 객체인데,
이것을 업데이트 하기 위해서는 setState, useState가 필요하다.
직접 state를 수정하면 리액트는 render함수를 호출하지 않아서 렌더링이 일어나지 않고 setState를 호출하여 state를 변경하면 리액트 엔진이 render함수를 이용해서 렌더링을 실행하기 때문이다.

### setState()를 호출할 때마다 컴포넌트가 리렌더링되나요?

값이 변경되지 않으면 리렌더링이 일어나지 않는다. 값을 바꿔야 리렌더링이 일어난다.

### 아래와 같은 코드일 때 count가 한 번에 5씩 늘어날 것 같은데, 의도와 다르게 작동하네요. 왜 그럴까요?

```JavaScript
function onClickCountUp() {
   setCount(count + 1); // state는 리액트 html에서 변경을 감지함(즉, document.get... 필요없음)
   setCount(count + 1);
   setCount(count + 1);
   setCount(count + 1);
   setCount(count + 1);
}
// 이 함수를 클릭이벤트 핸들러로 달면 5씩 올라가느게 아니라 1씩 올라감
```

setState는 불필요한 렌더링을 방지하면서 성능을 향상시키기 위해 즉시 함수를 수행하지 않도록 설계되었다. 이러한 작동 방식은 비동기적으로 작동한다고 할 수 있다. 의도대로 5씩 올라가게 하려면, prev라는 임시 저장 공간을 사용해 작성해야 한다.

```JavaScript
function onClickCountUp() {
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
}
```

이렇게 prev를 사용하면 임시 저장 공간에 있는 값을 먼저 꺼내오고, 만약 임시 저장 공간에 있는 값이 없다면 기본 값을 불러오게 된다.

그럼 왜 이렇게 비동기적으로 작동할까?

한 마디로 일정 시간동안 변화하는 상태를 모아 한 번에 렌더링 하기 위해서이다. 결국, 웹 페이지를 불필요한 렌더링 횟수를 줄어 좀 더 빠른 속도로 동작하게 만들기 위해서이다.
