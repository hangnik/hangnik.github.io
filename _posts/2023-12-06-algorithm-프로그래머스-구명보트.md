---
title: "[Algorithm] 프로그래머스 구명보트 - JS"
excerpt: "프로그래머스 Level2. 구명보트 풀이 (그리디)"

categories:
  - Algorithm
tags:
  - [algorithm, programmers]

permalink: /algorithm/프로그래머스-구명보트/

toc: true
toc_sticky: true

date: 2023-12-06
last_modified_at: 2023-12-06
---

## 문제

> https://school.programmers.co.kr/learn/courses/30/lessons/42885

## 풀이

처음에는 먼저 내림차순으로 정렬 후 앞에서부터 두명씩 묶어 limit보다 작으면 구멍보트 개수를 추가하고 넘으면 다른 조건식을 만들어주는 식으로 했다.
하지만 이렇게 하다보니 조건이 너무 많이 생겨 몇몇 테스트케이스는 통과하지 못했고, 효율성 테스트 또한 통과하지 못했다,,😢

결국 힌트를 본 후 먼저 가장 가벼운 무게와 가장 무거운 무게를 합친 후 비교하고 인덱스를 앞으로 땡기는 식으로 방법을 바꿨다! (투 포인터)

```js
function solution(people, limit) {
  let count = 0;
  // 내림차순으로 정렬
  people = people.sort((a, b) => a - b);

  // 가장 가벼운 인덱스
  let minIdx = 0;
  // 가장 무거운 인덱스
  let maxIdx = people.length - 1;

  while (minIdx < maxIdx) {
    // 가장 가벼운 사람과 무거운 사람을 합쳤을 때 limit보다 무거우면 무거운 사람 먼저 태우기
    if (people[minIdx] + people[maxIdx] > limit) {
      maxIdx--;
      // 무게가 넘치지 않으면 둘다 태우기
    } else {
      minIdx++;
      maxIdx--;
    }
    count++;

    // max와 min 인덱스가 같아지면 한 사람이 남으므로 보트 1개 추가
    if (maxIdx === minIdx) {
      count++;
    }
  }

  return count;
}
```
