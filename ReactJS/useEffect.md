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
