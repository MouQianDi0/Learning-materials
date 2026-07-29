# React 自定义 Hook 完整指南

> 自定义 Hook 的本质：把一段可以复用的 React 状态逻辑，封装成一个以 `use` 开头的函数。

## 目录

- [一、什么是自定义 Hook](#一什么是自定义-hook)
- [二、为什么要写自定义 Hook](#二为什么要写自定义-hook)
- [三、基本结构](#三基本结构)
- [四、编写自定义 Hook 的标准步骤](#四编写自定义-hook-的标准步骤)
- [五、Hook 一般接收什么、返回什么](#五hook-一般接收什么返回什么)
- [六、三种常见返回方式](#六三种常见返回方式)
- [七、示例一：useToggle](#七示例一usetoggle)
- [八、示例二：useWindowSize](#八示例二usewindowsize)
- [九、示例三：useDebounce](#九示例三usedebounce)
- [十、示例四：useNotes](#十示例四usenotes)
- [十一、自定义 Hook 和普通函数的区别](#十一自定义-hook-和普通函数的区别)
- [十二、自定义 Hook 必须遵守的规则](#十二自定义-hook-必须遵守的规则)
- [十三、自定义 Hook 不等于共享状态](#十三自定义-hook-不等于共享状态)
- [十四、什么时候应该抽成 Hook](#十四什么时候应该抽成-hook)
- [十五、推荐的项目目录结构](#十五推荐的项目目录结构)
- [十六、通用 Hook 模板](#十六通用-hook-模板)
- [十七、常见错误](#十七常见错误)
- [十八、固定思考流程](#十八固定思考流程)
- [十九、练习顺序](#十九练习顺序)

---

## 一、什么是自定义 Hook

自定义 Hook 本质上仍然是一个普通的 JavaScript 或 TypeScript 函数。

它和普通函数最大的区别是：

- 函数名必须以 `use` 开头；
- 函数内部可以调用 React 自带的 Hook；
- 函数内部也可以调用其他自定义 Hook；
- 它通常用于封装状态、生命周期、副作用和业务逻辑。

例如：

```tsx
import { useState } from "react";

export function useCounter() {
  const [count, setCount] = useState(0);

  function increase() {
    setCount((previousCount) => previousCount + 1);
  }

  return {
    count,
    increase,
  };
}
```

组件中使用：

```tsx
import { useCounter } from "./hooks/useCounter";

export default function App() {
  const { count, increase } = useCounter();

  return (
    <div>
      <p>当前数量：{count}</p>
      <button onClick={increase}>增加</button>
    </div>
  );
}
```

可以把它理解为：

```text
组件
  ↓ 调用
自定义 Hook
  ↓ 管理
状态、生命周期、副作用、业务逻辑
```

---

## 二、为什么要写自定义 Hook

假设多个组件都要实现下面这些功能：

- 管理弹窗打开和关闭；
- 监听浏览器窗口大小；
- 搜索输入防抖；
- 获取用户信息；
- 请求笔记列表；
- 管理登录状态；
- 读取和保存本地存储。

如果每个组件都重新写一次，会出现大量重复代码。

自定义 Hook 可以把这些逻辑封装起来，让组件主要负责页面展示。

```text
页面组件：负责 UI 和交互展示
自定义 Hook：负责状态和业务逻辑
Service：负责发送请求
Type：负责类型定义
```

这样做的优点包括：

1. 减少重复代码；
2. 让组件更容易阅读；
3. 让业务逻辑更容易测试；
4. 让逻辑可以在多个组件中重复使用；
5. 让项目职责划分更加清晰。

---

## 三、基本结构

一个常见的自定义 Hook 结构如下：

```tsx
import { useState } from "react";

export function useSomething() {
  const [value, setValue] = useState("");

  function changeValue(newValue: string) {
    setValue(newValue);
  }

  return {
    value,
    changeValue,
  };
}
```

组件中使用：

```tsx
function App() {
  const { value, changeValue } = useSomething();

  return (
    <div>
      <p>{value}</p>
      <button onClick={() => changeValue("Hello")}>修改</button>
    </div>
  );
}
```

自定义 Hook 通常由下面几个部分组成：

```text
参数
状态
副作用
操作方法
返回值
```

---

## 四、编写自定义 Hook 的标准步骤

### 第一步：找出组件中的可复用逻辑

例如下面这个组件：

```tsx
import { useState } from "react";

function App() {
  const [isOpen, setIsOpen] = useState(false);

  function open() {
    setIsOpen(true);
  }

  function close() {
    setIsOpen(false);
  }

  function toggle() {
    setIsOpen((previousValue) => !previousValue);
  }

  return (
    <div>
      <p>{isOpen ? "已经打开" : "已经关闭"}</p>
      <button onClick={open}>打开</button>
      <button onClick={close}>关闭</button>
      <button onClick={toggle}>切换</button>
    </div>
  );
}
```

打开、关闭、切换这套逻辑，可以用于：

- 弹窗；
- 抽屉；
- 下拉菜单；
- 侧边栏；
- 密码显示和隐藏；
- 折叠面板。

因此它很适合抽成一个 Hook。

### 第二步：创建以 `use` 开头的函数

```tsx
function useToggle() {
  // Hook 内部逻辑
}
```

常见命名：

```text
useToggle
useFetch
useWindowSize
useDebounce
useAuth
useNotes
useCategories
useCurrentUser
```

不要写成：

```tsx
function toggleHook() {}
function getToggle() {}
```

React 和 ESLint 会通过 `use` 前缀识别 Hook。

### 第三步：把状态放进 Hook

```tsx
import { useState } from "react";

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
}
```

### 第四步：封装操作方法

```tsx
function open() {
  setValue(true);
}

function close() {
  setValue(false);
}

function toggle() {
  setValue((previousValue) => !previousValue);
}
```

### 第五步：返回组件需要的内容

```tsx
return {
  value,
  open,
  close,
  toggle,
};
```

### 第六步：在组件中调用

```tsx
function App() {
  const {
    value: isOpen,
    open,
    close,
    toggle,
  } = useToggle(false);

  return (
    <div>
      <p>{isOpen ? "已经打开" : "已经关闭"}</p>
      <button onClick={open}>打开</button>
      <button onClick={close}>关闭</button>
      <button onClick={toggle}>切换</button>
    </div>
  );
}
```

---

## 五、Hook 一般接收什么、返回什么

可以把自定义 Hook 看成一个加工厂：

```text
传入参数
   ↓
Hook 内部处理
   ↓
返回状态和方法
```

### 常见输入

Hook 可以接收：

- 初始值；
- 配置对象；
- API 地址；
- 查询参数；
- 用户 ID；
- 延迟时间；
- 回调函数；
- 是否启用。

例如：

```tsx
useToggle(false);
useDebounce(keyword, 500);
useFetch("/api/notes");
useUser(userId);
```

### 常见输出

Hook 通常返回：

- 当前数据；
- 加载状态；
- 错误信息；
- 操作方法；
- 刷新方法；
- 重置方法。

例如：

```tsx
return {
  data,
  loading,
  error,
  refresh,
  reset,
};
```

---

## 六、三种常见返回方式

### 方式一：返回对象

```tsx
return {
  value,
  toggle,
  open,
  close,
};
```

使用：

```tsx
const { value, toggle } = useToggle();
```

优点：

- 字段含义明确；
- 不需要记住顺序；
- 适合返回内容较多的 Hook。

### 方式二：返回数组

```tsx
return [value, toggle] as const;
```

使用：

```tsx
const [isOpen, toggle] = useToggle();
```

优点是调用时可以自由命名：

```tsx
const [isModalOpen, toggleModal] = useToggle();
const [isMenuOpen, toggleMenu] = useToggle();
```

缺点是返回值太多时不容易记住每个位置的含义。

一般建议：

- 返回两三个内容时，可以考虑数组；
- 返回较多内容时，更推荐对象。

### 方式三：只返回一个值或函数

返回一个值：

```tsx
function useIsMobile() {
  return true;
}
```

返回一个函数：

```tsx
function useLogout() {
  return function logout() {
    localStorage.removeItem("token");
  };
}
```

使用：

```tsx
const isMobile = useIsMobile();
const logout = useLogout();
```

---

## 七、示例一：useToggle

### Hook 文件

```tsx
import { useState } from "react";

export function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  function open() {
    setValue(true);
  }

  function close() {
    setValue(false);
  }

  function toggle() {
    setValue((previousValue) => !previousValue);
  }

  return {
    value,
    open,
    close,
    toggle,
  };
}
```

### 组件使用

```tsx
import { useToggle } from "./hooks/useToggle";

export default function App() {
  const {
    value: isOpen,
    open,
    close,
    toggle,
  } = useToggle(false);

  return (
    <div>
      <p>{isOpen ? "弹窗已打开" : "弹窗已关闭"}</p>

      <button onClick={open}>打开</button>
      <button onClick={close}>关闭</button>
      <button onClick={toggle}>切换</button>
    </div>
  );
}
```

---

## 八、示例二：useWindowSize

监听浏览器窗口大小非常适合封装成 Hook。

### `useWindowSize.ts`

```tsx
import { useEffect, useState } from "react";

interface WindowSize {
  width: number;
  height: number;
}

export function useWindowSize(): WindowSize {
  const [windowSize, setWindowSize] = useState<WindowSize>({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return windowSize;
}
```

### 组件使用

```tsx
import { useWindowSize } from "./hooks/useWindowSize";

function App() {
  const { width, height } = useWindowSize();

  const isMobile = width < 768;

  return (
    <div>
      <p>宽度：{width}px</p>
      <p>高度：{height}px</p>
      <p>设备类型：{isMobile ? "移动端" : "桌面端"}</p>
    </div>
  );
}
```

### 为什么要清理监听

```tsx
return () => {
  window.removeEventListener("resize", handleResize);
};
```

组件卸载时，如果不移除监听，可能导致：

- 重复监听；
- 不必要的内存占用；
- 逻辑被多次触发；
- 潜在内存泄漏。

---

## 九、示例三：useDebounce

搜索框中，用户每输入一个字符就请求接口，会产生很多请求：

```text
n
no
not
note
notes
```

可以使用防抖 Hook，让用户停止输入一段时间后再处理。

### `useDebounce.ts`

```tsx
import { useEffect, useState } from "react";

export function useDebounce<T>(value: T, delay = 500): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = window.setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      window.clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

这里的泛型 `T` 表示该 Hook 可以处理多种数据类型：

```tsx
useDebounce<string>(keyword, 500);
useDebounce<number>(page, 500);
```

TypeScript 一般可以自动推断类型，所以通常直接写：

```tsx
const debouncedKeyword = useDebounce(keyword, 500);
```

### 组件使用

```tsx
import { useEffect, useState } from "react";
import { useDebounce } from "./hooks/useDebounce";

function SearchNotes() {
  const [keyword, setKeyword] = useState("");

  const debouncedKeyword = useDebounce(keyword, 500);

  useEffect(() => {
    if (!debouncedKeyword.trim()) {
      return;
    }

    console.log("请求搜索接口：", debouncedKeyword);

    // fetch(`/api/notes?keyword=${debouncedKeyword}`);
  }, [debouncedKeyword]);

  return (
    <input
      value={keyword}
      onChange={(event) => setKeyword(event.target.value)}
      placeholder="搜索笔记"
    />
  );
}
```

执行过程：

```text
用户输入 keyword
       ↓
useDebounce 等待 500ms
       ↓
500ms 内继续输入，则清除旧定时器并重新计时
       ↓
停止输入 500ms
       ↓
更新 debouncedKeyword
       ↓
触发搜索请求
```

---

## 十、示例四：useNotes

下面以笔记应用为例，封装一个获取笔记列表的 Hook。

### 类型定义

```tsx
interface Note {
  id: number;
  title: string;
  content: string;
}
```

### `useNotes.ts`

```tsx
import { useCallback, useEffect, useState } from "react";

interface Note {
  id: number;
  title: string;
  content: string;
}

export function useNotes() {
  const [notes, setNotes] = useState<Note[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchNotes = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);

      const response = await fetch("/api/notes");

      if (!response.ok) {
        throw new Error(`请求失败：${response.status}`);
      }

      const data: Note[] = await response.json();
      setNotes(data);
    } catch (error) {
      const message =
        error instanceof Error
          ? error.message
          : "获取笔记失败";

      setError(message);
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchNotes();
  }, [fetchNotes]);

  return {
    notes,
    loading,
    error,
    refresh: fetchNotes,
  };
}
```

### 页面使用

```tsx
import { useNotes } from "./hooks/useNotes";

function NotesPage() {
  const {
    notes,
    loading,
    error,
    refresh,
  } = useNotes();

  if (loading) {
    return <p>加载中...</p>;
  }

  if (error) {
    return (
      <div>
        <p>加载失败：{error}</p>
        <button onClick={refresh}>重新加载</button>
      </div>
    );
  }

  return (
    <div>
      <button onClick={refresh}>刷新</button>

      {notes.map((note) => (
        <article key={note.id}>
          <h2>{note.title}</h2>
          <p>{note.content}</p>
        </article>
      ))}
    </div>
  );
}
```

现在页面主要负责展示，Hook 负责：

- 保存笔记数据；
- 保存加载状态；
- 保存错误状态；
- 请求笔记列表；
- 提供刷新方法。

---

## 十一、自定义 Hook 和普通函数的区别

### 普通工具函数

```tsx
function formatDate(date: Date) {
  return date.toLocaleDateString();
}
```

它只是接收数据并返回计算结果，没有 React 状态和生命周期。

```tsx
const dateText = formatDate(new Date());
```

### 自定义 Hook

```tsx
function useToggle() {
  const [value, setValue] = useState(false);

  return {
    value,
    toggle: () => {
      setValue((previousValue) => !previousValue);
    },
  };
}
```

它使用了 React Hook，拥有状态能力。

### 判断方式

只做普通计算时，写普通函数：

```text
formatDate
calculatePrice
getFullName
validateEmail
```

需要 React 状态、生命周期或副作用时，写 Hook：

```text
useToggle
useWindowSize
useFetch
useAuth
useNotes
```

不要把所有函数都写成 Hook。

---

## 十二、自定义 Hook 必须遵守的规则

### 规则一：名字必须以 `use` 开头

正确：

```tsx
function useNotes() {}
```

错误：

```tsx
function getNotesHook() {}
```

### 规则二：只能在组件或其他 Hook 中调用

正确：

```tsx
function App() {
  const notes = useNotes();
}
```

正确：

```tsx
function useSearchNotes() {
  const notes = useNotes();
}
```

不推荐在普通函数中调用：

```tsx
function normalFunction() {
  const notes = useNotes();
}
```

### 规则三：不要在条件判断中调用 Hook

错误：

```tsx
function App({ isLogin }: { isLogin: boolean }) {
  if (isLogin) {
    const notes = useNotes();
  }

  return <div />;
}
```

正确：

```tsx
function App({ isLogin }: { isLogin: boolean }) {
  const notes = useNotes();

  if (!isLogin) {
    return <p>请先登录</p>;
  }

  return <div />;
}
```

React 依靠固定的调用顺序识别 Hook，所以每次渲染时 Hook 的调用顺序必须一致。

### 规则四：不要在循环中调用 Hook

错误：

```tsx
notes.map((note) => {
  const result = useSomething(note);
  return result;
});
```

Hook 应该在组件顶层调用。

### 规则五：副作用要按需清理

例如定时器：

```tsx
useEffect(() => {
  const timer = window.setInterval(() => {
    console.log("执行");
  }, 1000);

  return () => {
    window.clearInterval(timer);
  };
}, []);
```

常见需要清理的内容：

- `setInterval`；
- `setTimeout`；
- `addEventListener`；
- WebSocket；
- SSE；
- 第三方订阅；
- 视频流监听；
- 第三方库实例。

---

## 十三、自定义 Hook 不等于共享状态

这一点非常重要。

```tsx
function ComponentA() {
  const { value } = useToggle();
}

function ComponentB() {
  const { value } = useToggle();
}
```

`ComponentA` 和 `ComponentB` 会分别创建自己的状态。

```text
ComponentA → 一份独立状态
ComponentB → 另一份独立状态
```

自定义 Hook 共享的是：

> 状态逻辑，而不是状态本身。

要让多个组件共享同一份状态，可以考虑：

- 把状态提升到共同父组件；
- Context；
- Zustand；
- Redux；
- Jotai；
- 其他全局状态管理方案。

---

## 十四、什么时候应该抽成 Hook

### 情况一：逻辑在多个组件中重复

例如：

```text
useWindowSize
useDebounce
useToggle
useLocalStorage
```

### 情况二：组件中的业务逻辑太多

例如一个笔记页面同时包含：

- 获取笔记；
- 搜索笔记；
- 删除笔记；
- 分页加载；
- 错误处理；
- Loading 状态。

可以拆成：

```text
useNotes
useNoteSearch
useDeleteNote
useNotePagination
```

### 情况三：逻辑有明确的业务含义

例如：

```text
useAuth
useCurrentUser
useNotes
useCategories
useUploadAvatar
```

这样的命名，比在页面中堆积大量 `useEffect` 更容易阅读。

### 不一定要抽成 Hook 的情况

- 逻辑只使用一次；
- 逻辑非常简单；
- 抽离后反而更难理解；
- 只是一个普通计算函数；
- Hook 职责不明确。

不要为了“看起来高级”而过度封装。

---

## 十五、推荐的项目目录结构

基础项目可以这样组织：

```text
src/
├── components/
├── hooks/
│   ├── useToggle.ts
│   ├── useWindowSize.ts
│   ├── useDebounce.ts
│   └── useNotes.ts
├── services/
│   └── notes.service.ts
├── types/
│   └── note.types.ts
├── pages/
│   └── NotesPage.tsx
└── App.tsx
```

更推荐把请求代码放到 Service 中。

### `notes.service.ts`

```tsx
import type { Note } from "../types/note.types";

export async function getNotes(): Promise<Note[]> {
  const response = await fetch("/api/notes");

  if (!response.ok) {
    throw new Error(`请求失败：${response.status}`);
  }

  return response.json();
}
```

### Hook 调用 Service

```tsx
import { useCallback, useEffect, useState } from "react";
import { getNotes } from "../services/notes.service";
import type { Note } from "../types/note.types";

export function useNotes() {
  const [notes, setNotes] = useState<Note[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchNotes = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);

      const data = await getNotes();
      setNotes(data);
    } catch (error) {
      setError(
        error instanceof Error
          ? error.message
          : "获取笔记失败",
      );
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchNotes();
  }, [fetchNotes]);

  return {
    notes,
    loading,
    error,
    refresh: fetchNotes,
  };
}
```

职责划分：

```text
NotesPage.tsx
负责页面展示

useNotes.ts
负责状态和业务流程

notes.service.ts
负责发送 HTTP 请求

note.types.ts
负责 TypeScript 类型定义
```

---

## 十六、通用 Hook 模板

```tsx
import { useEffect, useState } from "react";

interface UseExampleOptions {
  initialValue?: string;
  enabled?: boolean;
}

interface UseExampleResult {
  value: string;
  loading: boolean;
  error: string | null;
  updateValue: (newValue: string) => void;
  reset: () => void;
}

export function useExample(
  options: UseExampleOptions = {},
): UseExampleResult {
  const {
    initialValue = "",
    enabled = true,
  } = options;

  const [value, setValue] = useState(initialValue);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    if (!enabled) {
      return;
    }

    // 执行初始化逻辑
  }, [enabled]);

  function updateValue(newValue: string) {
    setValue(newValue);
  }

  function reset() {
    setValue(initialValue);
    setError(null);
    setLoading(false);
  }

  return {
    value,
    loading,
    error,
    updateValue,
    reset,
  };
}
```

这个模板包含：

- 配置参数；
- 默认值；
- 状态；
- 副作用；
- 操作函数；
- 返回类型。

---

## 十七、常见错误

### 错误一：把 Hook 写成普通函数名

```tsx
function getUserData() {
  const [user, setUser] = useState(null);
}
```

应该写成：

```tsx
function useUserData() {
  const [user, setUser] = useState(null);
}
```

### 错误二：在事件函数里调用 Hook

```tsx
function handleClick() {
  const data = useNotes();
}
```

Hook 必须在组件顶层调用。

### 错误三：忘记返回组件需要的内容

```tsx
function useToggle() {
  const [value, setValue] = useState(false);
}
```

组件无法使用内部状态。

应该返回：

```tsx
return {
  value,
  setValue,
};
```

### 错误四：一个 Hook 负责太多事情

不推荐：

```text
useEverything
```

它同时处理：

- 登录；
- 笔记；
- 分类；
- 上传；
- 主题；
- 窗口监听。

更推荐按照职责拆分：

```text
useAuth
useNotes
useCategories
useUploadAvatar
useTheme
useWindowSize
```

### 错误五：没有清理副作用

```tsx
useEffect(() => {
  window.addEventListener("resize", handleResize);
}, []);
```

应该清理：

```tsx
useEffect(() => {
  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

### 错误六：为了复用 UI 而写 Hook

Hook 主要复用逻辑，不直接复用 JSX 页面结构。

需要复用 UI 时，更适合写组件。

```text
复用 UI → 组件
复用状态逻辑 → Hook
普通计算 → 工具函数
```

---

## 十八、固定思考流程

写自定义 Hook 时，可以依次问自己：

1. 我要封装的是什么逻辑？
2. 这个逻辑是否会重复使用？
3. Hook 需要接收什么参数？
4. 内部需要哪些状态？
5. 需要调用哪些 React Hook？
6. 是否存在副作用？
7. 副作用是否需要清理？
8. 组件需要拿到哪些数据和方法？
9. 返回对象还是数组？
10. 这个 Hook 的职责是否足够单一？

完整流程：

```text
确定职责
  ↓
定义参数
  ↓
创建状态
  ↓
处理副作用
  ↓
封装操作方法
  ↓
返回数据和方法
  ↓
在组件中调用
```

---

## 十九、练习顺序

建议按照下面的顺序练习：

### 初级

```text
useToggle
useCounter
useInput
```

主要练习：

- `useState`；
- 参数；
- 返回值；
- 对象和数组返回方式。

### 中级

```text
useWindowSize
useDebounce
useLocalStorage
```

主要练习：

- `useEffect`；
- 依赖数组；
- 清理函数；
- 浏览器 API。

### 进阶

```text
useFetch
useNotes
useAuth
usePagination
```

主要练习：

- 异步请求；
- Loading 状态；
- 错误处理；
- `useCallback`；
- 业务逻辑封装；
- Hook、Service 和 Type 的职责划分。

---

## 总结

自定义 Hook 可以概括成一句话：

> 把组件中可复用的状态逻辑抽出来，封装成一个以 `use` 开头的函数。

最需要记住的几点：

1. Hook 名字必须以 `use` 开头；
2. Hook 在组件顶层调用；
3. 不要在条件、循环和普通事件函数中调用 Hook；
4. 自定义 Hook 复用的是逻辑，不是同一份状态；
5. 副作用要注意清理；
6. 返回内容少时可以用数组，内容多时更适合用对象；
7. 页面负责 UI，Hook 负责状态和业务逻辑，Service 负责请求。
