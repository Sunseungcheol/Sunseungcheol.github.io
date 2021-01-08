---
layout: post
categories: 알고리즘
tags: [알고리즘, 프로그래머스]
---

## 문제 설명
대문자와 소문자가 섞여있는 문자열 s가 주어집니다. s에 'p'의 개수와 'y'의 개수를 비교해 같으면 True, 다르면 False를 return 하는 solution를 완성하세요. 'p', 'y' 모두 하나도 없는 경우는 항상 True를 리턴합니다. 단, 개수를 비교할 때 대문자와 소문자는 구별하지 않습니다.

예를 들어 s가 pPoooyY면 true를 return하고 Pyy라면 false를 return합니다.

## 제한사항
문자열 s의 길이 : 50 이하의 자연수
문자열 s는 알파벳으로만 이루어져 있습니다.

## 입출력 예

<table class="table">
        <thead><tr>
<th>s</th>
<th>answer</th>
</tr>
</thead>
        <tbody><tr>
<td><q>pPoooyY</q></td>
<td>true</td>
</tr>
<tr>
<td><q>Pyy</q></td>
<td>false</td>
</tr>
</tbody>
      </table>

## 문제풀이

```javascript
function solution(s) {
  if (s.match(/P/gi) === null && s.match(/Y/gi) === null) {
    return true;
  }
  return s.match(/P/gi).length === s.match(/Y/gi).length;
}
```

처음에는 위와 같은 방법을 생각했다. match 함수를 이용해 길이를 구하는 건데 두가지 테스트 케이스에서 계속 런타임 오류가 낫다. 뭐가 문제인지 생각하다보니 s.match(/P/gi) 나 s.match(/Y/gi) 하나만 null 값이 나와도 값 비교가 되지 않는다. 그래서 아래와 같이 else if 를 추가해 줬다.

```javascript
function solution(s) {
  if (s.match(/P/gi) === null && s.match(/Y/gi) === null) {
    return true;
  } else if (s.match(/P/gi) === null || s.match(/Y/gi) === null) {
    return false;
  }
  return s.match(/P/gi).length === s.match(/Y/gi).length;
}
```

다만 문제를 계속 생각해봐도 더 코드를 깔끔하게 할 수 있을 것 같았다. 잠도 안오고 해서 문자열 개수를 셀 방법이 또 뭐가 있을까 생각하다보니 split 이 생각났다. 

```javascript
function solution(s) {
  const upperCase = s.toUpperCase();
  return upperCase.split("P").length === upperCase.split("Y").length;
}
```

먼저 대소문자를 하나로 통일시켜주고 split 을 통해 P, Y 를 기준으로 짤라 배열을 만들어 개수를 비교했다. 
어떤식으로 짤리는지 다시 한번 확인 하기 위해 여러 값들을 넣고 console 을 찍어보니 생각대로 개수가 나왔다.

P 나 Y 가 없는 경우 문자열을 그대로 배열로 반환하기 때문에 그 부분에서도 문제가 생기지 않았다.