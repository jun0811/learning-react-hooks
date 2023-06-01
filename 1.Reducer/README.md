## Reducer and Effect

### State Hook의 문제점들

State Hook은 이미 복잡한 객체와 배열을 전달할 수 있도록 지원하며, 상태 변경도 완벽하게 처리할 수 있습니다. 하지만 **`항상 상태를 직접 변경해야 하므로`** 상태의 다른 부분을 덮어쓰지 않도록 하기 위해 많은 `스프레드 구문`을 사용해야 합니다.

```jsx
const [config, setConfig] = useState({ filter: "all", expandPosts: true });

//원하는 필터
setConfig({ filter: { byAuthor: "Daniel Bugl", fromDate: "2019-04-29" } });
// 구현 코드
setConfig({
  ...config,
  filter: { byAuthor: "Daniel Bugl", fromDate: "2019-04-29" },
});
```

지금은 별 다른게 없다고 느껴질 수 있죠.

하지만 이번엔 필터에서 byAuthor별은 같고 날짜만 바뀐다고 해보져

```jsx
setConfig({ ...config, filter: { ...config.filter, fromDate: "2019-04-30" } });
```

`spread syntax`가 두번이나 나오게 되죠.

만약, 상태가 문자열인 상태에서 위와 같은 작업을 수행하면 예상치 못한 결과가 발생할 수 있습니다. 문자열은 스프레드 구문을 사용하면 각 문자가 해당 문자의 인덱스를 키로 가진 객체로 분리됩니다. 따라서 다음과 같은 결과가 발생합니다:

`{ filter: { '0': 'a', '1': 'l', '2': 'l', fromDate: '2019-04-30' }, expandPosts: true }`

위의 결과에서 볼 수 있듯이, 문자열을 스프레드 구문으로 복사하면 각 문자가 키로 변환되어 상태 객체에 추가됩니다.

이렇게 상태 객체를 직접 변경하고 스프레드 구문을 사용하는 방식은 큰 상태 객체의 경우 매우 번거로울 수 있습니다. 또한, 버그를 방지하고 여러 곳에서 버그를 확인해야 하는 번거로움이 있습니다.

## Reducers

이제 상태 변경을 처리하는 함수를 정의해야합니다. **현재 상태와 동작을 인수로 취하고 새 상태를 반환합니다.**

```jsx
{ expandPosts: true, filter: 'all' } // 초기 상태

{ expandPosts: true, filter: { fromDate: '2019-04-29' } }

{ expandPosts: true, filter: { fromDate: '2019-04-29',
byAuthor: 'Daniel Bugl' } }

{ expandPosts: true, filter: { fromDate: '2019-04-30',
byAuthor: 'Daniel Bugl' } }


// 이 코드를 전달하면 상태는 초기 상태 all string으로 돌아간다.
{ type: 'CHANGE_FILTER', all: true }
```

위와 같이 상태가 변하는 로직이 필요합니다. 이를 위한 reducer를 만들어 보겠습니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "TOGGLE_EXPAND":
      return { ...state, expandPosts: !state.expandPosts };
    case "CHANGE_FILTER":
      if (action.all) {
        return { ...state, filter: "all" };
      }
      let filter = typeof state.filter === "object" ? state.filter : {};
      if (action.fromDate) {
        filter = { ...filter, fromDate: action.fromDate };
      }
      if (action.byAuthor) {
        filter = { ...filter, byAuthor: action.byAuthor };
      }
      return { ...state, filter };
    default:
      throw new Error();
  }
}
```

## **The Reducer Hook**

**액션과 리듀서 기능을 정의했으므로 리듀서에서 Reducer Hook를 생성할 수 있습니다. useReducer Hook은 다음과 같습니다.**

`const [ state, dispatch ] = useReducer(reducer, initialState)`
이제 초기 상태를 정의 해줘야합니다.

`const initialState = { all: true }`

이제 Reducer에서 반환된 상태 객체를 사용하여 상태에 액세스할 수 있습니다.
다음과 같이 디스패치 기능을 통해 작업을 후크 및 디스패치합니다.

`dispatch({ type: 'TOGGLE_EXPAND' })`

`dispatch({ type: 'CHANGE_FILTER', fromDate: '2019-04-30' })`

**액션과 리듀서를 사용하여 상태 변경을 처리하는 것은 상태 객체를 직접 조정하는 것보다 훨씬 쉽습니다.**

Global State의 변경은 어디서든 일어날 수 있기 때문에 일반적으로 State Hook보다는 Reducer Hook을 사용하는 것이 좋다. 작업을 처리하고 상태 변경 로직을 한곳에서만 업데이트 하는 것이 더 쉽기 때문입니다.

( 모든 상태 변경 로직이 한곳에 있다면 버그를 수정하기도 쉽다. )

### State Hook을 Reducer로 바꾸기

블로그앱에서 useReducer를 이용하여 state 관리를 할 수 있도록 바꾸어 보겠습니다.

먼저 reducer 함수를 작성해야합니다.

1. App() 함수 작성전에 userReducer를 작성해줍시다.

```jsx
function userReducer(state, action) {
  switch (action.type) {
    case "LOGIN":
    case "REGISTER":
      return action.username;
    case "LOGOUT":
      return "";
    default:
      throw new Error();
  }
}
```

1. useReducer를 가져오고

`import React, { useState, useReducer } from "react";`

1. 기존 useState로 관리했던 user문장을 지우고 useReducer를 사용하여 대체해줍니다.

`const [user, dispatchUser] = useReducer(userReducer, "");`

1. 이제 setState함수를 `dispatchUser` 로 바꿔야합니다.

```jsx
export default function App() {
  const [user, dispatchUser] = useReducer(userReducer, "");
  const [posts, setPosts] = useState(defaultPosts);
  return (
    <div style={{ padding: 20 }}>
      <UserBar user={user} dispatch={dispatchUser} /> //수정
      <br />
      {user && <CreatePost user={user} posts={posts} setPosts={setPosts} />}
      <br />
      <hr />
      <PostList posts={posts} />
    </div>
  );
}
//// UserBar.js
export default function UserBar({ user, dispatch }) {
  if (user) {
    return <Logout user={user} dispatch={dispatch} />;
  } else {
    return (
      <>
        <Login dispatch={dispatch} />
        <Register dispatch={dispatch} />
      </>
    );
  }
}
// Login.js
export default function Login({ dispatch }) {
  const [username, setUsername] = useState("");
  const handleUsername = (e) => {
    e.preventDefault();
    dispatch({ type: "LOGIN", username });
  };
  //......
// Logout.js
export default function Logout({ user, dispatch }) {
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        dispatch({ type: "LOGOUT" });
      }}
    >
      Logged in as: <b>{user}</b>
      <input type="submit" value="Logout" />
    </form>
  );
}
// Register
export default function Register({ dispatch }) {
  const [password, setPassword] = useState("");
  const [repeatedPassword, setRepeatedPassword] = useState("");
  const [username, setUsername] = useState("");

  function handlePassword(evt) {
    setPassword(evt.target.value);
  }
  function handlePasswordRepeat(evt) {
    setRepeatedPassword(evt.target.value);
  }

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        dispatch({ type: "REGISTER", username });
      }}
    >
