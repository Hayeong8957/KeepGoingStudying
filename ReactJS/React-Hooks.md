# React Hooks?

Class 컴포넌트 사용을 위해서는 JS의 this가 어떻게 작동하는지 알아야만 합니다.
JS의 this는 다른 언어와 다르게 작동함으로 사용자에게 큰 혼란을 주었고, 코드의 재사용성과 구성을 매우 어렵게 만들었습니다.
또한 Class는 코드의 최소화를 힘들게 만들었습니다. 길어지는 코드 길이 문제, 중복 코드, 가독성 문제가 많았습니다.

이러한 문제를 해결하기 위해 Hook은 Class 없이 React기능들을 사용하는 방법을 제시했습니다.
use키워드를 앞에 붙여서 클래스 컴포넌트가 가지고 있는 기능을 함수형 컴포넌트에서 낚아채서(hooking) 사용할 수 있게 만든 것이 Hooks입니다.

즉, React Hooks는 DX(개발자 경험)를 위해 개발 된 것이라고 생각한다.

# 훅의 규칙

1. 최상위에서만 훅을 호출해야 한다. 반복문이나 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다. 이 규칙을 따라야만 컴포넌트가 렌더링 될 때마다 항상 동일한 순서로 훅이 호출되는 것을 보장할 수 있다.
2. 훅을 호출할 수 있는 것은 리액트 함수형 컴포넌트, 혹은 사용자 정의 훅의 두 가지 경우뿐이다. 일반 자바스크립트 함수에서는 훅을 사용할 수 없다.

> **잠깐! React Fiber란?**
>
> - 리액트 핵심 알고리즘을 재구성한 새 재조정 엔진이다.
> - React Fiber의 목표는 애니메이션, 레이아웃, 제스처, 중단 또는 재사용 기능과 같은 영역에 대한 적합성을 높이고 다양한 유형의 업데이트에 우선 순위를 지정하는 것이다.
> - 핵심 기능은 렌더링을 증분하는 것이다. 이는 렌더링 작업을 여러 덩어리로 나누어 여러 프레임에 분산하는 기능이다.
>
> 문서에 의하면, 이 기능의 주요 목표는 다음과 같다.
>
> 1. 중단 가능한 작업을 덩어리로 나누기
> 2. 진행 중인 작업의 우선순위를 지정하고, 리베이스하고 재사용
> 3. 리액트의 레이아웃을 지원하기 위해 부모와 자식 간에 yield back and forth
> 4. render()로부터 다수 엘리먼트들을 반환
> 5. 에러 바운더리에 대한 더 나은 지원

useState나 useEffect는 순서에 큰 영향을 받는다. 객체 기반 링크드 리스트로 구현돼있기 때문이다.

```jsx
function Component() {
  const [count, setCount] = useState(0);
  const [required, setRequired] = useState(false);

  useEffect(() => {
    // do something...
  }, [count, required]);
}
```

이 컴포넌트는 파이버에서 다음과 같이 저장된다.

```js
{
  memoizedState: 0, // setCount 훅
  baseState: 0,
  queue: {/* ... */},
  baseUpdate: null,
  next: { // setRequired 훅
    memoizedState: false,
    baseState: false,
    queue: {/* ... */},
    baseUpdate: null,
    next: { // useEffect 훅
      memoizedState: {
        tag: 192,
        create: () => {},
        destroy: undefined,
        deps: [0, false],
        next: {/* ... */}
      },
      baseState: null,
      queue: null,
      baseUpdate: null,
    }
  }
}
```

리액트 훅은 파이버 객체의 링크드 리스트의 호출 순서에 따라 저장된다. 그 이유는 각 훅이 파이버 객체 내에서 순서에 의존해 state나 effect의 결과에 대한 값을 저장하고 있기 때문이다. 이렇게 고정된 순서에 의존해 훅과 관련된 정보를 저장함으로써 이전 값에 대한 비교와 실행이 가능해진다.

- 리액트 공식문서에 있는 훅에 대한 잘못된 예제

```jsx
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

만약 setName을 빈 값으로 업데이트 하면 어떻게 될까? name이 업데이트 되면서 2번째로 있던 훅 UseEffect를 호출해서 변경하려고 했지만 조건부 구문이 추가되면서 2번째 훅이 useState가 되어버렸다. 즉, 링크드 리스트가 깨져버린 것이다. 이렇게 조건이나 다른 이슈로 인해 훅의 순서가 깨지거나 보장되지 않을 경우 리액트 코드는 에러를 발생시킨다. 이 상황을 코드로 나타내면 다음과 같다.

```jsx
// ------------
// First render
// ------------
useState('Mary'); // 1. Mary 할당
useEffect(persistForm); // 2. 1에 있던 state를 기반으로 effect 실행
useState('Poppins'); // 3. 'Poppins' 할당
useEffect(updateTitle); // 4. 3에 있는 35를 기반으로 effect 실행

// -------------
// Second render
// -------------
useState('Mary'); // 1. state를 읽음(useState의 인수는 첫 번째 렌더링에서 초깃값으로 사용됐으므로 여기에서 인수값은 무시되고, 이전에 저장해 두었던 Mary 값이 사용된다.)
// useEffect(persistForm)  // 🔴 조건문으로 인해 실행이 안 됨
useState('Poppins'); // 🔴 2. 원래는 3이었음. 이제 2가 되면서 useState의 값을 읽어오지 못하고 비교도 할 수 없음
useEffect(updateTitle); // 🔴 3. 원래는 4였음. updateTitle을 하기 위한 함수를 대체하는 데 실패

// ...
```

그러므로 훅은 절대 조건문, 반복문 등에 의해 리액트에서 예측 불가능한 순서로 실행되게 해서는 안 된다. 항상 훅은 실행 순서를 보장받을 수 있는 컴포넌트 최상단에 선언돼 있어야 한다. 조건문이 필요하다면 반드시 훅 내부에서 수행해야 한다.
