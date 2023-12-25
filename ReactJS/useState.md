# useState

state : 컴포넌트가 가질 수 있는 상태이다. <br/>
useState : 컴포넌트 상태를 간편하게 생성, 업데이트하게 해준다.

- useState의 초기값에 아무값을 넘겨주지 않으면 undefined

### 왜 state를 직접 바꾸지 않고 useState를 사용해야 하나요?

> **💡 면접을 위한 quick answer :** state는 컴포넌트 렌더링에 영향을 주는 데이터를 갖고 있는 객체인데,
> 이것을 업데이트 하기 위해서는 setState, useState가 필요하다.
> 직접 state를 수정하면 리액트는 render함수를 호출하지 않아서 렌더링이 일어나지 않고 setState를 호출하여 state를 변경하면 리액트 엔진이 render함수를 이용해서 렌더링을 실행하기 때문이다.

```jsx
function Component() {
  let state = 'hello';

  function handleBtnClick() {
    state = 'hayeong';
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleBtnClick}>눌러봐~</button>
    </>
  );
}
```

리액트에서 렌더링은 함수형 컴포넌트의 return 과 클래스형 컴포넌트의 render 함수를 실행한 다음, 이 실행 결과를 이전의 리액트 트리와 비교해 리렌더링이 필요한 부분만 업데이트해 이뤄진다.
위 코드에서는 리렌더링을 발생시키기 위한 조건을 충족하지 못하고 있다.

### 게으른 초기화(lazy initialization)

useState의 인수로 특정한 값을 넘기는 함수를 인수로 넣어줄 수도 있다. 이것을 게으른 초기화라고 한다.
게으른 초기화 함수는 오로지 state가 처음 만들어질 때만 사용된다. 만약 이후 리렌더링이 발생된다면 이 함수의 실행은 무시된다.

만약 `Number.parseInt(window.localStorage.getItem(cacheKey))`와 같이 한 번 실행되는 데 어느정도 비용이 드는 값이 있다고 가정해보자. useState의 인수로 이 값 자첼 사용한다면 매번 렌더링 시 해당 값에 접근해서 낭비가 발생한다. 따라서 이런 경우 함수 형태로 인수에 넘겨주는 것이 경제적이다. **초기값이 없다면 함수를 실행해 무거운 연산을 시도할 것이고, 이미 초기값이 존재한다면 함수 실행을 하지 않고 기존 값을 사용할 것이다.**

- 게으른 초기화 쓰는 경우 : 무거운 연산이 요구될 때 사용
  - localStorage나 sessionStorage에 대한 접근
  - map, filter, find같은 배열에 대한 접근
  - 초깃값 계산을 위해 함수 호출이 필요할 때와 같이 무거운 연산을 포함해 실행 비용이 많이 드는 경우

### setState()를 호출할 때마다 컴포넌트가 리렌더링되나요?

> **💡 면접을 위한 quick answer :** 값이 변경되지 않으면 리렌더링이 일어나지 않는다. 값을 바꿔야 리렌더링이 일어난다.

### 아래와 같은 코드일 때 count가 한 번에 5씩 늘어날 것 같은데, 의도와 다르게 작동하네요. 왜 그럴까요?

> **💡 면접을 위한 quick answer :** setState는 불필요한 렌더링을 방지하면서 성능을 향상시키기 위해 즉시 함수를 수행하지 않도록 설계되었다. 이러한 작동 방식은 비동기적으로 작동한다고 할 수 있다. 의도대로 5씩 올라가게 하려면, prev라는 임시 저장 공간을 사용해 작성해야 한다. 한 마디로 일정 시간동안 변화하는 상태를 모아 한 번에 렌더링 하기 위해서이다. 결국, 웹 페이지를 불필요한 렌더링 횟수를 줄어 좀 더 빠른 속도로 동작하게 만들기 위해서이다.

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

### useState 내부의 모습을 구현한 모습 (from. 모던 리액트 deep dive)

```jsx
const MyReact = function () {
  const global = {};
  let index = 0;

  function useState(initialState) {
    if (!global.states) {
      // 애플리케이션 전체의 states 배열을 초기화한다.
      // 최초 접근이라면 빈 배열로 초기화한다.
      global.states = [];
    }

    // states 정보를 조회해서 현재 상태값이 있는지 확인하고,
    // 없다면 초깃값으로 설정한다.
    const currentState = global.states[index] || initialState;
    // states의 값을 위에서 조회한 현재 값으로 업데이트한다.
    global.states[index] = currentState;

    // 즉시 실행 함수로 setter를 만든다.
    const setState = (function () {
      // 현재 index를 클로저로 가둬놔서 이후에도 계속해서 동일한 index에
      // 접근할 수 있도록 한다.
      let currentIndex = index;
      return function (value) {
        global.states[currentIndex] = value;
        // 컴포넌트를 렌더링한다. 실제로 컴포넌트를 렌더링하는 코드는 생략했다.
      };
    })();
    // useState를 쓸 때마다 index를 하나씩 추가한다. 이 index는 setState에서 사용된다.
    // 즉, 하나의 state마다 index가 할당돼 있어 그 index가 배열의 값(global.states)을
    // 가리키고 필요할 때마다 그 값을 가져오게 한다.
    index = index + 1;

    return [currentState, setState];
  }

  // 실제 useState를 사용하는 컴포넌트
  function 컴포넌트() {
    const [value, setValue] = useState(0);

    // ..
  }
};
```

useState는 클로저에 의존해 구현돼 있을 것이다. 클로저를 사용함으로써 외부에 해당 값을 노출시키지 않고 오직 리액트에서만 쓸 수 있었고, 함수형 컴포넌트가 매번 실행되더라도 useState에서 이전의 값을 정확하게 꺼내 쓸 수 있게 됐다.
