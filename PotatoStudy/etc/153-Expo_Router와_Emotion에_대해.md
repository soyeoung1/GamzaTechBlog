## Expo Router란?
Expo Router는 React Native 및 웹 애플리케이션을 위한 파일 기반 라우터이며 앱 화면 간 탐색을 관리하여 사용자가 여러 플랫폼에서 동일한 구성 요소를 사용하여 앱 UI의 여러 부분 사이를 원활하게 이동할 수 있도록 지원한다. 

### 특징

**1. 파일 기반 라우팅**

React Navigation을 사용할 때는 이런 식으로 코드에서 화면을 등록한다

```tsx
const Stack = createNativeStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```
이 방식은 화면이 많아질수록 Screen 등록 코드도 길어지고, 전체 구조를 파악하려면 코드를 쭉 스크롤해서 봐야 한다. 반면 Expo Router는 이 부분을 파일/폴더 구조로 대체한다.

Expo Router에서는 프로젝트 안에 app/ 폴더를 만들고, 그 안에 파일을 생성하면 그게 곧 하나의 화면이 된다. **폴더 구조만 봐도, 이 앱이 무슨 화면이 있는지 직관적으로 알 수 있음**
```bash
app/
  index.tsx        → "/"
  profile.tsx      → "/profile"
  settings/
    index.tsx      → "/settings"
    detail.tsx     → "/settings/detail"
```


**2. 동적 라우팅**

웹에서 /posts/1, /posts/2처럼 URL 일부가 계속 바뀌는 주소를 사용할 때가 있는데 Expo Router에서는 이런 주소를 파일 이름으로 표현할 수 있다. (파일이나 폴더 이름에 [] 대괄호를 사용해서 동적 경로 세그먼트를 만듦)

```bash
app/
  posts/
    [id].tsx      → "/posts/:id"
   ```
  -> 이 구조에서 /posts/1 혹은 /posts/abc로 이동하면, 둘 다 app/posts/[id].tsx 화면이 렌더링되고, id 값만 다르게 들어옴
  
  
  
  
  **3. 레이아웃**
  
  Expo Router에서 또 하나 중요한 특징은 _layout.tsx다. 이 파일은 해당 폴더의 공통 레이아웃 + 네비게이션 컨테이너 역할을 한다. 여러 화면에 사용되는 공통적인 UI나 내비게이션 설정을 관리할 수 있다.
  
  화면 상단의 헤더, 하단 네비게이션 바 등과 같이 반복적으로 사용되는 부분들을 모든 화면의 파일들 마다 넣으면 비효율적이고 유지보수에도 적절하지 않음 -> **레이아웃이라는 하나의 '템플릿'을 사용해서 관리**
  
  ```bash
  app/
  _layout.tsx // 폴더의 모든 화면을 Stack으로 관리
  index.tsx
  profile.tsx
```

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: "홈" }} />
      <Stack.Screen name="profile" options={{ title: "프로필" }} />
    </Stack>
  );
}
``` 
_layout.tsx 파일이 하나의 틀 역할을 하고 다른 화면들을 쌓는 것

```bash
app/
  (tabs)/
    _layout.tsx
    home.tsx
    profile.tsx
  settings.tsx
