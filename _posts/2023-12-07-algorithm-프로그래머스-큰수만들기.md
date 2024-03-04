---
title: "[Algorithm] 프로그래머스 큰 수 만들기 - JS"
excerpt: "프로그래머스 Level2. 큰 수 만들기 풀이 (그리디)"

categories:
  - Algorithm
tags:
  - [algorithm, programmers]

permalink: /algorithm/프로그래머스-큰수만들기/

toc: true
toc_sticky: true

date: 2023-12-07
last_modified_at: 2023-12-07
---

## 문제

> https://school.programmers.co.kr/learn/courses/30/lessons/42883#qna

## 풀이

처음에는 number를 split 한 후 숫자를 조합해 그 중 가장 큰 숫자를 리턴하는 방법으로 하려 했으나... 조건이 1,000,000 자리 이하인 숫자인 것을 보고 포기,,! ^\_ㅠ

결국 검색을 해서 <mark>스택</mark>을 이용하는 방법을 참고했다.

```js
function solution(number, k) {
  const stack = [];

  for (let i = 0; i < number.length; i++) {
    const el = number[i];

    // k(제거해야 될 숫자 개수)가 아직 남아있고, stack에 들어있는 마지막 숫자가 현재 값(el) 보다 작다면 제거하고 현재값을 stack에 push
    while (k > 0 && stack[stack.length - 1] < el) {
      stack.pop();
      k--;
    }
    stack.push(el);
  }

  // k>0일 경우, stack 끝에 값만 없애주기
  stack.splice(stack.length - k, k);
  return stack.join("");
}
```

## 🔗 참고

- https://taesung1993.tistory.com/46
