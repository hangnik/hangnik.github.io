## Github Blog
### ▪ 포스트 작성

1. `_posts/YYYY-MM-DD-post-name-here.md` 파일 생성
2. 포스트에 사용할 이미지는 `assets/images/posts_img/post-name-here/` 하위에 저장
3. 포스트 front matter 작성

```txt
---
title: "[포스팅 예시] 이곳에 제목을 입력하세요"
excerpt: "본문의 주요 내용을 여기에 입력하세요"

categories: # 카테고리 설정
  - categories1
tags: # 포스트 태그
  - [tag1, tag2]

permalink: /categories1/post-name-here/ # 포스트 URL

toc: true # 우측에 본문 목차 네비게이션 생성
toc_sticky: true # 본문 목차 네비게이션 고정 여부

date: 2020-05-21 # 작성 날짜
last_modified_at: 2021-10-09 # 최종 수정 날짜
---
```
4. front matter 하단에 포스팅 내용 작성