```

```tsx
import { Tabs } from "expo-router";

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen name="home" options={{ title: "홈" }} />
      <Tabs.Screen name="profile" options={{ title: "프로필" }} />
    </Tabs>
  );
}
```
- /home → 탭 네비게이션 안의 홈 탭
- /profile → 탭 네비게이션 안의 프로필 탭
- /settings → 탭 바 바깥의 독립적인 화면

괄호로 폴더 이름을 묶으면 실제 URL 경로에는 포함되지 않고 구조를 나눌 수 있으며 탭 네비게이션을 만들 때 자주 쓰인다.

### Expo Router를 쓰는 이유?

**1. `app/` 폴더 구조가 곧 라우팅 구조라서 직관적**
- 별도의 라우트 등록 코드 없이도 구조 그대로 화면이 만들어져 코드 안에서 Stack.Screen을 계속 추가할 필요가 없고, 파일 트리만 봐도 직관적으로 파악할 수 있다는 점이 편함

**2. 새 화면 추가가 빠르고 생산성이 높음**
- React Navigation만 사용할 때는, 새 화면을 만들 때 여러 과정을 거쳐야 하는데.
1.app/ 폴더 안에 새 파일 생성
2.그 파일에서 화면 컴포넌트를 export
이 두 단계만으로 라우트가 자동으로 등록되어 파일만 생성해도 바로 라우트가 생긴다는 점 덕분에 개발 속도가 빨라짐

**3. _layout.tsx로 레이아웃과 네비게이션을 폴더 단위로 관리**

네비게이션 구조를 폴더 단위로 나누어 설계해 폴더 구조와 네비 구조가 거의 1:1로 대응되기 때문에, 나중에 코드를 다시 볼 때도 이해하기 쉽고 레이아웃을 분리해서 관리할 수 있다는 장점이 있음

## ✔️ 
React Navigation만 쓸 때는, 새로운 화면을 만들 때마다 네비게이션 설정 파일을 열고 `Stack.Screen`을 하나씩 추가해야 하고 화면 수가 많아질수록 라우팅 설정 코드도 같이 길어지고, 전체 구조를 파악하려면 코드를 스크롤해서 전부 확인해야 했다.

Expo Router는 `app/` 폴더 안의 **파일·폴더 구조 자체가 라우팅이 되기 때문에 프로젝트 트리만 봐도 어떤 화면들이 있는지, URL 구조가 어떻게 생겼는지 한눈에 파악**할 수 있다는 점이 장점이고 특히 동적 라우팅이나 탭 레이아웃처럼 복잡해질 수 있는 부분들도 폴더 구조와 파일 이름만 잘 잡아주면 자연스럽게 정리되는 느낌이라 좋은 거 같다. 

실제 프로젝트에서 더 많은 화면과 복잡한 플로우를 구성해볼수록, 이 파일 기반 라우팅의 장점이 더 잘 느껴질 것 같고, 반대로 초반 폴더 구조 설계를 어떻게 하느냐가 중요하겠다는 걸 깨닫게 됨


***


## Emotion이란?
JS/TS 파일 안에서 스타일을 직접 작성할 수 있게 해주는 CSS-in-JS 라이브러리다. 프레임워크에 구애 받지 않고 사용이 가능하며 React와 함께 사용할 수도 있다.

CSS를 쓸 때는 보통 
1. App.css에 클래스를 정의
2. 컴포넌트에서 className="button" 사용

이 방식은 컴포넌트와 스타일이 물리적으로 멀리 떨어져 있어서, 관련 코드를 한 번에 보기 어렵운 단점이 있는데 Emotion은 컴포넌트 파일 안에서 바로 스타일을 정의하는 방식을 지원한다.

### Emotion 패키지 3가지 비교
|이름         | 사용 환경            | 주요 특징                                                  |
|---------------------|----------------------|-----------------------------------------------------------|
| `@emotion/react`    | React 전용           | `css` prop, `css` 함수, ThemeProvider, Global 등 코어 기능 |
| `@emotion/styled`   | React 전용           | `styled.div` 형태로 styled 컴포넌트 생성                  |
| `@emotion/css`      | React 무관(순수 JS)  | `css()`가 className 문자열을 반환, 어디서나 사용 가능     |


### 1.@emotion/react
React에서 Emotion을 쓸 때 기본이 되는 패키지

```tsx
import { css } from "@emotion/react";

const boxStyle = css`
  padding: 16px;
  background: #f5f5f5;
  border-radius: 8px;
`;

function Box() {
  return <div css={boxStyle}>내용</div>;
}

```
- ThemeProvider로 테마(컬러, 폰트 등) 제공
- Global 컴포넌트로 전역 스타일(body, html 등) 설정

### 2.@emotion/styled
styled-components랑 거의 같은 문법으로 styled.div, styled.button 식으로 스타일 입힌 컴포넌트를 만드는 방식
```tsx
import styled from "@emotion/styled";

const Button = styled.button`
  padding: 8px 16px;
  border-radius: 8px;
  font-weight: 600;
  background: #ff6435;
  color: white;
`;

function App() {
  return <Button>클릭</Button>;
}
```
- 가장 유명한 패턴인 “styled 컴포넌트” 방식 제공
- 컴포넌트를 단위로 재사용 가능한 UI 만들기 좋음


### 3.@emotion/css
React에 종속되지 않고, 순수 JS 환경에서도 쓸 수 있으며 클래스를 문자열로 리턴해서 붙이는 방식
```tsx
import { css } from "@emotion/css";

const buttonClass = css`
  padding: 8px 16px;
  border-radius: 8px;
  background: #ff6435;
  color: white;
`;

function App() {
  return <button className={buttonClass}>버튼</button>;
}
```
->css 함수가 className 문자열을 반환함
->그 문자열을 <button className={ }>에 넣어서 사용
  
  
 ## ✔️
  React 프로젝트에서는 보통 `@emotion/react`와 `@emotion/styled`를 함께 사용하는 경우가 많고, 컴포넌트와 스타일을 같은 파일 안에 두어서 코드 가독성이 높으며 자주 쓰는 UI 컴포넌트를 스타일과 함께 묶어서 재사용하기 쉽다는 점이 좋은 거 같다. 