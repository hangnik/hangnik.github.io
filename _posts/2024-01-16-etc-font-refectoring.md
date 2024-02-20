---
title: "[Refectoring] 폰트 로딩 속도 개선하기"
excerpt: "웹폰트를 사용하여 페이지 로딩 속도를 개선해보자"

categories:
  - Etc
tags:
  - [react, refectoring]

permalink: /etc/font-refectoring/

toc: true
toc_sticky: true

date: 2024-01-16
last_modified_at: 2024-01-16
---

## 1️⃣ 무엇이 문제였을까

이미지 렌더링 최적화까지 하고 나니 또 눈에 띄는 것이 폰트의 렌더링 속도였다.

프로젝트를 진행할 때 `지마켓산스`라는 폰트로 정하고난 뒤 어떻게 적용할지 고민을 했었다. 그 당시 찾아봤을 땐 다운은 TTF와 OTF 포맷 밖에 없었고, 웹 폰트도 고려해봤지만 굵기가 한정적이여서(인 줄 알아서...😅) 결국 OTF 파일을 다운 받은 후 프로젝트 내부에 폰트를 저장하는 방식으로 사용했다.

<img src="https://github.com/hangnik/hangnik.github.io/assets/97607752/fae3c0c3-e11f-4060-9239-ae200213ba42" width="600" />

OTF 포맷을 사용했을 때 사이즈는 모두 800KB가 넘었고, medium의 경우 `302ms` light는 `182ms`의 속도가 걸렸다.
또한 Bold 굵기를 사용하지 않음에도 파일 내에 저장해두고 있었다. 매번 불필요한 파일을 모두 불러와 로딩 속도를 더 늦추기만 했다.

### 🔸기존 폰트 적용 방식

```jsx
@font-face {
  font-family: "GmarketSansLight";
  src: url("../assets/fonts/GMARKETSANSLIGHT.OTF") format("opentype");
  font-weight: 300;
}

@font-face {
  font-family: "GmarketSansMedium";
  src: url("../assets/fonts/GMARKETSANSMEDIUM.OTF") format("opentype");
  font-weight: 500;
}

@font-face {
  font-family: "GmarketSansBold";
  src: url("../assets/fonts/GMARKETSANSBOLD.OTF") format("opentype");
  font-weight: 700;
}

```

기존에는 굵기별로 모두 font를 저장해둔 후 root에는 font-family를 `GmarketSansMedium`로, 다른 굵기를 적용시키고 싶은 곳에는 모두 font-family 속성을 사용하여 변경해주었다.

이렇게 했을 경우 문제점은 사용하지 않는 폰트의 굵기까지 모두 다운로드 된다는 것이다.

## 2️⃣ 웹폰트를 적용해보자!

### 🔸왜 웹폰트야?

웹 폰트는 온라인상에서 폰트를 다운로드해서 사용한다. 다운로드 한 폰트 파일을 캐시에 저장하고, 이 다음에 웹 페이지에서 해당 폰트를 사용할 때는 다시 다운로드 하는 것이 아니라 캐시된 폰트 파일을 사용하게 된다. 따라서 웹 페이지에서 폰트 로딩 속도를 빠르게 할 수 있다.

### 🔸어떤 포맷을 사용해야 될까?

`WOFF(Web Open Font Format)`는 기존의 폰트 형식인 OTF, TTF와 동일하게 작동하지만, 압축을 통해 더 작은 파일 크기를 가진다. 2010년에 첫 출시되어 현재는 거의 모든 브라우저에서 지원한다.

이와 비슷한 `WOFF2(Web Open Font Format 2)`는 이름에서 유추할 수 있듯이 WOFF를 개선한 버전이다. Brotli 압축 알고리즘을 사용하여 WOFF보다 약 30%에서 50% 정도 더 작은 파일 크기를 가진다. 원래는 WOFF보다 브라우저 호환성이 낮았지만, 현재는 1~2개 빼곤 모두 지원하기 때문에 WOFF2의 사용을 권장한다. 보통 WOFF2를 우선으로 로드하고 WOFF를 풀백 폰트로 지정하는 추세인 것 같다.

하지만 아쉽게도 프로젝트에서 사용하는 지마켓산스는 WOFF 포맷만 지원해서 WOFF만 적용하였다. 🥲

### 🔸웹폰트로 변경 후 폰트 적용 방식

```jsx
@font-face {
  font-family: "GmarketSans";
  src: url("https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2001@1.1/GmarketSansLight.woff")
    format("woff");
    // 풀백 포맷을 사용하고 싶다면 src 속성에 사용하고 싶은 폰트 url과 format을 추가로 입력해주면 된다.
  font-weight: 300;
}

@font-face {
  font-family: "GmarketSans";
  src: url("https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2001@1.1/GmarketSansMedium.woff")
    format("woff");
  font-weight: 500;
}

// root 지정
font-family: 'GmarketSans', serif;
```

기존과 달리 font-family를 모두 `GmarketSans`로 지정한 후 font-weight 속성으로 굵기를 변경하였다.

## 3️⃣ 결과

  <img src="https://github.com/hangnik/hangnik.github.io/assets/97607752/a5fd665c-8a14-40a8-a517-32396d6196e2" width="600" />

렌더링 속도 측정 결과 사이즈는 medium은 `302ms -> 102ms`로 light는 `182ms -> 46ms`로 약 3배의 단축 효과를 볼 수 있었다! (최조 실행 시)

<img src="https://github.com/hangnik/hangnik.github.io/assets/97607752/9722c94b-7a10-4212-b4ba-3886d0ee28cc" width="800" />

한 번 렌더링을 한 후 다시 측정해보니 size는 memory cache로, time은 0에 수렴하는 것을 확인할 수 있었다.
