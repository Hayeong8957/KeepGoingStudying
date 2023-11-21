# ReactJs Life Cycle

컴포넌트가 실행되거나 업데이트되거나 제거될 때, 특정한 이벤트들이 발생한다.
그리고 class 컴포넌트와 함수 컴포넌트에서의 라이프 사이클이 조금 다르게 적용된다.

<br/>
class 컴포넌트는 라이프 사이클 메서드를 사용하고, 함수형 컴포넌트는 Hook을 사용한다.

# class 컴포넌트 Life Cycle

class 컴포넌트는 라이프 사이클이 컴포넌트에 중심이 맞춰져 있다.
클래스가 마운드 되려할 때(`componentWillMount`), 마운트 되고 나서(`componentDidMount`), 업데이트 되었을 때(`componentDidUpdate`), 언마운트(`componentWillUnmount`) 될 때 실행된다.

![image](https://github.com/Hayeong8957/KeepGoingStudying/assets/70371342/f4d24e82-e763-433f-b991-35b8089cccd9)

### Mounting : 컴포넌트 생성될 때 발생

**1. state, context, defaultProps 저장**

컴포넌트가 시작되면 우선 context, defaultProps와 state를 저장한다.

> - context : React 컴포넌트 트리 내에서 데이터를 전역적으로 공유할 수 있는 방법을 제공한다. props를 전달하지 않고도 컴포넌트 간 데이터를 공유할 수 있다.
>
> - defaultProps : 컴포넌트에 props가 전달되지 않았을 때 기본값을 설정하는데 사용된다. 이를 통해 예상치 못한 props값으로 인해 오류가 발생하는 것을 방지하고, 더욱 안정적인 컴포넌트 개발을 할 수 있다.
>
> - state : 컴포넌트의 상태를 관리하는 객체이다.

**2. componentWillMount**

그 후에 componentWillMount메소드를 호출한다.

주의할 점은, componentWillMount에서는 props나 state를 바꾸면 안된다. Mount중이기 때문이다.
아직 DOM에 render하지 않았기에 DOM에도 접근할 수 없다.

**3. render**
render로 컴포넌트를 DOM에 부착한 후 Mount한다. 렌더링한다.

**4. componentDidMount**

그리고 render로 컴포넌트를 DOM에 부착한 후 Mount가 완료된 후 componentDidMount가 호출된다.
DOM에 접근할 수 있다. 주로 AJAX요청을 하거나, setTimeout, seInterval같은 행동을 한다.

### Updating : 컴포넌트가 업데이트 되는 시점에 일어남

props가 업데이트 될 때 과정이다.

**1. getDerivedStateFromProps**

컴포넌트의 props나 state가 바뀌었을때도 이 메서드가 호출된다.

**2. shouldComponentUpdate**

컴포넌트가 리렌더링 할지 말지를 결정하는 메서드이다.

아직 render하기 전이기에 return false를 하면 render를 취소할 수 있다.

주로 여기서 성능 최적화를 한다. 쓸데없는 update가 일어나면 여기서 걸러낸다.

**3. componentWillUpdate**

여기서는 state를 바꿔서는 안된다. 아직 props도 업데이트 하지 않았으므로 state를 바꾸면 또 shouldComponentUpdate가 발생한다.

**4. render**

render로 컴포넌트를 DOM에 부착한 후 Mount한다. 렌더링한다.

**5. componentDidUpdate**

컴포넌트가 업데이트 되고 난 후 발생한다. DOM에 접근할 수 있다.

### Unmounting : 컴포넌트가 화면에서 사라질 때 일어남

**componentWillUnmount**

더는 컴포넌트를 사용하지 않을 때 발생하는 이벤트이다.

주로 연결했던 이벤트 리스너를 제거하는 등의 여러 가지 정리 활동을 한다.

# 함수 컴포넌트 Life Cycle

함수 컴포넌트를 사용하면 useEffect를 빼놓을 수 없다.
함수 컴포넌트는 특정 데이터에 대해서 라이프 사이클이 진행된다.
<br/>
클래스 컴포넌트에서는 `componentWillMount`, `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`를 컴포넌트 당 한 번씩만 사용했다.
<br/>
useEffect는 데이터의 개수에 따라 여러 번 사용하게 된다.

```javascript
useEffect(() => {}); // 리렌더링마다 코드 실행
useEffect(() => {}, []); // mount시 첫 1회 코드 실행
useEffect(() => {}, [의존성1, 의존성2, ..]); // 조건부 effect 발생 의존성 배열 바뀔 때
useEffect(() => { return () => {}}); // unmount시 1회 실행
```

## 의존성 배열에 따른 라이프 사이클 예시

### componentDidMount + componentDidUpdate의 역할

```javascript
useEffect(() => {
  console.log('하영 등장');
}, [하영]);
```

위 코드는 컴포넌트 첫 렌더링 시 한 번 실행되고, 그 다음부턴 '하영'이 바뀔 때마다 실행된다.

### componentWillUnmount의 역할

보통 return으로 cleanup함수를 제공하면 된다.

```javascript
useEffect(() => {
  console.log('하영 등장');
  return () => {
    console.log('하영 제거');
  };
}, [하영]);
```

### componentDidUpdate의 역할?

useEffect는 기본적으로 componentDidMount + componentDidUpdate 동시에 수행한다.

즉, componentDidMount의 역할을 기대해선 안된다.

이를 위해서는 useRef라는 훅이 필요하다.
