---
layout: post
categories: React.js
tags: [React.js, TypeScript, ToDoList]
---
javascript 로 만드는 ToDoList 와 크게 다른 점은 없다.

```ts
//modules/todos.ts
const ADD_TODO = "todos/ADD_TODO" as const;
const TOGGLE_TODO = "todos/TOGGLE_TODO" as const;
const REMOVE_TODO = "todos/REMOVE_TODO" as const;

let nextId = 1;

export const addTodo = (text: string) => ({
  type: ADD_TODO,
  payload: {
    id: nextId++,
    text,
  },
});

export const toggleTodo = (id: number) => ({
  type: TOGGLE_TODO,
  payload: id,
});

export const removeTodo = (id: number) => ({
  type: REMOVE_TODO,
  payload: id,
});

//액션에 대한 타입스크립트 타입
type TodosAction =
  | ReturnType<typeof addTodo>
  | ReturnType<typeof toggleTodo>
  | ReturnType<typeof removeTodo>;

//상태 할일 목록 타입
export type Todo = {
  id: number;
  text: string;
  done: boolean;
};

//초기 상태에 대한 타입
type TodosState = Todo[];

const initialState: TodosState = [];

//리듀서
function todos(
  state: TodosState = initialState,
  action: TodosAction
): TodosState {
  switch (action.type) {
    case ADD_TODO:
      return state.concat({
        id: action.payload.id,
        text: action.payload.text,
        done: false,
      });
    case TOGGLE_TODO:
      return state.map((todo) =>
        todo.id === action.payload ? { ...todo, done: !todo.done } : todo
      );
    case REMOVE_TODO:
      return state.filter((todo) => todo.id !== action.payload);
    default:
      return state;
  }
}

export default todos;

//modules/index.js
import { combineReducers } from "redux";
import counter from "./counter";
import todos from "./todos";

const rootReducer = combineReducers({
  counter,
  todos,
});

export default rootReducer;
//리덕스에서 관리하는 상태에 대한 타입
export type RootState = ReturnType<typeof rootReducer>;
```

몇가지 다른 점은 파라미터를 받을 때 타입을 설정해주는 것과, 기본 값과 액션에 대한 타입을 지정해준다는 점이다.

이제 프리젠테이셔널 컴포넌트를 만들어보자.

```ts
//components/TodoInsert.tsx
import React, { FormEvent, useState } from "react";

type TodoInsertProps = {
  onInsert: (text: string) => void;
};

function TodoInsert({ onInsert }: TodoInsertProps) {
  const [value, setValue] = useState("");
  const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };
  const onSubmit = (e: FormEvent) => {
    e.preventDefault();
    onInsert(value);
    setValue("");
  };

  return (
    <form onSubmit={onSubmit}>
      <input
        placeholder="할 일을 입력하세요"
        value={value}
        onChange={onChange}
      />
      <button type="submit">등록</button>
    </form>
  );
}

export default TodoInsert;

//components/TodoItem.tsx
import React, { CSSProperties } from "react";
import { Todo } from "../modules/todos";

type TodoItemProps = {
  todo: Todo;
  onToggle: (id: number) => void;
  onRemove: (id: number) => void;
};

function TodoItem({ todo, onToggle, onRemove }: TodoItemProps) {
  const handleToggle = () => onToggle(todo.id);
  const handleRemove = () => onRemove(todo.id);

  const textStyle: CSSProperties = {
    textDecoration: todo.done ? "line-through" : "none",
  };

  const removeStyle: CSSProperties = {
    color: "red",
    marginLeft: 8,
  };

  return (
    <li>
      <span onClick={handleToggle} style={textStyle}>
        {todo.text}
      </span>
      <span onClick={handleRemove}>삭제</span>
    </li>
  );
}

export default TodoItem;

//components/TodoList.tsx
import React from "react";
import { Todo } from "../modules/todos";
import TodoItem from "./TodoItem";

type TodoListProps = {
  todos: Todo[];
  onToggle: (id: number) => void;
  onRemove: (id: number) => void;
};

function TodoList({ todos, onToggle, onRemove }: TodoListProps) {
  if (todos.length === 0) return <p>등록된 항목이 없습니다.</p>;

  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem
          todo={todo}
          onToggle={onToggle}
          onRemove={onRemove}
          key={todo.id}
        ></TodoItem>
      ))}
    </ul>
  );
}

export default TodoList;
```

컴포넌트의 props 에 대한 타입을 지정해줘야 한다.

이제 컨테이너컴포넌트를 만들고 내보자보자

```ts
//container/TodoApp.tsx
import React from "react";
import { useDispatch, useSelector } from "react-redux";
import TodoInsert from "../components/TodoInserts";
import TodoList from "../components/TodoList";
import { RootState } from "../modules";
import { addTodo, removeTodo, toggleTodo } from "../modules/todos";

function TodoApp() {
  const todos = useSelector((state: RootState) => state.todos);
  const dispatch = useDispatch();

  const onInsert = (text: string) => {
    dispatch(addTodo(text));
  };
  const onToggle = (id: number) => {
    dispatch(toggleTodo(id));
  };
  const onRemove = (id: number) => {
    dispatch(removeTodo(id));
  };

  return (
    <>
      <TodoInsert onInsert={onInsert}></TodoInsert>
      <TodoList
        todos={todos}
        onToggle={onToggle}
        onRemove={onRemove}
      ></TodoList>
    </>
  );
}

export default TodoApp;

```