---
layout: post
categories: 알고리즘
tags: [알고리즘, 프로그래머스]
---

## 문제 설명

함수 solution은 정수 x와 자연수 n을 입력 받아, x부터 시작해 x씩 증가하는 숫자를 n개 지니는 리스트를 리턴해야 합니다. 다음 제한 조건을 보고, 조건을 만족하는 함수, solution을 완성해주세요.

## 제한사항

x는 -10000000 이상, 10000000 이하인 정수입니다.
n은 1000 이하인 자연수입니다.

## 입출력 예

<table class="table">
        <thead><tr>
<th>x</th>
<th>n</th>
<th>answer</th>
</tr>
</thead>
        <tbody><tr>
<td>2</td>
<td>5</td>
<td>[2,4,6,8,10]</td>
</tr>
<tr>
<td>4</td>
<td>3</td>
<td>[4,8,12]</td>
</tr>
<tr>
<td>-4</td>
<td>2</td>
<td>[-4, -8]</td>
</tr>
</tbody>
      </table>

## 문제풀이

```javascript
function solution(x, n) {
    var answer = [];
    for(let i=1;i<=n;i++){
        answer.push(i*x)
    }
    return answer;
}
```

반복문에서 i를 1부터 시작해 n 개와 같을 때까지 돌려주고 answer 에 i*x 를 넣어주었다.