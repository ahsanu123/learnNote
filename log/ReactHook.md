### Summary React Hook karangan Sendiri
- Suspense: butuh throw (dan wrap promise) untuk mengetahui apakah komponen siap render
- Lazy: akan diimport ketika perlu (promise juga)
- useTransition: return 2 value [isPending, startTransition], ispending adalah boolean yg bernilai true jika `startTransition` Masih bekerja, startTransition memiliki prioritas yg rendah dibandingkan fungsi lain yg tidak berada dalam start transition

## 24 Agustus 2023
### UseTransition
useTransition is a React Hook that lets you update the state without blocking the UI.
```typescript
const [isPending, startTransition] = useTransition()
```
----
Parameters

useTransition does not take any parameters.

----
Returns

useTransition returns an array with exactly two items:

1. The isPending flag that tells you whether there is a pending transition.
2. The startTransition function that lets you mark a state update as a transition.


### UseMemo
useMemo is a React Hook that lets you cache the result of a calculation between re-renders.
```typescript
const cachedValue = useMemo(calculateValue, dependencies)
```
----
Parameters

calculateValue: The function calculating the value that you want to cache. It should be pure, should take no arguments, and should return a value of any type. React will call your function during the initial render. On next renders, React will return the same value again if the dependencies have not changed since the last render. Otherwise, it will call calculateValue, return its result, and store it so it can be reused later.  


 dependencies: The list of all reactive values referenced inside of the calculateValue code. Reactive values include props, state, and all the variables and functions declared directly inside your component body. If your linter is configured for React, it will verify that every reactive value is correctly specified as a dependency. The list of dependencies must have a constant number of items and be written inline like [dep1, dep2, dep3]. React will compare each dependency with its previous value using the Object.is comparison.
 
----
Returns

On the initial render, useMemo returns the result of calling calculateValue with no arguments.

During next renders, it will either return an already stored value from the last render (if the dependencies haven’t changed), or call calculateValue again, and return the result that calculateValue has returned.

<details>
 <summary> Pemahaman sendiri tentang useMemo </summary>
 useMemo memiliki 2 parameter `calculateValue dan Dependencies` ketika `todos` atau `tab` berubah (lihat kode dibawah ini), use memo akan menggunakan fungsi pada argument pertama untuk menghitung ulang/re-calculate `visibleTodos` karena salah satu dependencies berubah, disini theme tidak digunakan sebagai dependencies, sehingga ketika `theme` berubah argument pertama tidak akan di re-execute, **namun** ketika `theme` dimasukan ke **array** dependencies useMemo akan tetap melakukan re-calculate walaupun data tidak terpengaruh oleh "theme".

 ### memo
 memo lets you skip re-rendering a component when its props are unchanged.
 ```typescript
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```
 
 ```typescript
 function TodoList({ todos, theme, tab }: { todos: TodosModel[], theme: string, tab: string }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );

  return (
    <div className={theme} >
      <List items={visibleTodos} />
    </div>
  );
}

const MemoList = ({ items }: { items: TodosModel[] }) => {
  console.log('[ARTIFICIALLY SLOW] Rendering <List /> with ' + items.length + ' items');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Do nothing for 500 ms to emulate extremely slow code
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.completed ?
            <s>{item.text}</s> :
            item.text
          }
        </li>
      ))}
    </ul>
  );

}

const List = memo(MemoList);


 ```
</details>


### useReducer
useReducer is a React Hook that lets you add a reducer to your component.
```typescript
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

----
Parameters

  reducer: The reducer function that specifies how the state gets updated. It must be pure, should take the state and action as arguments, and should return the next state. State and action can be of any types.  

  initialArg: The value from which the initial state is calculated. It can be a value of any type. How the initial state is calculated from it depends on the next init argument. optional init: The initializer function that should return the initial state. If it’s not specified, the initial state is set to initialArg. Otherwise, the initial state is set to the result of calling init(initialArg).

----
Returns

useReducer returns an array with exactly two values:

The current state. During the first render, it’s set to init(initialArg) or initialArg (if there’s no init). The dispatch function that lets you update the state to a different value and trigger a re-render.

<details>
<summary>Converted JS Example to Typescript</summary>

```typescript
import { useReducer, useState } from 'react';

interface Task {
  id: number;
  text: string;
  done: boolean;
}

interface AddTaskProps {
  onAddTask: (text: string) => void;
}

export default function AddTask({ onAddTask }: AddTaskProps) {
  const [text, setText] = useState('');

  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}
      >
        Add
      </button>
    </>
  );
}

interface TaskListProps {
  tasks: Task[];
  onChangeTask: (task: Task) => void;
  onDeleteTask: (taskId: number) => void;
}

export default function TaskList({ tasks, onChangeTask, onDeleteTask }: TaskListProps) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

interface TaskProps {
  task: Task;
  onChange: (task: Task) => void;
  onDelete: (taskId: number) => void;
}

function Task({ task, onChange, onDelete }: TaskProps) {
  const [isEditing, setIsEditing] = useState(false);

  let taskContent;

  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Save</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Edit</button>
      </>
    );
  }

  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>Delete</button>
    </label>
  );
}

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text: string) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task: Task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId: number) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

type Action =
  | { type: 'added'; id: number; text: string }
  | { type: 'changed'; task: Task }
  | { type: 'deleted'; id: number };

function tasksReducer(tasks: Task[], action: Action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;

const initialTasks: Task[] = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];
```
</details>

## 25 Agustus 2023
### useContext
useContext is a React Hook that lets you read and subscribe to context from your component.
```typescript
const value = useContext(SomeContext)
```

----

Parameters

SomeContext: The context that you’ve previously created with createContext. The context itself does not hold the information, it only represents the kind of information you can provide or read from components.

----

Returns  

useContext returns the context value for the calling component. It is determined as the value passed to the closest SomeContext.Provider above the calling component in the tree. If there is no such provider, then the returned value will be the defaultValue you have passed to createContext for that context. The returned value is always up-to-date. React automatically re-renders components that read some context if it changes.
