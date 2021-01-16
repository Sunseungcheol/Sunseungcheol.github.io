---
layout: post
categories: React.js
tags: [React.js, TypeScript]
---
TypeScript 로 상태관리를 하는 법을 알아보기 위해 Counter 를 만들어 보자.

다른 건 거의 없지만 각 함수, 변수등이 모두 타입이 지정되 있는 것을 볼 수 있다.

```tsx
import React, { useState } from "react";

function Counter() {
  //useState 사용 시 제너릭스로 타입 지정
  const [count, setCount] = useState<number>(0);

  //아무것도 안받고 아무것도 리턴하지 않으므로 void
  const onIncrease = () => setCount(count + 1);
  const onDecrease = () => setCount(count - 1);

  return (
    <div>
      <h1>{count}</h1>
      <div>
        <button onClick={onIncrease}>+1</button>
        <button onClick={onDecrease}>-1</button>
      </div>
    </div>
  );
}

export default Counter;
```

이번에는 인풋 상태를 관리하고 submit 하는 이벤트를 관리해보자.

```typescript
//App.tsx
import React from "react";
import Counter from "./Counter";
import MyForm from "./MyForm";

const App: React.FC = () => {
  const onSubmit = (form: { name: string; description: string }) => {
    console.log(form);
  };
  return <MyForm onSubmit={onSubmit}></MyForm>;
};

export default App;

//MyForm.tsx
import React, { useState } from "react";

//props 타입 설정
type MyFormProps = {
  //props 로 onSubmit 이라는 함수, 함수는 form 을 파라미터로 가져오고 그 안의 name, description 값은 string, 함수호출 후 반환 값은 없음
  onSubmit: (form: { name: string; description: string }) => void;
};

function MyForm({ onSubmit }: MyFormProps) {
  const [form, setForm] = useState({
    name: "",
    description: "",
  });

  const { name, description } = form;

  const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setForm({
      ...form,
      [name]: value,
    });
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    onSubmit(form);
    setForm({
      name: "",
      description: "",
    });
  };

  return (
    //onSubmit, onChange 등에 마우스를 갖다대면 이벤트 객체의 타입이 나온다.
    <form onSubmit={handleSubmit}>
      <input type="text" name="name" value={name} onChange={onChange} />
      <input
        type="text"
        name="description"
        value={description}
        onChange={onChange}
      />
      <button type="submit">등록</button>
    </form>
  );
}

export default MyForm;
```