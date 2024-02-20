---
title: "[Refectoring] 이미지 렌더링 최적화하기"
excerpt: "이미지 렌더링 속도를 줄여보자"

categories:
  - Etc
tags:
  - [react, refectoring]

permalink: /etc/image-rendering-refectoring/

toc: true
toc_sticky: true

date: 2024-01-09
last_modified_at: 2024-01-09
---

## 1️⃣ 렌더링 속도 체크 ✔️

'줍깅' 프로젝트를 마친 후 가장 리팩토링 하고싶었던 부분이 이미지 최적화였다. 체감 상으로도 너무 늦게 렌더링되어 불편함을 느꼈고 실제로도 측정 결과 LCP에서 낮은 점수가 나왔기 때문에 가장 먼저 최적화를 진행해야겠다고 마음 먹었다. (무엇보다 성질 급한 한국인으로서 이미지가 느리게 뜨는건 참을 수 없었기에..😇)

최적화 전 정확한 수치를 확인하기 위해 크롬 개발자도구를 통해 속도 검사를 했다.
(모든 검사는 피드 페이지를 중점으로 했다)

<img src="https://github.com/FRONTENDSCHOOL7/jubging/assets/97607752/0a8b7fbe-175a-49d6-9500-6180bcad501a" width="500" />

측정 결과 `LCP`는 6.6s, `TBT`는 480ms

<img src="https://github.com/FRONTENDSCHOOL7/jubging/assets/97607752/10d7eb70-1dd2-4f0e-867e-08d24f5c9c78" width="400" />

역시나 이미지 용량 때문에 낮은 점수를 받은 것을 볼 수 있었다.

<img src="https://github.com/FRONTENDSCHOOL7/jubging/assets/97607752/61607d75-14e7-4c18-842a-ab487d9cac0f" width="400" />

네트워크 탭에서 총 29개의 이미지를 불러오는데 걸린 속도는 `1.8s`

> 측정 시 반드시 브라우저 위 새로고침 아이콘에서 <mark>'캐시 비우기 및 강력 새로고침'</mark> 으로 새로고침 해야 정확한 측정이 가능하다.

## 2️⃣ 이미지 최적화 진행시켜!

### 🔸WebP 포맷 사용

이미지 용량을 줄이기 위해 기존에 사용했던 png/svg에서 `WebP`로 교체했다.

> **WepP 란?**
> 2010년 구글에서 개발한 이미지 포맷으로 VP8이라는 비디오 코덱 기술을 비한으로 한 영상압축 방식을 사용하여 파일크기를 25%~35% 정도로 압축할 수 있다.
>
> - 동일한 품질의 이미지라도 JPEG와 PNG에 비해 평균 25~34% 작은 파일을 가져 웹페이지의 로딩 속도를 향상시키고 데이터 사용량을 감소시킬 수 있음
> - PNG와 같은 투명 이미지 표현이 가능하며 GIF처럼 애니메이션 처리도 가능
> - IE를 제외한 웹 브라우저에서 지원

가장 최신 포맷인 AVIF는 WebP보다 뛰어난 성능을 가지고 있긴 하지만 아직 지원하지 않는 브라우저가 많아 위의 장점들을 고려하여 WebP를 선택하였다.

