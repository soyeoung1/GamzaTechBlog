![Uploaded Image](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/post-images/158/1374f8c3-231c-41e4-a05d-915dd7dda215_image.png)

# React Query란?
React Query는 리액트 애플리케이션에서 **서버 상태**를 효율적으로 관리하기 위한 라이브러리이다.

쉽게 말해 웹 애플리케이션에서 서버로부터 데이터를 가져오고,
캐싱하고, 동기화하고, 업데이트하는 작업을 매우 간편하게 만들어준다.

## 서버 상태와 클라이언트 상태

리액트에서 상태는 크게 두 가지로 나눌 수 있다.                                      

| 클라이언트 상태 | 서버상태 |
| --- | --- |
| 모달 열림 여부, 버튼 클릭 상태 등 UI와 관련된 상태 | API를 통해 가져온 데이터  |
| 사용자의 행동에 따라 바로 변경 | 언제 변경될지 예측 어려움  |
| 내부에서만 관리 | 여러 컴포넌트에서 공유 |

클라이언트 상태는 UI 중심, 서버 상태는 데이터 중심이라고 구분하면 이해하기 쉽다.
React Query는 이런 서버 상태 관리를 매우 잘 도와주는 라이브러리이다.

## 왜 React Query를 쓸까?


useEffect와 useState로도 서버 데이터를 관리할 수는 있지만,
로딩, 에러, 캐싱, 최신성까지 직접 처리해야 한다는 한계가 있다.

간단한 예시로 유저 목록을 불러오는 컴포넌트를 만들어보면


useEffect + useState로 구현한 코드이다.
```tsx

import { useEffect, useState } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchUsers = async () => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (err) {
      setError(err);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  if (isLoading) return <p>로딩 중...</p>;
  if (error) return <p>에러가 발생했습니다.</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

```

여기서 알 수 있는 점은

#### 1. 로딩 상태를 직접 관리해야 한다
```tsx
const [isLoading, setIsLoading] = useState(false);

```
로딩 상태의 타이밍을 개발자가 직접 신경 써야 한다.

#### 2. 에러 처리를 매번 구현해야 한다
```tsx
try {
  ...
} catch (err) {
  setError(err);
}
```
비슷한 에러 처리 코드가 여러 컴포넌트에 반복되기 쉽다.

#### 3. 같은 API를 여러 번 호출하게 된다
```tsx
<UserList />
<UserProfile />
```
두 컴포넌트가 각각 같은 API를 호출하게 된다.

#### 4. 새로고침, 재요청 로직이 복잡하다
```tsx
<button onClick={fetchUsers}>다시 불러오기</button>
```

재요청을 위해 함수 분리 필요하며
서버 데이터를 다루는 로직이 점점 컴포넌트 안에 쌓이게 된다.

#### 여기서 React Query를 사용하면?
같은 기능을 React Query로 구현하면 코드가 훨씬 단순해진다.

```tsx
import { useQuery } from '@tanstack/react-query';

function UserList() {
  const { data: users, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const response = await fetch('/api/users');
      return response.json();
    },
  });

  if (isLoading) return <p>로딩 중...</p>;
  if (error) return <p>에러가 발생했습니다.</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```
기존 방식은 서버 상태를 클라이언트 상태처럼 관리하려다 보니
코드가 점점 복잡해지는 느낌이 들었지만

React Query는 이러한 반복적인 문제를 해결해
서버 상태에만 집중할 수 있게 해주는 도구라고 생각할 수 있다.


## React Query의 기본 문법


#### 1. useQuery 데이터를 조회
```tsx
useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
});
```
**queryKey**는 **데이터를 구분하는 고유한 키**이고
**queryFn**는 **실제 API 요청 함수**이다

queryKey가 같으면 같은 데이터로 인식하고 캐시를 공유한다.
 
#### 2. useQuery 반환
```tsx
const {
  data,
  isLoading,
  isError,
  error,
  refetch,
} = useQuery(...);
```
data : 서버에서 받아온 데이터
isLoading : 처음 데이터를 불러오는 중인지
isError / error : 에러 상태
refetch : 수동으로 다시 요청

#### 3. Query Key

 

```tsx
// 사용자 목록을 조회하는 쿼리
useQuery({ queryKey: ['users'], ... })

// 특정 사용자 상세 정보를 조회하는 쿼리
useQuery({ queryKey: ['user', userId], ... })
첫 번째 값: 리소스 이름
```
Query Key는 데이터를 캐싱하기 관리하는 고유의 키이며
같은 queryKey를 가진 useQuery는 동일한 데이터를 공유한다.

```tsx
useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
});
```
queryKey를 기준으로 캐싱, 리패칭, 데이터 공유가 이루어진다.



### 마치며

React Query가 단순히 데이터를 가져오는 라이브러리가 아니라
서버 상태를 효율적으로 관리하기 위한 도구라는 점을 알 수 있었고

기존의 useEffect와 useState 방식은
로딩과 에러 처리, 캐싱, 재요청 로직까지 모두 직접 관리해야 해서
컴포넌트가 점점 복잡해진다는 것을 알게 되었다.

React Query를 사용하면 이러한 반복적인 로직을 라이브러리가 대신 처리해주기 때문에
개발자는 데이터와 UI에 더 집중할 수 있다는 점이 개발하기 훨씬 편하게 만들어 주는 거 같다.