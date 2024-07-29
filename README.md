# React-Hooks

## State Hooks
State lets a component “remember” information like user input. For example, a form component can use state to store the input value, while an image gallery component can use state to store the selected image index.

To add state to a component, use one of these Hooks:

- useState declares a state variable that you can update directly.
- useReducer declares a state variable with the update logic inside a reducer function.

### useState
useState is a React Hook that lets you add a state variable to your component.
```typescript
const [state, setState] = useState(initialState)
```

#### Basic use case
```typescript
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      You pressed me {count} times
    </button>
  );
}
```
![image](https://github.com/user-attachments/assets/abfc570c-5fe4-4696-989c-8075fa9dde7d)

#### Updating objects and arrays in state 
```typescript
import { useState } from 'react';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(initialTodos);

  function handleAddTodo(title) {
    setTodos([
      ...todos,
      {
        id: nextId++,
        title: title,
        done: false
      }
    ]);
  }

  function handleChangeTodo(nextTodo) {
    setTodos(todos.map(t => {
      if (t.id === nextTodo.id) {
        return nextTodo;
      } else {
        return t;
      }
    }));
  }

  function handleDeleteTodo(todoId) {
    setTodos(
      todos.filter(t => t.id !== todoId)
    );
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```
![image](https://github.com/user-attachments/assets/bf7af98e-3eec-470e-a2c5-1eadf1e35d94)

#### Updating state based on the previous state 
Wrong:
```typescript
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```
Correct:
```typescript
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```
When you use setAge(age + 1), you are directly using the current value of age from the closure at the time the component renders. If multiple setAge(age + 1) calls are made sequentially in a single event handler, they all use the same initial state value because the state updates are batched and applied asynchronously.

When you use setAge(prevAge => prevAge + 1), you are using the functional form of the state updater function. This form takes the previous state value (prevAge) as an argument and returns the new state value. This ensures that each state update is based on the most recent state, even if multiple updates are queued.

#### Avoiding recreating the initial state 
Bad Practice:
```typescript
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
```
Good Practice:
```typescript
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
```
Although the result of createInitialTodos() is only used for the initial render, you’re still calling this function on every render. This can be wasteful if it’s creating large arrays or performing expensive calculations.

To solve this, you may pass it as an initializer function to useState instead. Notice that you’re passing createInitialTodos, which is the function itself, and not createInitialTodos(), which is the result of calling it. If you pass a function to useState, React will only call it during initialization.

## Effect Hooks
Effects let a component connect to and synchronize with external systems. This includes dealing with network, browser DOM, animations, widgets written using a different UI library, and other non-React code.
- useEffect connects a component to an external system.
- useLayoutEffect fires before the browser repaints the screen. You can measure layout here.
- useInsertionEffect fires before React makes changes to the DOM. Libraries can insert dynamic CSS here.

### useEffect
useEffect is a React Hook that lets you synchronize a component with an external system.
```typescript
useEffect(setup, dependencies?)
```
#### Fetching data with Effects 
```typescript
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    async function startFetching() {
      setBio(null);
      const result = await fetchBio(person);
      if (!ignore) {
        setBio(result);
      }
    }

    let ignore = false;
    startFetching();
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```
![image](https://github.com/user-attachments/assets/8542f0a7-5e79-422b-8f5a-813046ff52ec)

#### Passing Reactive Dependencies
1. Passing a dependency array
```typescript
useEffect(() => {
  // ...
}, [a, b]); // Runs again if a or b are different
```
If you specify the dependencies, your Effect runs after the initial render and after re-renders with changed dependencies.

2. Passing an empty dependency array
```typescript
useEffect(() => {
  // ...
}, []); // Does not run again (except once in development)
```
If your Effect truly doesn’t use any reactive values, it will only run after the initial render.

3. Passing no dependency array
```typescript
useEffect(() => {
  // ...
}); // Always runs again
```
If you pass no dependency array at all, your Effect runs after every single render (and re-render) of your component.

## Context Hooks 
Context lets a component receive information from distant parents without passing it as props. For example, your app’s top-level component can pass the current UI theme to all components below, no matter how deep.

- useContext reads and subscribes to a context.

### useContext
useContext is a React Hook that lets you read and subscribe to context from your component.
```typescript
const value = useContext(SomeContext)
```

#### Passing data deeply into the tree 
Call useContext at the top level of your component to read and subscribe to context.

useContext returns the context value for the context you passed. To determine the context value, React searches the component tree and finds the closest context provider above for that particular context.

To pass context to a Button, wrap it or one of its parent components into the corresponding context provider. It doesn’t matter how many layers of components there are between the provider and the Button. When a Button anywhere inside of Form calls useContext(ThemeContext), it will receive "dark" as the value.

```typescript
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  )
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```
![image](https://github.com/user-attachments/assets/4cbd762b-28d2-4819-8490-2652a3aaa279)



