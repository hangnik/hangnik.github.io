---
title: "[React] 버튼 컴포넌트 재사용하기"
excerpt: "재사용 가능한 버튼 컴포넌트를 만들어보자"

categories:
  - React
tags:
  - [react, component]

permalink: /react/button-component-reuse/

toc: true
toc_sticky: true

date: 2023-11-24
last_modified_at: 2023-12-04
---

## 1️⃣ **기존에 사용한 방식**

기존에 프로젝트에서 버튼 컴포넌트를 구성할 때는 props를 사용하여 컴포넌트를 생성했습니다.

```jsx
// ButtonContainer.jsx

export default function ButtonContainer({
  children,
  type,
  color,
  bgColor,
  width,
  height,
  disabled = false,
  onClick,
  rmargin,
  fontSize,
  border,
  hoverFilter,
}) {
  return (
    <Button
      type={type || "button"}
      onClick={onClick}
      $color={color}
      $bgColor={bgColor}
      $width={width}
      $height={height}
      $disabled={disabled}
      $rmargin={rmargin}
      $fontSize={fontSize}
      $border={border}
      $hoverFilter={hoverFilter}
    >
      {children}
    </Button>
  );
}
```

```jsx
// ButtonContainerStyle.jsx

export const Button = styled.button`
  width: ${(props) => (props.$width ? props.$width : "120px")};
  height: ${(props) => (props.$height ? props.$height : "44px")};
  background-color: ${(props) =>
    props.$bgColor ? props.$bgColor : props.theme.colors.mainColor};
  color: ${(props) => (props.$color ? props.$color : "#ffffff")};
  border-radius: 30px;
  margin-right: ${(props) => (props.$rmargin ? props.$rmargin : "")};
  font-size: ${(props) =>
    props.$fontSize ? props.$fontSize : props.theme.fontSize.small};
  border: ${(props) => (props.$border ? props.$border : null)};

  &:hover {
    filter: ${(props) => (props.$hoverFilter ? "brightness(0.9)" : null)};
    transition: all 0.2s ease-in-out;
  }
`;
```

기본 스타일을 styled-component을 사용하여 지정하고, 스타일링에 필요한 요소들을 props로 받아온 후 필요한 스타일을 버튼마다 적용시켜주었습니다.

> 하지만, 이런식으로 기본 스타일을 지정해주니 버튼마다 크기도 스타일도 다른 경우가 많아 추가되는 코드가 오히려 더 많아지고 너무 많아진 props 때문에 한번에 알아보기가 어려웠습니다. 🤧

리팩토링을 고민하던 중 한 블로그에서 css 함수를 사용한 방법을 보고 방식을 바꿔봤습니다.

## 2️⃣ **CSS 함수 사용**

### ✔️ 기본 스타일링

버튼 컴포넌트에 공통적으로 필요한 스타일을 적용시켜줍니다. 이때 버튼 종류에 따라 바뀔 수 있는 속성값은 CSS 변수를 사용해줍니다.

```jsx
function Button({ disabled, children, ...props }) {
  return (
    <StyledButton disabled={disabled} {...props}>
      {children}
    </StyledButton>
  );
}

const StyledButton = styled.button`
  margin: 0;
  font: inherit;
  cursor: pointer;
  background: var(--button-background);
  border: var(--button-border);
  color: var(--button-color);
  padding: var(--button-padding);
  height: var(--button-height);
  line-height: var(--button-height);
  font-size: var(--button-font-size);
  display: inline-flex;
  justify-content: center;
  align-items: center;
  border-radius: 2.75rem;
  transition: all 0.3s ease-out;

  &:disabled {
    --button-background: ${(props) => props.theme.colors.disabledColor};
    --button-color: ${(props) => props.theme.colors.whiteColor};
    cursor: not-allowed;
  }

  &:hover {
    filter: brightness(0.9);
  }
`;
```

### ✔️ 크기 스타일링

크기에 따른 버튼 컴포넌트를 생성하기 위해 `size` 라는 prop을 추가해줍니다. 이때 styled-components의 css 함수를 사용하여 `SIZES`라는 자바스크립트 객체에 저장해주었습니다.

```jsx
import styled, { css } from "styled-components";

const SIZES = {
  sm: css`
    --button-font-size: ${(props) => props.theme.fontSize.xsmall};
    --button-padding: 0 0.85rem;
    --button-height: 1.75rem;
  `,
  md: css`
    --button-font-size: ${(props) => props.theme.fontSize.small};
    --button-padding: 0.5rem;
    --button-height: 2.15rem;
  `,
  lg: css`
    --button-font-size: ${(props) => props.theme.fontSize.medium};
    --button-padding: 0 1.15rem;
    --button-height: 2.75rem;
  `,
};
```

### ✔️ 변형 스타일링

성공, 오류, 경고 등 여러 상황에 따라 활용하거나 버튼 UI가 여러 타입일 때 사용할 수 있는 버튼 컴포넌트입니다. `variant`라는 prop을 추가해준 뒤 css 함수를 사용하여 `VARIANTS`라는 자바스크립트 객체에 저장해줍니다.

```jsx
import styled, { css } from "styled-components";

const VARIANTS = {
  primary: css`
    --button-color: ${(props) => props.theme.colors.whiteColor};
    --button-background: ${(props) => props.theme.colors.mainColor};
    --button-border: 1px solid ${(props) => props.theme.colors.mainColor};
  `,
  white: css`
    --button-color: ${(props) => props.theme.colors.blackColor};
    --button-background: ${(props) => props.theme.colors.whiteColor};
    --button-border: 1px solid ${(props) => props.theme.colors.placeHolderColor};
  `,
};
```

size, variant prop을 받아서 저장해둔 객체에 따라 해당하는 스타일을 적용하기 위해 기본 `StyledButton` 컴포넌트에 아래처럼 추가해줍니다.

```jsx
function Button({ disabled, size, variant, children, ...props }) {
  return (
    <StyledButton disabled={disabled} size={size} variant={variant} {...props}>
      {children}
    </StyledButton>
  );
}

const StyledButton = styled.button`
  ${({ size }) => SIZES[size]}
  ${({ variant }) => VARIANTS[variant]} /* 이하 생략 */
`;
```

### 3️⃣ 사용하기

```jsx
<Button type="button" size="lg" variant="white">
  회원가입
</Button>
```

이렇게 사이즈, 상황 별로 변수를 지정해주니 props만 보고 어떤 버튼인지 한눈에 파악할 수 있고, 중복되는 코드 또한 줄일 수 있었습니다. 👍

위의 분류 말고도 필요한 상황에 따라 변수를 추가하여 버튼 컴포넌트를 분류해주면 좋을 것 같습니다.

## 🔗 참고

- [React로 버튼 컴포넌트 만들기](https://www.daleseo.com/react-button-component/)
