---
title: "[A11Y] 컨텐츠 숨김 처리 사용하기"
excerpt: "컨텐츠 숨김 처리는 왜 하는것이고 리액트에선 어떻게 활용할까"

categories:
  - Etc
tags:
  - [a11y, react]

permalink: /etc/a11y-hidden/

toc: true
toc_sticky: true

date: 2024-02-05
last_modified_at: 2024-02-05
---

## 1️⃣ **숨김 처리를 하는 이유?**

일반 사용자들은 웹페이지를 시각적으로 탐색하는 것에 큰 어려움이 없지만, `스크린리더`와 같은 보조 도구를 사용해야 하는 사용자들은 웹페이지의 정보를 알기가 어렵다.

이들은 스크린리더에서 제공해주는 텍스트 정보에 의지할 수 밖에 없기 때문에, 우리는 접근성을 고려한 숨김 처리 기술을 활용하여 보이지 않는 정보를 제공해줘야 한다.

## 2️⃣ **CSS를 활용한 숨김 처리 기술**

숨김 처리를 하는 기술은 매우 다양하다. 실제 기업들에서는 "blind", "sr-only" 등 다양한 class명으로 활용 중이다.

### 🔸주의해야 할 CSS 속성

`display:none;` or `visibilty:hidden;`

스크린리더에서 무시되기 때문에, 모든 사용자에게 콘텐츠를 숨길 때만 사용해야 한다.

`width:0px; height:0px;`

정의된 높이나 너비가 없으면 대부분의 스크린리더가 읽지 않기 때문에 사용하지 않는게 좋다.

`text-indent:-9999px;`

콘텐츠를 왼쪽으로 밀어내는 방법으로, 스크린리더가 해당 텍스트를 읽는다. 하지만 포커스가 가능한 요소에 적용할 경우 포커스를 받을 수 있지만 페이지에는 표시되지 않는다.

### 🔸네이버 방식

```css
.blind {
  position: absolute;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  width: 1px;
  height: 1px;
  margin: -1px;
}
```

- `position: absolute;` : 페이지 레이아웃에서 제거
- `overflow: hidden;` : 요소의 크기를 벗어나는 콘텐츠 숨김
- `clip: rect(0,0,0,0);` : 요소가 화면에 보이는 영역을 0으로 지정
- `width: 1px; height: 1px;` : 스크린리더가 인식하도록 길이 최소로 지정
- `margin: -1px` : 요소의 크기 만큼 역마진을 주어, 요소가 차지하는 공간 제거

> mdn 문서에 따르면 clip 속성 대신, 최신 속성인 clip-path을 사용하는 것을 권장한다.
> 따라서 구 브라우저 대응을 위해 clip 속성을 유지하되 clip-path를 추가해주는 것이 좋다.

### 🔸CSS 클립 방법

```css
.a11y-hidden {
  clip: rect(1px, 1px, 1px, 1px);
  clip-path: inset(50%);
  width: 1px;
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
}
```

## 3️⃣ **리액트에서 숨김처리 컴포넌트 구현**

리액트에서 숨김처리를 하는 방법은 매우 간단하다. 한 페이지에는 수많은 요소가 있는 만큼, 대체 텍스트로 정보를 제공해줘야 하는 요소가 많기 때문에 재사용 가능한 컴포넌트를 생성해주었다.

```jsx
components / common / A11yHidden;

export default function A11yHidden({ children }) {
  return <SrOnly>{children}</SrOnly>;
}

const SrOnly = styled.span`
  clip: rect(1px, 1px, 1px, 1px);
  clip-path: inset(50%);
  width: 1px;
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
`;
```

`Styled-components`를 활용하여 모든 자식요소를 받아 컨텐츠를 숨길 수 있는 컴포넌트이다.

`줍깅` 프로젝트에도 해당 컴포넌트를 활용하여 h태그, 버튼 등 추가적인 정보가 필요하다고 생각되는 곳에 아래와 같이 추가해줬다.

### 🔸활용 예시

```jsx
<CardContainer>
  <h2>
    <A11yHidden>플로깅 등급 정보</A11yHidden>
  </h2>
  <ContentContainer>
      ...
      <ContentTitle>등급</ContentTitle>
      <TierInfoBtn onClick={handleOpenModal}>
        <A11yHidden>모든 플로깅 등급 정보 보기</A11yHidden>
      </TierInfoBtn>
</CardContainer>;
```

회원의 플로깅 등급 정보를 제공해수는 모달 창이다. 실제 모달에는 제목 텍스트가 없지만, 어떤 모달인지 알려주기 위해 A11yHidden 컴포넌트를 활용하여 h2태그를 추가해줬다.

또한 시각 장애인은 버튼의 종류를 알기 어렵기 때문에 Button 컴포넌트 내부에 어떤 버튼인지 알려주는 텍스트도 추가해줬다.

추가적으로 tailwindCSS를 활용한다면 `sr-only`라는 className만 추가해주면 따로 CSS를 정의하지 않아도 사용 가능하다.

- tailwindCSS에서 사용하는 CSS 속성

```css
position: absolute;
width: 1px;
height: 1px;
padding: 0;
margin: -1px;
overflow: hidden;
clip: rect(0, 0, 0, 0);
white-space: nowrap;
border-width: 0;
```

## 🔗 참고

- <a href="https://accessibility.naver.com/">https://accessibility.naver.com/</a>
- <a href="https://omins.tistory.com/90">https://omins.tistory.com/90</a>
- <a href="https://tailwindcss.com/docs/screen-readers">https://tailwindcss.com/docs/screen-readers</a>
