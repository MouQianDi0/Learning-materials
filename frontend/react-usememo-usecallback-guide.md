# React `useMemo` 与 `useCallback` 详细指南

> 最核心的一句话：
>
> - `useMemo` 缓存“计算结果”。
> - `useCallback` 缓存“函数引用”。
>
> 它们都是性能优化工具，不是 React 组件正常工作的必要条件。

---

## 目录

- [一、为什么需要它们](#一为什么需要它们)
- [二、先理解引用相等](#二先理解引用相等)
- [三、useMemo：缓存计算结果](#三usememo缓存计算结果)
- [四、useCallback：缓存函数引用](#四usecallback缓存函数引用)
- [五、useCallback 为什么常与 React.memo 配合](#五usecallback-为什么常与-reactmemo-配合)
- [六、三者之间的关系](#六三者之间的关系)
- [七、完整组合案例](#七完整组合案例)
- [八、依赖数组应该怎么写](#八依赖数组应该怎么写)
- [九、旧闭包问题](#九旧闭包问题)
- [十、使用函数式更新减少依赖](#十使用函数式更新减少依赖)
- [十一、对象依赖的常见陷阱](#十一对象依赖的常见陷阱)
- [十二、在 useEffect 场景中的选择](#十二在-useeffect-场景中的选择)
- [十三、不要滥用 useMemo 和 useCallback](#十三不要滥用-usememo-和-usecallback)
- [十四、什么时候适合使用](#十四什么时候适合使用)
- [十五、常见误区](#十五常见误区)
- [十六、Strict Mode 为什么可能执行两次](#十六strict-mode-为什么可能执行两次)
- [十七、React Compiler 会带来什么变化](#十七react-compiler-会带来什么变化)
- [十八、实际开发中的判断流程](#十八实际开发中的判断流程)
- [十九、总结速查表](#十九总结速查表)
- [参考资料](#参考资料)

---

## 一、为什么需要它们

React 函数组件每次重新渲染时，组件函数都会重新执行。

```tsx
function App() {
  const user = {
    name: "Timmmi",
  };

  function handleClick() {
    console.log("点击了");
  }

  return <button onClick={handleClick}>{user.name}</button>;
}
```

每次 `App` 重新渲染时：

- `{ name: "Timmmi" }` 会创建一个新对象；
- `handleClick` 会产生一个新的函数引用；
- 组件中的普通计算会重新执行；
- JSX 也会重新计算。

这本身不是错误。

React 的设计就是在状态变化后重新执行组件函数，根据最新数据计算新的界面。大多数普通计算和普通函数的创建都很快，不需要专门优化。

但是，在下面这些场景中，重复计算或不稳定的引用可能带来额外开销：

- 有一个耗时明显的计算，每次渲染都重新执行；
- 一个新对象、新数组或新函数被传给经过 `React.memo` 优化的子组件；
- 对象或函数被放入 `useEffect`、`useMemo` 等 Hook 的依赖数组；
- 父组件频繁更新，而子组件的渲染成本较高；
- 页面中有大量数据过滤、排序或转换操作。

`useMemo` 和 `useCallback` 就是在这些特定情况下使用的优化工具。

---

## 二、先理解引用相等

理解这两个 Hook 之前，需要先理解 JavaScript 中的“引用”。

### 1. 基本类型比较的是值

```ts
1 === 1; // true
"hello" === "hello"; // true
true === true; // true
```

### 2. 对象、数组和函数比较的是引用

```ts
{} === {}; // false
[] === []; // false
(() => {}) === (() => {}); // false
```

虽然两个对象的内容一样，但它们是在不同位置创建的两个对象。

```ts
const userA = {
  name: "Timmmi",
};

const userB = {
  name: "Timmmi",
};

console.log(userA === userB); // false
```

只有两个变量指向同一个对象时，引用比较才会相等：

```ts
const userA = {
  name: "Timmmi",
};

const userB = userA;

console.log(userA === userB); // true
```

数组和函数也是同样的规则。

```ts
const listA = [1, 2, 3];
const listB = listA;

console.log(listA === listB); // true

const handleA = () => {
  console.log("点击");
};

const handleB = handleA;

console.log(handleA === handleB); // true
```

在 React 中，“内容看起来一样”和“引用完全相同”是两件不同的事。

`useMemo` 可以让一个计算结果的引用在依赖不变时保持稳定；`useCallback` 可以让一个函数引用在依赖不变时保持稳定。

---

## 三、useMemo：缓存计算结果

### 1. 基本语法

```tsx
const result = useMemo(() => {
  return 计算结果;
}, [依赖项]);
```

例如：

```tsx
const total = useMemo(() => {
  return price * count;
}, [price, count]);
```

`useMemo` 接收两个参数：

1. 一个计算函数；
2. 一个依赖数组。

它的运行过程可以理解为：

1. 第一次渲染时，执行计算函数；
2. 保存计算结果；
3. 后续渲染时，比较依赖项；
4. 依赖没有变化，返回上一次保存的结果；
5. 依赖发生变化，重新执行计算并保存新结果。

React 使用 `Object.is` 逐项比较依赖。

### 2. 不使用 useMemo 的笔记搜索

假设 IRisNote 的笔记页面中有搜索功能：

```tsx
import { useState } from "react";

type Note = {
  id: number;
  title: string;
};

function NotesPage({ notes }: { notes: Note[] }) {
  const [keyword, setKeyword] = useState("");
  const [theme, setTheme] = useState<"light" | "dark">("light");

  const visibleNotes = notes.filter((note) => {
    console.log("正在过滤笔记");

    return note.title
      .toLowerCase()
      .includes(keyword.toLowerCase());
  });

  return (
    <div className={theme}>
      <input
        value={keyword}
        onChange={(event) => {
          setKeyword(event.target.value);
        }}
      />

      <button
        onClick={() => {
          setTheme(theme === "light" ? "dark" : "light");
        }}
      >
        切换主题
      </button>

      {visibleNotes.map((note) => (
        <p key={note.id}>{note.title}</p>
      ))}
    </div>
  );
}
```

这里即使用户只是切换主题，`NotesPage` 也会重新渲染，于是下面的过滤操作会再次执行：

```tsx
notes.filter(...);
```

但是，过滤结果实际上只和两个数据有关：

- `notes`；
- `keyword`。

它和 `theme` 没有关系。

### 3. 使用 useMemo 缓存过滤结果

```tsx
import { useMemo, useState } from "react";

type Note = {
  id: number;
  title: string;
};

function NotesPage({ notes }: { notes: Note[] }) {
  const [keyword, setKeyword] = useState("");
  const [theme, setTheme] = useState<"light" | "dark">("light");

  const visibleNotes = useMemo(() => {
    console.log("正在过滤笔记");

    return notes.filter((note) =>
      note.title
        .toLowerCase()
        .includes(keyword.toLowerCase()),
    );
  }, [notes, keyword]);

  return (
    <div className={theme}>
      <input
        value={keyword}
        onChange={(event) => {
          setKeyword(event.target.value);
        }}
      />

      <button
        onClick={() => {
          setTheme(theme === "light" ? "dark" : "light");
        }}
      >
        切换主题
      </button>

      {visibleNotes.map((note) => (
        <p key={note.id}>{note.title}</p>
      ))}
    </div>
  );
}
```

现在的执行情况是：

| 发生的变化 | 是否重新过滤 |
| --- | ---: |
| `keyword` 变化 | 是 |
| `notes` 变化 | 是 |
| `theme` 变化 | 否 |
| 父组件重渲染，但 `notes` 引用不变 | 否 |

### 4. useMemo 不只是避免重复计算

`useMemo` 还可以让对象或数组的引用保持稳定。

```tsx
const visibleNotes = notes.filter(...);
```

每次执行 `filter` 都会产生一个新数组：

```ts
const resultA = notes.filter(...);
const resultB = notes.filter(...);

console.log(resultA === resultB); // false
```

使用 `useMemo` 后，只要依赖没有变化，React 返回的就是上一次保存的那个数组：

```tsx
const visibleNotes = useMemo(() => {
  return notes.filter(...);
}, [notes, keyword]);
```

这对于 `React.memo` 子组件非常重要，因为子组件接收到的数组 prop 可以保持同一个引用。

### 5. useMemo 的第一次渲染不会更快

第一次渲染时，计算函数仍然必须执行：

```tsx
const result = useMemo(() => {
  return expensiveCalculation(data);
}, [data]);
```

`useMemo` 优化的是后续依赖不变时的重新渲染，不会让第一次计算消失。

---

## 四、useCallback：缓存函数引用

### 1. 基本语法

```tsx
const cachedFunction = useCallback(() => {
  // 函数内容
}, [依赖项]);
```

例如：

```tsx
const handleSave = useCallback(() => {
  saveNote(noteId);
}, [noteId]);
```

`useCallback` 不会立即执行 `handleSave`。

它只是返回一个函数：

- 依赖没有变化时，返回上一次保存的函数引用；
- 依赖发生变化时，返回当前渲染中的新函数。

### 2. 普通函数为什么可能导致引用变化

```tsx
function Parent() {
  const [count, setCount] = useState(0);

  const handleDelete = (id: number) => {
    console.log("删除笔记", id);
  };

  return <NoteList onDelete={handleDelete} />;
}
```

每次 `Parent` 重新渲染，下面这段代码都会再次执行：

```tsx
const handleDelete = (id: number) => {
  console.log("删除笔记", id);
};
```

因此每次都会得到一个新的函数引用。

可以把它理解为：

```ts
const firstFunction = (id: number) => {};
const secondFunction = (id: number) => {};

console.log(firstFunction === secondFunction); // false
```

虽然它们的代码一模一样，但在 JavaScript 看来，它们是两个不同的函数对象。

### 3. 使用 useCallback

```tsx
import { useCallback, useState } from "react";

function Parent() {
  const [count, setCount] = useState(0);

  const handleDelete = useCallback((id: number) => {
    console.log("删除笔记", id);
  }, []);

  return <NoteList onDelete={handleDelete} />;
}
```

这里的依赖数组是 `[]`，因为函数内部没有读取会变化的 props、state 或组件内部变量。

父组件重新渲染时：

```ts
previousHandleDelete === currentHandleDelete;
// true
```

更准确地说，在每次渲染期间，传给 `useCallback` 的函数表达式仍然会被计算出来；但当依赖不变时，React 会忽略当前函数并返回上一次缓存的函数引用。

因此，`useCallback` 的核心价值是：

> 稳定函数引用，而不是阻止函数代码被创建，也不是自动阻止组件渲染。

---

## 五、useCallback 为什么常与 React.memo 配合

这是 `useCallback` 最重要的使用场景之一。

### 1. 只有 useCallback，但子组件没有 memo

```tsx
function NoteList({
  onDelete,
}: {
  onDelete: (id: number) => void;
}) {
  console.log("NoteList 渲染");

  return (
    <button onClick={() => onDelete(1)}>
      删除第一条笔记
    </button>
  );
}
```

即使父组件使用了：

```tsx
const handleDelete = useCallback(() => {}, []);
```

父组件重新渲染时，`NoteList` 默认仍然会跟着重新渲染。

所以，单独使用 `useCallback` 不一定能减少子组件渲染。

### 2. 配合 React.memo

```tsx
import { memo, useCallback, useState } from "react";

type NoteListProps = {
  onDelete: (id: number) => void;
};

const NoteList = memo(function NoteList({
  onDelete,
}: NoteListProps) {
  console.log("NoteList 渲染");

  return (
    <button onClick={() => onDelete(1)}>
      删除第一条笔记
    </button>
  );
});

export default function NotesPage() {
  const [count, setCount] = useState(0);

  const handleDelete = useCallback((id: number) => {
    console.log("删除笔记：", id);
  }, []);

  return (
    <>
      <p>计数：{count}</p>

      <button
        onClick={() => {
          setCount(count + 1);
        }}
      >
        增加计数
      </button>

      <NoteList onDelete={handleDelete} />
    </>
  );
}
```

点击“增加计数”以后：

1. `NotesPage` 重新渲染；
2. `handleDelete` 的引用没有变化；
3. `NoteList` 被 `memo` 包裹；
4. `memo` 比较新旧 props；
5. `onDelete` 仍然是同一个函数；
6. `NoteList` 可以跳过这次重新渲染。

`React.memo` 默认使用 `Object.is` 比较每一个 prop。

但是要注意：

- 子组件自己的 state 变化时，仍然会重新渲染；
- 子组件使用的 context 变化时，仍然会重新渲染；
- 任意一个 prop 变成新引用，都可能打破这次 memo 优化。

---

## 六、三者之间的关系

| API | 缓存或比较什么 | 主要用途 |
| --- | --- | --- |
| `useMemo` | 计算结果、对象、数组或其他值 | 避免重复计算，保持值的引用稳定 |
| `useCallback` | 函数引用 | 保持传给子组件或其他 Hook 的函数稳定 |
| `React.memo` | 组件接收到的 props | props 不变时跳过一次子组件渲染 |

可以这样记：

```tsx
// 缓存计算结果
const data = useMemo(() => calculateData(), []);

// 缓存函数引用
const handleClick = useCallback(() => {
  doSomething();
}, []);

// props 不变时，允许跳过组件渲染
const Child = memo(function Child() {
  return <div>Child</div>;
});
```

从概念上看：

```tsx
useCallback(fn, dependencies);
```

近似于：

```tsx
useMemo(() => fn, dependencies);
```

区别是 `useCallback` 专门用来表达“我要缓存一个函数引用”，写法也更直接。

---

## 七、完整组合案例

下面用一个笔记列表案例，同时观察 `useMemo`、`useCallback` 和 `React.memo` 的配合。

```tsx
import {
  memo,
  useCallback,
  useMemo,
  useState,
} from "react";

type Note = {
  id: number;
  title: string;
};

type NoteListProps = {
  notes: Note[];
  onSelect: (id: number) => void;
};

const NoteList = memo(function NoteList({
  notes,
  onSelect,
}: NoteListProps) {
  console.log("NoteList 重新渲染");

  return (
    <div>
      {notes.map((note) => (
        <button
          key={note.id}
          onClick={() => {
            onSelect(note.id);
          }}
        >
          {note.title}
        </button>
      ))}
    </div>
  );
});

export default function NotesPage({
  notes,
}: {
  notes: Note[];
}) {
  const [keyword, setKeyword] = useState("");
  const [theme, setTheme] = useState<"light" | "dark">("light");
  const [selectedId, setSelectedId] = useState<number | null>(
    null,
  );

  // 缓存过滤结果。
  // theme 和 selectedId 变化时，不需要重新过滤。
  const visibleNotes = useMemo(() => {
    console.log("重新过滤笔记");

    const normalizedKeyword = keyword
      .trim()
      .toLowerCase();

    return notes.filter((note) =>
      note.title.toLowerCase().includes(normalizedKeyword),
    );
  }, [notes, keyword]);

  // React 的 state setter 引用稳定。
  // 函数没有读取其他会变化的响应式值，因此依赖数组为空。
  const handleSelect = useCallback((id: number) => {
    setSelectedId(id);
  }, []);

  return (
    <div className={theme}>
      <input
        value={keyword}
        onChange={(event) => {
          setKeyword(event.target.value);
        }}
      />

      <button
        onClick={() => {
          setTheme((currentTheme) =>
            currentTheme === "light" ? "dark" : "light",
          );
        }}
      >
        切换主题
      </button>

      <p>当前笔记：{selectedId ?? "未选择"}</p>

      <NoteList
        notes={visibleNotes}
        onSelect={handleSelect}
      />
    </div>
  );
}
```

### 当 theme 变化时

- 父组件会重新渲染；
- `notes` 和 `keyword` 没变，所以 `visibleNotes` 的引用不变；
- `handleSelect` 的依赖没变，所以函数引用不变；
- `NoteList` 收到的两个 props 都没有变化；
- `React.memo` 可以让 `NoteList` 跳过这次渲染。

### 如果去掉 useMemo

```tsx
const visibleNotes = notes.filter(...);
```

每次父组件渲染都会产生新数组，`React.memo` 会发现 `notes` prop 的引用变化，因此子组件仍会渲染。

### 如果去掉 useCallback

```tsx
const handleSelect = (id: number) => {
  setSelectedId(id);
};
```

每次父组件渲染都会产生新函数，`React.memo` 会发现 `onSelect` prop 的引用变化，因此子组件仍会渲染。

### 如果去掉 React.memo

即使 `visibleNotes` 和 `handleSelect` 的引用都很稳定，父组件重新渲染时，子组件默认仍会跟着渲染。

这说明三者扮演的是不同角色，不能互相完全替代。

---

## 八、依赖数组应该怎么写

基本原则是：

> Hook 内部使用到的、来自组件作用域并且可能变化的响应式值，都应该写入依赖数组。

响应式值通常包括：

- props；
- state；
- 在组件内部声明的变量；
- 在组件内部声明的函数。

例如：

```tsx
function Product({
  productId,
  userId,
}: {
  productId: number;
  userId: number;
}) {
  const handleBuy = useCallback(() => {
    buyProduct(productId, userId);
  }, [productId, userId]);

  return <button onClick={handleBuy}>购买</button>;
}
```

函数内部使用了：

- `productId`；
- `userId`。

因此依赖数组应该是：

```tsx
[productId, userId];
```

### 依赖变化意味着什么

```tsx
const handleBuy = useCallback(() => {
  buyProduct(productId, userId);
}, [productId, userId]);
```

只要 `productId` 或 `userId` 中任意一个发生变化，React 就会返回新的 `handleBuy` 函数。

这是正确行为，因为函数需要使用最新的商品 ID 和用户 ID。

### 不要为了“保持引用不变”而故意漏写依赖

错误：

```tsx
const handleBuy = useCallback(() => {
  buyProduct(productId, userId);
}, []); // 错误：漏掉了 productId 和 userId
```

这样做可能让函数一直使用第一次渲染时的数据。

应当遵循 React Hooks ESLint 规则给出的依赖提示，除非你非常清楚为什么某个值不属于响应式依赖。

---

## 九、旧闭包问题

遗漏依赖经常会造成 stale closure，也就是“闭包拿到了旧值”。

```tsx
import { useCallback, useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  const printCount = useCallback(() => {
    console.log(count);
  }, []); // 错误：函数使用了 count，却没有声明依赖

  return (
    <>
      <p>{count}</p>

      <button onClick={() => setCount(count + 1)}>
        增加
      </button>

      <button onClick={printCount}>
        打印 count
      </button>
    </>
  );
}
```

第一次渲染时：

```ts
count === 0;
```

由于依赖数组是空数组，`printCount` 一直复用第一次渲染时创建的函数。即使页面上的 `count` 已经变成 `5`，它也可能仍然打印：

```txt
0
```

正确写法：

```tsx
const printCount = useCallback(() => {
  console.log(count);
}, [count]);
```

现在 `count` 变化时，React 会返回一个读取最新 `count` 的新函数。

不要把“函数引用始终不变”当成最高目标。数据正确永远比 memo 优化重要。

---

## 十、使用函数式更新减少依赖

看下面的代码：

```tsx
const addNote = useCallback(
  (newNote: Note) => {
    setNotes([...notes, newNote]);
  },
  [notes],
);
```

因为函数读取了外部的 `notes`，所以依赖数组中必须包含 `notes`。

这意味着每次笔记数组变化时，`addNote` 都会变成新函数。

可以改用函数式更新：

```tsx
const addNote = useCallback((newNote: Note) => {
  setNotes((currentNotes) => [
    ...currentNotes,
    newNote,
  ]);
}, []);
```

这里不再直接读取组件当前渲染中的 `notes`，而是让 React 把最新状态作为参数传进来：

```tsx
currentNotes;
```

因此可以安全使用空依赖数组。

### 两种写法对比

读取外部 state：

```tsx
setNotes([...notes, newNote]);
```

需要依赖：

```tsx
[notes];
```

使用函数式更新：

```tsx
setNotes((currentNotes) => [
  ...currentNotes,
  newNote,
]);
```

不再读取外部 `notes`，因此可以减少这个依赖。

这在将回调函数传给 `memo` 子组件时非常实用。

---

## 十一、对象依赖的常见陷阱

下面的 `options` 每次渲染都会重新创建：

```tsx
function SearchPage({ keyword }: { keyword: string }) {
  const options = {
    caseSensitive: false,
    keyword,
  };

  const result = useMemo(() => {
    return search(options);
  }, [options]);

  return <SearchResult result={result} />;
}
```

虽然 `keyword` 可能没有变化，但是：

```ts
previousOptions === currentOptions; // false
```

所以 `useMemo` 每次都会重新计算，优化被新对象引用破坏了。

### 解决方式一：把对象放进 useMemo 内部

```tsx
const result = useMemo(() => {
  const options = {
    caseSensitive: false,
    keyword,
  };

  return search(options);
}, [keyword]);
```

这是更直接、更推荐的写法，因为真正影响计算的是 `keyword`。

### 解决方式二：先缓存对象

```tsx
const options = useMemo(() => {
  return {
    caseSensitive: false,
    keyword,
  };
}, [keyword]);

const result = useMemo(() => {
  return search(options);
}, [options]);
```

只有在其他位置也需要稳定的 `options` 引用时，才有必要使用这种方式。

### 同样的问题也会出现在数组和函数上

```tsx
const ids = [1, 2, 3];
const config = { pageSize: 20 };
const transform = (value: number) => value * 2;
```

这些值都在每次渲染时创建新引用。如果它们被用作依赖项，相关 Hook 就会在每次渲染时重新执行。

---

## 十二、在 useEffect 场景中的选择

函数或对象有时会成为 `useEffect` 的依赖。

### 1. 使用 useCallback 稳定函数

```tsx
function NotesPage({ userId }: { userId: number }) {
  const [notes, setNotes] = useState<Note[]>([]);

  const loadNotes = useCallback(async () => {
    const result = await getNotes(userId);
    setNotes(result);
  }, [userId]);

  useEffect(() => {
    loadNotes();
  }, [loadNotes]);

  return <NoteList notes={notes} />;
}
```

这里 `loadNotes` 依赖 `userId`：

- `userId` 不变时，`loadNotes` 引用不变；
- `userId` 变化时，`loadNotes` 变成新函数；
- Effect 随后重新执行，加载新用户的笔记。

### 2. 如果函数只在 Effect 中使用，直接写进 Effect

```tsx
function NotesPage({ userId }: { userId: number }) {
  const [notes, setNotes] = useState<Note[]>([]);

  useEffect(() => {
    async function loadNotes() {
      const result = await getNotes(userId);
      setNotes(result);
    }

    loadNotes();
  }, [userId]);

  return <NoteList notes={notes} />;
}
```

这种写法依赖更直接，也不需要额外的 `useCallback`。

### 3. 对象只在 Effect 中使用时，也可以直接写进去

不推荐：

```tsx
const options = useMemo(() => {
  return {
    roomId,
    serverUrl: "https://example.com",
  };
}, [roomId]);

useEffect(() => {
  const connection = createConnection(options);
  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [options]);
```

更直接：

```tsx
useEffect(() => {
  const options = {
    roomId,
    serverUrl: "https://example.com",
  };

  const connection = createConnection(options);
  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

当一个函数或对象只服务于某个 Effect 时，把它移动到 Effect 内部，通常比增加 memo 更简单。

---

## 十三、不要滥用 useMemo 和 useCallback

### 1. 简单计算不需要 useMemo

下面的计算非常轻：

```tsx
const fullName = firstName + lastName;
```

没有必要改成：

```tsx
const fullName = useMemo(() => {
  return firstName + lastName;
}, [firstName, lastName]);
```

`useMemo` 本身也需要：

- 创建依赖数组；
- 保存缓存；
- 比较依赖项；
- 增加代码复杂度。

简单计算直接执行通常更清楚。

### 2. 普通点击事件不一定需要 useCallback

```tsx
function LoginButton() {
  const handleClick = () => {
    console.log("登录");
  };

  return <button onClick={handleClick}>登录</button>;
}
```

如果这个函数：

- 没有传给 `memo` 子组件；
- 没有作为其他 Hook 的依赖；
- 创建成本很低；
- 没有造成可测量的性能问题；

那么就没有必要使用 `useCallback`。

### 3. 新函数本身通常不是性能问题

不要看到组件里有函数就立刻套上 `useCallback`：

```tsx
const handleClick = () => {
  setOpen(true);
};
```

创建普通函数一般很便宜。

真正要关注的是：

- 新函数是否破坏了子组件的 `memo`；
- 是否导致 Effect 反复执行；
- 是否处在高频渲染且已测量到性能问题的路径中。

### 4. 一个永远变化的 prop 就能打破整个 memo

```tsx
const MemoChild = memo(Child);

function Parent() {
  const stableHandler = useCallback(() => {}, []);

  return (
    <MemoChild
      onClick={stableHandler}
      options={{ color: "blue" }}
    />
  );
}
```

虽然 `onClick` 引用稳定，但 `options` 每次都是新对象。

因此 `MemoChild` 仍然会重新渲染。

如果确实需要优化，可以稳定 `options`：

```tsx
const options = useMemo(() => {
  return {
    color: "blue",
  };
}, []);
```

但在动手优化前，应当先确认子组件渲染是否真的昂贵。

---

## 十四、什么时候适合使用

### 适合使用 useMemo 的情况

#### 1. 计算确实比较重

```tsx
const sortedData = useMemo(() => {
  return largeData
    .filter((item) => item.enabled)
    .sort((a, b) => b.score - a.score)
    .map((item) => ({
      id: item.id,
      label: item.name,
    }));
}, [largeData]);
```

#### 2. 需要稳定对象或数组，传给 memo 子组件

```tsx
const visibleNotes = useMemo(() => {
  return filterNotes(notes, keyword);
}, [notes, keyword]);

return <MemoNoteList notes={visibleNotes} />;
```

#### 3. 需要稳定另一个 Hook 的依赖项

```tsx
const options = useMemo(() => {
  return { roomId };
}, [roomId]);
```

不过，如果对象只在 Effect 中使用，优先考虑把对象放进 Effect 内部。

### 适合使用 useCallback 的情况

#### 1. 函数传给 memo 子组件

```tsx
const handleDelete = useCallback((id: number) => {
  deleteNote(id);
}, []);

return <MemoNoteList onDelete={handleDelete} />;
```

#### 2. 函数是其他 Hook 的依赖项

```tsx
const loadNotes = useCallback(async () => {
  const result = await getNotes(userId);
  setNotes(result);
}, [userId]);

useEffect(() => {
  loadNotes();
}, [loadNotes]);
```

如果函数只被这个 Effect 使用，通常可以直接将函数写进 Effect。

#### 3. 自定义 Hook 对外返回函数

```tsx
function useNotes() {
  const deleteNote = useCallback((id: number) => {
    // 删除逻辑
  }, []);

  return {
    deleteNote,
  };
}
```

这样，使用这个自定义 Hook 的组件更容易继续进行引用稳定性优化。

---

## 十五、常见误区

### 误区一：useMemo 可以阻止组件重新渲染

不可以。

```tsx
useMemo(...);
```

它只能缓存一个值。组件该重新渲染时仍然会重新渲染。

负责根据 props 判断是否跳过子组件渲染的是：

```tsx
React.memo;
```

### 误区二：useCallback 可以阻止父组件重新渲染

不可以。

它只是让某个函数引用在依赖不变时保持稳定。

### 误区三：使用 useCallback 就一定更快

不一定。

如果子组件没有 `memo`，或者函数没有作为其他 Hook 的依赖项，缓存这个函数可能没有实际收益。

### 误区四：useMemo 适合执行副作用

不适合。

不要这样写：

```tsx
useMemo(() => {
  fetch("/api/notes");
}, []);
```

网络请求、订阅、定时器等副作用应该使用 `useEffect`：

```tsx
useEffect(() => {
  fetch("/api/notes");
}, []);
```

`useMemo` 的计算函数应该保持纯粹：

- 相同输入产生相同结果；
- 不修改外部数据；
- 不在计算期间执行副作用。

### 误区五：useMemo 是永久存储

不是。

它是性能缓存，React 在特定情况下可以丢弃缓存。程序的正确性不能依赖 `useMemo` 一直保留某个值。

如果数据需要影响页面并触发渲染，使用：

```tsx
useState;
```

如果只是需要跨渲染保存，变化时不触发渲染，使用：

```tsx
useRef;
```

### 误区六：第一次渲染也会更快

不会。

第一次渲染时，`useMemo` 仍然必须执行计算。它只可能减少后续依赖不变时的重复计算。

### 误区七：依赖越少越好

不对。

依赖数组不是“优化参数”，而是用来描述 Hook 读取了哪些响应式数据。

漏写依赖可能导致旧闭包、旧数据和难以排查的逻辑错误。

### 误区八：只要内容相同，memo 就会认为 prop 相同

不对。

默认情况下，`memo` 不会深度比较对象内容：

```tsx
{ name: "A" } !== { name: "A" };
```

两个内容相同但引用不同的对象，仍会被视为不同的 prop。

---

## 十六、Strict Mode 为什么可能执行两次

在 React Strict Mode 的开发环境中，React 可能会额外调用 `useMemo` 的计算函数，用来帮助发现不纯的计算逻辑。

```tsx
const result = useMemo(() => {
  console.log("计算");
  return calculateData(data);
}, [data]);
```

开发环境中可能看到两次日志，但这不代表生产环境也会以同样方式重复调用。

因此计算函数必须是纯函数，不能依赖“只执行一次”。

错误示例：

```tsx
const result = useMemo(() => {
  // 错误：修改了外部数组。
  data.push(newItem);
  return data;
}, [data, newItem]);
```

更合适的写法：

```tsx
const result = useMemo(() => {
  // 创建新数组，不修改原数组。
  return [...data, newItem];
}, [data, newItem]);
```

如果额外执行导致结果错误，通常说明计算逻辑中包含了不应该存在的副作用。

---

## 十七、React Compiler 会带来什么变化

React Compiler 可以自动处理许多值、函数和组件的记忆化，从而减少手动编写 `useMemo`、`useCallback` 和 `memo` 的需要。

但需要注意：

- 项目没有启用 React Compiler 时，仍然按照传统规则手动优化；
- 启用了 Compiler，也不代表可以忽略组件纯度和正确依赖；
- 不要仅仅因为 API 存在，就在所有地方手动 memo；
- 应优先写清楚、正确、纯粹的组件，再根据实际情况优化。

即使使用 Compiler，理解这些 API 仍然重要，因为你需要能够：

- 阅读现有 React 项目；
- 判断依赖数组是否正确；
- 排查旧闭包；
- 理解引用稳定性；
- 分析组件为什么重新渲染；
- 维护没有启用 Compiler 的代码。

---

## 十八、实际开发中的判断流程

写代码时，可以按下面的顺序判断。

### 第一步：现在真的存在性能问题吗

如果页面没有明显卡顿，组件渲染也不昂贵，先保持代码简单。

### 第二步：是否存在耗时计算

例如：

- 大数组过滤；
- 多次排序；
- 大量数据转换；
- 复杂递归计算；
- 高频渲染时的明显耗时操作。

如果确实存在，并且依赖不会每次都变化，可以考虑 `useMemo`。

### 第三步：是否存在昂贵子组件的无意义渲染

先判断子组件是否真的昂贵，再考虑：

```tsx
memo(Component);
```

### 第四步：传给 memo 子组件的对象或数组是否每次都是新引用

如果是，并且它破坏了已经有价值的 `memo` 优化，可以考虑：

```tsx
useMemo;
```

### 第五步：传给 memo 子组件的函数是否每次都是新引用

如果是，可以考虑：

```tsx
useCallback;
```

### 第六步：是不是普通小计算或普通点击事件

如果只是：

```tsx
const total = price * count;
```

或者：

```tsx
const handleClick = () => {
  setOpen(true);
};
```

通常直接写就够了。

### 第七步：用工具验证，而不是凭感觉优化

可以使用：

- React DevTools Profiler；
- 浏览器 Performance 面板；
- `console.time` 和 `console.timeEnd`；
- 生产构建；
- 接近真实用户性能的设备。

例如：

```tsx
console.time("filter notes");

const visibleNotes = filterNotes(notes, keyword);

console.timeEnd("filter notes");
```

先确认耗时，再决定是否需要缓存。

---

## 十九、总结速查表

### 1. 一句话总结

```tsx
useMemo(() => value, [dependencies]);
// 记住计算结果

useCallback(() => {}, [dependencies]);
// 记住函数引用

memo(Component);
// props 不变时，允许跳过组件渲染
```

### 2. 选择对照表

| 需求 | 优先考虑 |
| --- | --- |
| 缓存耗时计算的结果 | `useMemo` |
| 稳定对象或数组引用 | `useMemo` |
| 稳定函数引用 | `useCallback` |
| props 不变时跳过子组件渲染 | `React.memo` |
| 保存数据并触发页面更新 | `useState` |
| 跨渲染保存数据但不触发更新 | `useRef` |
| 执行请求、订阅、定时器等副作用 | `useEffect` |

### 3. 最重要的三个原则

1. 正确性优先于性能优化，不要故意漏写依赖。
2. 没有确认性能问题之前，不要到处添加 memo。
3. `useMemo`、`useCallback` 和 `React.memo` 解决的是不同问题。

---

## 参考资料

- [React 官方文档：useMemo](https://react.dev/reference/react/useMemo)
- [React 官方文档：useCallback](https://react.dev/reference/react/useCallback)
- [React 官方文档：memo](https://react.dev/reference/react/memo)
- [React 官方文档：React Compiler 简介](https://react.dev/learn/react-compiler/introduction)

