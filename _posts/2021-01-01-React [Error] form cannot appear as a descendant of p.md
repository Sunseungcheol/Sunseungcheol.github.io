---
layout: post
categories: React.js
tags: [React.js, Error]
---

react 로 가계부를 만들어 보던 도중 콘솔창에 아래와 같은 에러가 떳다.

```
Warning: validateDOMNesting(...): <form> cannot appear as a descendant of <p>.
    in form (at HouseholdCreate.js:120)
    in p (at Dialog.js:54)
    in div (created by styled.div)
    in styled.div (at Dialog.js:52)
    in div (created by styled.div)
    in styled.div (at Dialog.js:51)
    in Dialog (at HouseholdCreate.js:114)
    in HouseholdCreate (at App.js:34)
    in div (created by styled.div)
    in styled.div (at Template.js:19)
    in Template (at App.js:30)
    in HouseProvider (at App.js:28)
    in Ge (at App.js:19)
    in App (at src/index.js:7)
```

콘솔창에만 뜬거라 현재 상태에서 구동에 문제는 없지만 언젠가 문제가 생기기 마련이다. 
p태그는 내부에 인라인 요소만 가질 수 있기에 나온 문제다. 어디서 꼬인건지 찾아보니 친절하게 어디 부분이 문제인지 알려준다. 확인을 해보자.

```javascript
//Dialog.js
function Dialog({
  title,
  children,
  confirmText,
  cancelText,
  visible,
  onConfirm,
  onCancle,
}) {
  if (!visible) return null;
  return (
    <DarkBkg>
      <DialogBlock>
        <h3>{title}</h3>        
        <p>{children}</p>{/* 54번줄 */}
        <ButtonGroup>
          <Button color="blue" onClick={onCancle}>
            {cancelText}
          </Button>
          <Button color="pink" onClick={onConfirm}>
            {confirmText}
          </Button>
        </ButtonGroup>
      </DialogBlock>
    </DarkBkg>
  );
}
```

공용으로 사용하려고 만들어 놓은 dialog 인데 뒷일은 생각못하고 children 을 p 태그 안에 넣어두었다.

HouserholdCreate.js 에서 아래와 같은 방식으로 작성하고 있어서 나온 에러였다. 

```javascript
<Dialog>
  <form action=""></form>
</Dialog>
```
