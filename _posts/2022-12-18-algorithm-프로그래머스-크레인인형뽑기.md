---
title: "[Algorithm] 프로그래머스 크레인 인형뽑기 게임 - JS"
excerpt: "프로그래머스 Level2. 크레인 인형뽑기 게임 풀이"

categories:
  - Algorithm
tags:
  - [algorithm, programmers]

permalink: /algorithm/프로그래머스-크레인인형뽑기/

toc: true
toc_sticky: true

date: 2023-12-18
last_modified_at: 2023-12-18
---

## 문제

> https://school.programmers.co.kr/learn/courses/30/lessons/64061

## 풀이

```js
function solution(board, moves) {
  let basket = [];
  let count = 0; // 제거한 인형의 개수

  for (let i = 0; i < moves.length; i++) {
    // 배열의 인덱스는 0부터 시작하기 때문에 moves의 위치에서 -1 빼주기
    let position = moves[i] - 1;

    for (let j = 0; j < board.length; j++) {
      // 타겟 위치 정하기
      let target = board[j][position];

      // 타켓이 0일 경우 빈 칸이기 때문에 넘어가기
      if (target === 0) {
        continue;
      } else {
        // 0이 아닐 경우
        // 바구니 마지막에 있는 숫자와 타겟 숫자가 같으면 바구니에서 제거하고 count +2
        if (basket[basket.length - 1] === target) {
          basket.pop();
          count += 2;
          // 다를 경우 바구니에 push
        } else {
          basket.push(target);
        }
        // board에서 타겟 인형을 제거했으므로 빈 칸으로 만들기
        board[j][position] = 0;
        break;
      }
    }
  }
  return count;
}
```

처음에 문제를 잘못 이해해서 시간이 좀 걸렸습니다.. 먼저 주어진 board 배열이 아래부터 쌓이는 줄 알았는데 정말 보이는 그대로 위에서부터 쌓이는 것이고, 리턴 값이 사라진 인형의 개수가 아니라 남아있는 인형의 개수인 줄 알았...
다음부터는 시간을 낭비하는 일이 없도록 문제와 예시를 꼼꼼히 읽어야겠다고 다짐했던 문제였습니다 🫠