```

식으로 바꾸어 주면 된다. post는 개별로 해보고 적용해보기 바란다.

## useEffect

Effect Hook, 즉 useEffect는 함수 컴포넌트 내에서 이런 side effects를 수행할 수 있게 해줍니다. React class의 `componentDidMount` 나 `componentDidUpdate`, `componentWillUnmount`와 같은 목적으로 제공되지만, 하나의 API로 통합되어있습니다.

```jsx
import React, { useState, useEffect } from "react";

function Example() {
  const [count, setCount] = useState(0);

  // componentDidMount, componentDidUpdate와 비슷합니다
  useEffect(() => {
    // 브라우저 API를 이용해 문서의 타이틀을 업데이트합니다
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

- React는 우리가 넘긴 함수를 기억했다가(이 함수를 ‘effect’라고 부릅니다) DOM 업데이트를 수행한 이후에 불러냄
- 기본적으로 첫번째 렌더링과 이후의 모든 업데이트에서 수행

### 정리(clean-up)

```jsx
import React, { useState, useEffect } from "react";

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // effect 이후에 어떻게 정리(clean-up)할 것인지 표시합니다.
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return "Loading...";
  }
  return isOnline ? "Online" : "Offline";
}
```

**effect에서 함수를 반환하는 이유 : effect를 위한 추가적인 정리(clean-up) 메커니즘**

**반환 시점 :** 컴포넌트가 마운트 해제 되는 때 `클린 업` 을 실행

### 멀티 Effect

```jsx
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

- Hook을 이용하면 생명주기 메서드에 따라서가 아니라 코드가 무엇을 하는지에 따라 나눌 수가 있습니다. React는 컴포넌트에 사용된 모든 effect를 지정된 순서에 맞춰 적용

### Effect를 건너뛰어 성능 최적화하기

```jsx
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // count가 바뀔 때만 effect를 재실행합니다.
```

- useEffect로 전달된 함수는 `지연 이벤트 동안에 레이아웃 배치와 그리기를 완료한 후` 발생

### 마운트/ 언마운트

```jsx
useEffect(() => {
  console.log("컴포넌트가 화면에 나타남");
  return () => {
    console.log("컴포넌트가 화면에서 사라짐");
  };
}, []);
```

- 첫번째 파라미터에는 함수, 두번째 파라미터에는 의존값이 들어있는 배열 (deps)
- deps 배열을 비우게 된다면, 컴포넌트가 처음 나타날때에만 useEffect 에 등록한 함수가 호출되고 컴퍼넌트가 사라질때 return 함수가 호출
- `useEffect` 안에서 사용하는 상태나, props 가 있다면, `useEffect` 의 `deps` 에 넣어주어야 합니다. 만약 `useEffect` 안에서 사용하는 상태나 props 를 `deps` 에 넣지 않게 된다면 `useEffect` 에 등록한 함수가 실행 될 때 최신 props / 상태를 가르키지 않게 됩니다.