이미지 포맷 변경은 많은 사이트에서 지원하지만, 나는 구글에서 제공해주는 [squoosh](https://squoosh.app/)에서 webp로 변환하였다. 이미지마다 다르지만 크게는 96%부터 대부분 50% 이상의 용량을 줄일 수 있었다.

### 🔸svg 파일 압축

아이콘의 경우 이미지 깨짐을 방지하여 모두 SVG 포맷을 사용했다. 다만, 해당 프로젝트의 경우 스타일에 맞춰 직접 그려서 사용한 아이콘들도 많았는데, 다른 아이콘에 비해 용량이 10배 이상은 컸기 때문에 [svgomg](https://svgomg.net/)에서 압축을 했다.

사실 svg -> svg로 기존 포맷을 압축하는거라 큰 기대가 없었는데 30~50% 정도의 압축 효과를 볼 수 있었다!

### 🔸업로드 시 이미지 압축하기

로컬에 저장한 이미지들은 위와 같이 이미지 포맷 변경와 압축을 통해 용량을 줄일 수 있었다. 하지만 사용자가 직접 사진을 업로드 할 때는 어떻게 최적화를 할 수 있을지 고민하던 중 `browser-image-compression`라는 라이브러리를 알게 되었다.

먼저 라이브러리를 설치해준다.

```
npm install browser-image-compression --save
# or
yarn add browser-image-compression
```

`browser-image-compression`에서 제공하는 다양한 옵션을 통해 최대 용량이나 이미지의 최대 넓이/높이 값 등을 입력하여 원하는 크기로 이미지를 압축할 수 있다.

나같은 경우에는 회원가입 시 프로필 등록, 프로필 수정, 피드 게시글 작성 이렇게 총 3군데에서 이미지 업로드가 필요했기 때문에 중복 코드를 줄이기 위해 `useImageUploader` 커스텀훅을 만들어 적용하였다.

```jsx
import imageCompression from "browser-image-compression";

function useImageUploader() {
  const [image, setImage] = useState(null);

  const handleImgUpload = async (imageFile) => {
    const form = new FormData();

    // 이미지 압축 옵션 지정 (maxSizeMB나 maxWidthOrHeight를 필수적으로 지정해야함)
    const options = {
      maxSizeMB: 1,
      maxWidthOrHeight: 500,
    };

    // 파일 압축
    const compressedBlod = await imageCompression(imageFile, options);

    // File로 변환
    const compressedFile = new File([compressedBlod], imageFile.name, {
      type: compressedBlod.type,
      lastModified: imageFile.lastModified,
    });

    form.append("image", compressedFile);

    try {
      const imageData = await postImgUpload(form);
      const imageUrl = BASE_URL + imageData.filename;
      setImage(imageUrl);
      console.log(imageData);
    } catch (error) {
      console.log(error);
    }
  };

  return { image, setImage, handleImgUpload };
}

export default useImageUploader;
```

imageCompression 모듈은 압축 파일을 `Blob` 객체로 반환해준다. 내가 진행하는 프로젝트에서는 `File` 타입을 사용해야 하므로 File 생성자를 통해 변환해주었다.

<img src="https://github.com/hangnik/hangnik.github.io/assets/97607752/684f4ba3-6b3f-4bc5-899e-1d67d7dae53b" width="300" />
<img src="https://github.com/hangnik/hangnik.github.io/assets/97607752/0a2cb1cf-72d1-42e6-a744-f2a54bb8ffdf" width="300" />

원본 파일 크기와 리사이징한 파일 크기를 비교해보니 크기가 매우 줄어든 것을 확인할 수 있었다.

## 3️⃣ 결과

<img src="https://github.com/FRONTENDSCHOOL7/jubging/assets/97607752/18608ce9-5828-410b-ae2d-e9c1c85cf14c" width="500" />

LCP는 `6.6s -> 5.3s`로, TBT는 `480ms -> 340ms`로 단축!

<img src="https://github.com/FRONTENDSCHOOL7/jubging/assets/97607752/6c7ccf4d-cb1a-48c1-915f-330f05b9c5c4" width="500" />

네트워크탭에서의 측정 결과 역시 총 29개의 이미지 용량이 `9.5MB -> 5.9MB`로, 시간은 `1.81s -> 1.40s`로 단축되었다. 👏

이렇게 정량적인 지표 뿐만 아니라 눈으로 봤을 때도 확연한 차이를 느낄 수 있었다. 앞으로도 꾸준히 코드 개선을 통해 사용자 경험이 높아질 수 있도록 페이지 성능을 개선해야겠다!

## 🔗 참고

- <a href="https://oliveyoung.tech/blog/2021-11-22/how-to-improve-web-performance-with-image-optimization/">https://oliveyoung.tech/blog/2021-11-22/how-to-improve-web-performance-with-image-optimization/</a>
- <a href="https://www.npmjs.com/package/browser-image-compression">https://www.npmjs.com/package/browser-image-compression</a>
- <a href="https://web.dev/articles/choose-the-right-image-format?hl=ko">https://web.dev/articles/choose-the-right-image-format?hl=ko</a>
