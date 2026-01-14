# React: Основные хуки и мемоизация

---

## Основы: useState, useEffect и useRef

Прежде чем говорить о мемоизации, важно понять базовые хуки React.

---

## useState — хранение состояния

### Что это?

Хук для хранения данных, которые влияют на отображение компонента. При изменении состояния React **автоматически** перерисовывает компонент.

### Синтаксис

```tsx
const [value, setValue] = useState(initialValue)
```

### Как работает?

```tsx
function Counter() {
  const [count, setCount] = useState(0)
  
  // При вызове setCount:
  // 1. React сохраняет новое значение
  // 2. React перерисовывает компонент
  // 3. UI обновляется автоматически
  
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  )
}
```

### Главное правило

> **`useState`** = "что показать на экране"
> 
> Изменение state → React сам обновит UI. Никаких дополнительных действий не нужно.

---

## useEffect — побочные эффекты

### Что это?

Хук для выполнения **побочных эффектов** — действий, которые выходят за рамки "чистого" рендеринга: запросы к API, таймеры, подписки на события, работа с localStorage.

### Ключевая концепция: React vs Внешний мир

```
┌─────────────────────────────────────────────────────┐
│                   Территория React                  │
│                                                     │
│   JSX → Virtual DOM → Real DOM                      │
│   (React сам управляет всем)                        │
│                                                     │
│   return <p>{count}</p>  ← React отрисует сам       │
│                                                     │
└─────────────────────────────────────────────────────┘
                         │
                         │ useEffect (дверь наружу)
                         ▼
┌─────────────────────────────────────────────────────┐
│                    Внешний мир                      │
│                                                     │
│   • fetch() / API запросы                           │
│   • setInterval / setTimeout                        │
│   • document.title                                  │
│   • localStorage                                    │
│   • window.addEventListener                         │
│   • WebSocket                                       │
│   • Любой код вне контроля React                    │
└─────────────────────────────────────────────────────┘
```

**Всё что внутри `return (...)` — React обработает сам.**  
**Всё что снаружи React (таймеры, API, браузерные API) — нужен `useEffect`.**

### Синтаксис

```tsx
useEffect(() => {
  // Код эффекта выполняется ПОСЛЕ рендера
  
  return () => {
    // Cleanup функция (опционально)
    // Выполняется перед следующим эффектом или при размонтировании
  }
}, [dependencies]) // Массив зависимостей
```

### Три варианта использования

#### 1️⃣ Без массива зависимостей — выполняется после КАЖДОГО рендера

```tsx
useEffect(() => {
  console.log('Компонент отрендерился')
})
```

⚠️ Используется редко — может вызвать проблемы с производительностью.

#### 2️⃣ Пустой массив `[]` — выполняется ОДИН раз при монтировании

```tsx
useEffect(() => {
  console.log('Компонент смонтирован')
  
  return () => {
    console.log('Компонент размонтирован')
  }
}, [])
```

**Типичное использование:** загрузка данных при открытии страницы, инициализация.

#### 3️⃣ С зависимостями — выполняется когда зависимости меняются

```tsx
useEffect(() => {
  console.log(`userId изменился: ${userId}`)
  fetchUserData(userId)
}, [userId])
```

**Типичное использование:** реакция на изменение конкретных данных.

### Примеры

#### Загрузка данных с API

```tsx
function NewsList() {
  const [news, setNews] = useState([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    async function fetchNews() {
      setLoading(true)
      const response = await fetch('/api/news')
      const data = await response.json()
      setNews(data)
      setLoading(false)
    }
    
    fetchNews()
  }, []) // Пустой массив = загрузить один раз при монтировании
  
  if (loading) return <div>Загрузка...</div>
  return <ul>{news.map(item => <li key={item.id}>{item.title}</li>)}</ul>
}
```

#### Подписка на события (с cleanup)

```tsx
useEffect(() => {
  const handleResize = () => {
    console.log('Окно изменилось:', window.innerWidth)
  }
  
  window.addEventListener('resize', handleResize)
  
  // Cleanup — отписываемся при размонтировании!
  return () => {
    window.removeEventListener('resize', handleResize)
  }
}, [])
```

#### Таймер

```tsx
function Timer() {
  const [seconds, setSeconds] = useState(180)

  useEffect(() => {
    const timer = setInterval(() => {
      setSeconds(s => s - 1)
    }, 1000)
    
    return () => clearInterval(timer) // Очищаем таймер
  }, [])

  return <span>{seconds}</span>
}
```

### Почему нужен useEffect для таймеров и API?

В vanilla JS ты напрямую манипулируешь DOM:

```js
// Vanilla JS — прямое управление
document.querySelector('.seconds').textContent = value
```

В React ты описываешь **что** должно быть, а React решает **как** это отрисовать:

```tsx
// React — декларативное описание
return <span>{seconds}</span>
```

**`useEffect` нужен когда ты берёшь управление в свои руки** и обходишь React:

```tsx
useEffect(() => {
  // Ручное управление — нужен useEffect
  document.title = `Счёт: ${count}`          // Браузерный API
  localStorage.setItem('count', count)        // Storage API
  fetch('/api/save', { body: count })         // Network API
  setInterval(() => { ... }, 1000)            // Timer API
}, [count])
```

### Главное правило

> **`useState`** = что показать на экране
> 
> **`useEffect`** = что сделать снаружи React (после рендера)

### Сводная таблица useState vs useEffect

| Задача | Инструмент |
|--------|------------|
| Хранить данные и показывать их | `useState` (сам обновит UI) |
| Загрузить данные с сервера | `useEffect` + `useState` |
| Запустить таймер | `useEffect` |
| Подписаться на события браузера | `useEffect` |
| Изменить document.title | `useEffect` |
| Работа с localStorage | `useEffect` |

---

## useRef — коробка для хранения

### Что это?

Хук для хранения **мутабельного значения**, которое:
- Сохраняется между рендерами (не сбрасывается)
- НЕ вызывает ре-рендер при изменении

### Простая аналогия

Представь, что `useRef` — это **коробка с наклейкой**. Ты можешь положить в неё что угодно, достать, поменять — и эта коробка всегда остаётся на месте.

```tsx
const myBox = useRef(null)  // Создали пустую коробку

myBox.current = 5           // Положили туда число 5
myBox.current = "привет"    // Заменили на слово "привет"
console.log(myBox.current)  // Достали — получили "привет"
```

### Синтаксис

```tsx
const ref = useRef(initialValue)

ref.current  // Доступ к значению внутри "коробки"
```

### Зачем нужна эта "коробка"?

В React есть проблема: когда что-то меняется на странице, компонент как бы "пересоздаётся". Все обычные переменные при этом сбрасываются:

```tsx
function Counter() {
  let count = 0  // ❌ Сбросится при каждом ре-рендере!
  
  return (
    <button onClick={() => count++}>
      {count}  {/* Всегда будет 0 */}
    </button>
  )
}
```

А `useRef` — это коробка, которая **не сбрасывается**:

```tsx
function Counter() {
  const countRef = useRef(0)  // ✅ Сохраняется между рендерами
  
  // Но! Изменение countRef.current НЕ обновит UI
  // Для UI нужен useState
}
```

---

### Два главных применения useRef

#### 1️⃣ Доступ к DOM-элементам

Самое частое использование — "поймать" элемент на странице:

```tsx
function TextInput() {
  const inputRef = useRef(null)  // Шаг 1: создаём пустую коробку
  
  const focusInput = () => {
    inputRef.current.focus()     // Шаг 3: используем элемент
  }
  
  return (
    <>
      {/* Шаг 2: React положит элемент в коробку */}
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Фокус на поле</button>
    </>
  )
}
```

**Как это работает:**
1. `useRef(null)` — создаём пустую коробку
2. `ref={inputRef}` — говорим React: "положи этот элемент в коробку"
3. `inputRef.current` — достаём элемент и работаем с ним

**Аналогия из жизни:**
- `useRef(null)` — наклеил на шкаф стикер "Шкаф №1"
- `ref={inputRef}` — момент, когда клеишь стикер
- `inputRef.current` — сам шкаф, к которому приклеен стикер
- `inputRef.current.offsetHeight` — меришь высоту шкафа рулеткой

#### 2️⃣ Хранение значений без ре-рендера

Когда нужно запомнить что-то, но НЕ нужно обновлять UI:

```tsx
function ScrollTracker() {
  const lastScrollY = useRef(0)  // Предыдущая позиция скролла
  
  useEffect(() => {
    const onScroll = () => {
      const currentY = window.scrollY
      const delta = currentY - lastScrollY.current  // Разница
      
      lastScrollY.current = currentY  // Запоминаем новую позицию
      // ↑ НЕ вызывает ре-рендер! Просто обновляет значение в коробке
      
      console.log('Скролл:', delta > 0 ? 'вниз' : 'вверх')
    }
    
    window.addEventListener('scroll', onScroll)
    return () => window.removeEventListener('scroll', onScroll)
  }, [])
  
  return <div>Скролль страницу</div>
}
```

---

### useRef vs useState — в чём разница?

| | `useState` | `useRef` |
|---|---|---|
| **При изменении** | Компонент перерисовывается | Ничего не происходит |
| **Для чего** | Данные, которые показываем на экране | Данные "за кулисами" |
| **Синтаксис изменения** | `setValue(newValue)` | `ref.current = newValue` |

**Простое правило:**
- Нужно показать на экране → `useState`
- Нужно просто запомнить → `useRef`

```tsx
function Example() {
  const [count, setCount] = useState(0)    // Показываем на экране
  const renderCount = useRef(0)            // Просто считаем (не показываем)
  
  renderCount.current++  // Считаем рендеры (не вызывает новый рендер)
  
  return (
    <div>
      <p>Счёт: {count}</p>
      <p>Рендеров было: {renderCount.current}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  )
}
```

---

### useRef vs useMemo — в чём разница?

| | `useRef` | `useMemo` |
|---|---|---|
| **Что это** | Коробка для хранения | Калькулятор с кэшем |
| **Кто меняет значение** | Ты сам (`ref.current = ...`) | React автоматически |
| **Когда обновляется** | Когда ты напишешь | Когда изменятся зависимости `[a, b]` |
| **Для чего** | Хранить DOM-элементы, счётчики, флаги | Кэшировать результаты вычислений |

```tsx
// useRef — ты сам кладёшь и достаёшь
const myBox = useRef(0)
myBox.current = 5      // Ты положил
myBox.current = 10     // Ты заменил

// useMemo — React сам вычисляет когда нужно
const result = useMemo(() => {
  return a + b         // React посчитает сам
}, [a, b])             // И пересчитает только когда a или b изменятся
```

---

### Примеры использования

#### Измерение размеров элемента

```tsx
function Header() {
  const headerRef = useRef(null)
  const [height, setHeight] = useState(0)
  
  useEffect(() => {
    // Измеряем высоту header'а
    const h = headerRef.current?.offsetHeight || 0
    setHeight(h)
  }, [])
  
  return (
    <header ref={headerRef}>
      <p>Высота header'а: {height}px</p>
    </header>
  )
}
```

#### Хранение предыдущего значения

```tsx
function Counter() {
  const [count, setCount] = useState(0)
  const prevCount = useRef(0)
  
  useEffect(() => {
    prevCount.current = count  // Запоминаем после каждого рендера
  }, [count])
  
  return (
    <div>
      <p>Сейчас: {count}</p>
      <p>Было: {prevCount.current}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  )
}
```

#### Предотвращение лишних вызовов (throttle/debounce)

```tsx
function ScrollHandler() {
  const ticking = useRef(false)  // Флаг "уже обрабатываем"
  
  useEffect(() => {
    const onScroll = () => {
      if (ticking.current) return  // Если уже обрабатываем — пропускаем
      
      ticking.current = true
      
      requestAnimationFrame(() => {
        // Делаем что-то со скроллом
        console.log('Скролл:', window.scrollY)
        ticking.current = false  // Готовы к следующему
      })
    }
    
    window.addEventListener('scroll', onScroll)
    return () => window.removeEventListener('scroll', onScroll)
  }, [])
  
  return <div>Скролль!</div>
}
```

---

### Главное правило

> **`useState`** = что показать на экране (изменение → ре-рендер)
> 
> **`useRef`** = что запомнить за кулисами (изменение → ничего не происходит)

### Сводная таблица всех базовых хуков

| Хук | Для чего | При изменении |
|-----|----------|---------------|
| `useState` | Данные для отображения | Ре-рендер |
| `useEffect` | Побочные эффекты (API, таймеры) | — |
| `useRef` | Хранение без ре-рендера, доступ к DOM | Ничего |

---





# Мемоизация — React.memo, useMemo, useCallback

## Зачем нужна мемоизация?

При ре-рендере родительского компонента React по умолчанию ре-рендерит **все** дочерние компоненты, даже если их пропсы не изменились. Мемоизация позволяет пропустить лишние ре-рендеры и вычисления.

---

## 1. `React.memo` — мемоизация компонента

### Что это?

Higher-Order Component (HOC), который оборачивает компонент и предотвращает его ре-рендер, если пропсы не изменились.

### Синтаксис

```tsx
const MyComponent = React.memo(({ prop1, prop2 }) => {
  return <div>{prop1} {prop2}</div>
})

// Или с существующим компонентом:
const MemoizedComponent = React.memo(ExistingComponent)
```

### Как работает?

1. При ре-рендере родителя React проверяет пропсы дочернего компонента
2. Сравнивает новые пропсы со старыми (shallow comparison — по ссылке)
3. Если все пропсы те же — пропускает ре-рендер
4. Если хоть один пропс изменился — ре-рендерит

### Пример

```tsx
// ❌ Без memo — ре-рендерится при КАЖДОМ ре-рендере родителя
const NavItem = ({ item }) => {
  console.log('render', item.label) // Вызывается каждый раз
  return <li>{item.label}</li>
}

// ✅ С memo — ре-рендерится только если item изменился
const NavItem = React.memo(({ item }) => {
  console.log('render', item.label) // Вызывается только при изменении item
  return <li>{item.label}</li>
})
```

### Когда использовать?

- Дочерний компонент "тяжёлый" (много элементов, сложные вычисления)
- Компонент часто получает одинаковые пропсы при ре-рендере родителя
- Список с множеством элементов

### ⚠️ Важно

Сравнение происходит **по ссылке** (shallow comparison):
- Примитивы (string, number, boolean) — сравниваются по значению ✓
- Объекты, массивы, функции — сравниваются по ссылке!

```tsx
// ❌ Проблема: объект создаётся заново при каждом рендере
<Child config={{ theme: 'dark' }} />  // memo бесполезен!

// ❌ Проблема: функция создаётся заново при каждом рендере
<Child onClick={() => handleClick()} />  // memo бесполезен!
```

---

## 2. `useMemo` — мемоизация значения

### Что это?

Хук, который кэширует результат вычисления. Пересчитывает только при изменении зависимостей.

### Синтаксис

```tsx
const memoizedValue = useMemo(() => {
  return computeExpensiveValue(a, b)
}, [a, b])  // Зависимости
```

### Как работает?

1. При первом рендере выполняет функцию и сохраняет результат
2. При следующих рендерах проверяет зависимости
3. Если зависимости не изменились — возвращает сохранённый результат
4. Если изменились — пересчитывает и сохраняет новый результат

### Примеры

```tsx
// Кэширование тяжёлого вычисления
const sortedItems = useMemo(() => {
  return items.sort((a, b) => b.price - a.price)
}, [items])

// Кэширование отфильтрованного списка
const activeUsers = useMemo(() => {
  return users.filter(user => user.isActive)
}, [users])

// Кэширование объекта для передачи в дочерний компонент
const config = useMemo(() => ({
  theme: 'dark',
  showIcons: true,
  locale: userLocale
}), [userLocale])
```

### Когда использовать?

- Тяжёлые вычисления (сортировка, фильтрация, агрегация больших массивов)
- Нужен стабильный объект/массив для передачи в `React.memo` компонент
- Значение используется в зависимостях других хуков

### ⚠️ Важно

**`useMemo` НЕ предотвращает ре-рендер компонента, в котором находится!**

```tsx
function Dashboard({ items }) {
  const [count, setCount] = useState(0)
  
  // При setCount компонент Dashboard ре-рендерится
  // Но sortedItems НЕ пересчитывается (items не менялся)
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => b.price - a.price)
  }, [items])
  
  return <div>...</div>
}
```

---

## 3. `useCallback` — мемоизация функции

### Что это?

Хук, который кэширует функцию. Возвращает ту же ссылку на функцию, пока зависимости не изменились.

### Синтаксис

```tsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b)
}, [a, b])  // Зависимости
```

### 🔑 Ключевой факт: useCallback — это синтаксический сахар от useMemo

**`useCallback` под капотом — это просто `useMemo`, который возвращает функцию:**

```tsx
// Эти две записи ПОЛНОСТЬЮ ИДЕНТИЧНЫ:

// useCallback
const handleClick = useCallback(() => {
  console.log('clicked', id)
}, [id])

// useMemo (эквивалент)
const handleClick = useMemo(() => {
  return () => {
    console.log('clicked', id)
  }
}, [id])
```

### Почему это работает?

В JavaScript функция — это **значение** (объект). Поэтому `useMemo` может кэшировать функцию как любое другое значение.

```tsx
useMemo(() => fn, deps)
//       ^^^^^^^^
//       Функция-фабрика возвращает fn
//       useMemo кэширует результат = fn
```

### Почему тогда существует useCallback?

**Синтаксический сахар для удобства:**

```tsx
// Громоздко с useMemo:
const fn = useMemo(() => () => doSomething(), [])
//                       ^^^ вложенная стрелка, некрасиво

// Удобно с useCallback:
const fn = useCallback(() => doSomething(), [])
```

### Примеры

```tsx
// Обработчик для передачи в дочерний компонент
const handleItemClick = useCallback((id: string) => {
  setSelectedId(id)
}, [])

// Функция с зависимостью
const handleSubmit = useCallback(() => {
  submitForm(formData)
}, [formData])

// Для useEffect
const fetchData = useCallback(async () => {
  const result = await api.getData(userId)
  setData(result)
}, [userId])

useEffect(() => {
  fetchData()
}, [fetchData])
```

### Когда использовать?

- Функция передаётся в `React.memo` компонент
- Функция в зависимостях `useEffect` / `useMemo` / другого `useCallback`
- Функция передаётся в список элементов, каждый из которых мемоизирован

---

## Сводная таблица

| Инструмент | Тип | Что делает | При изменении |
|------------|-----|------------|---------------|
| `useState` | Hook | Хранит данные для UI | Ре-рендер |
| `useEffect` | Hook | Побочные эффекты | — |
| `useRef` | Hook | Хранит значение / DOM-элемент | Ничего |
| `useMemo` | Hook | Кэширует вычисление | Пересчёт при изменении deps |
| `useCallback` | Hook | Кэширует функцию | Пересоздание при изменении deps |
| `React.memo` | HOC | Предотвращает ре-рендер | Ре-рендер при изменении props |

---

## Как они работают вместе

```tsx
function Parent() {
  const [count, setCount] = useState(0)
  const [items, setItems] = useState([...])

  // ✅ useCallback — стабильная ссылка на функцию
  const handleItemClick = useCallback((id: string) => {
    console.log('clicked', id)
  }, [])

  // ✅ useMemo — стабильная ссылка на объект
  const config = useMemo(() => ({
    showIcons: true,
    theme: 'dark'
  }), [])

  // ✅ useMemo — кэширование тяжёлого вычисления
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a.name.localeCompare(b.name))
  }, [items])

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      
      {/* ✅ Child НЕ ре-рендерится при изменении count */}
      {/* Потому что items, handleItemClick, config — стабильные ссылки */}
      <Child 
        items={sortedItems} 
        onClick={handleItemClick} 
        config={config} 
      />
    </>
  )
}

// ✅ React.memo — пропускает ре-рендер если пропсы те же
const Child = React.memo(({ items, onClick, config }) => {
  console.log('Child render') // НЕ вызывается при изменении count
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  )
})
```

---

## Комбинации и их эффект

| React.memo | useCallback/useMemo | Результат |
|------------|---------------------|-----------|
| ❌ Нет | ❌ Нет | Ре-рендер всегда (норма для лёгких компонентов) |
| ✅ Есть | ❌ Нет | Ре-рендер всегда — **memo бесполезен!** |
| ❌ Нет | ✅ Есть | Ре-рендер всегда — **callback/memo бесполезен!** |
| ✅ Есть | ✅ Есть | Ре-рендер только при изменении пропсов ✓ |

**Вывод:** `React.memo` и `useCallback`/`useMemo` работают **в паре**. По отдельности — бесполезны для оптимизации ре-рендеров.

---

## Пример: Список с callback

### ❌ Проблема: memo не работает

```tsx
function TodoList() {
  const [todos, setTodos] = useState([...])
  const [filter, setFilter] = useState('all')

  // ❌ Новая функция при каждом рендере
  const handleToggle = (id: string) => {
    setTodos(prev => prev.map(t => 
      t.id === id ? { ...t, done: !t.done } : t
    ))
  }

  return (
    <ul>
      {todos.map(todo => (
        // ❌ TodoItem получает новую ссылку на handleToggle каждый раз
        // React.memo бесполезен!
        <TodoItem 
          key={todo.id} 
          todo={todo} 
          onToggle={() => handleToggle(todo.id)} 
        />
      ))}
    </ul>
  )
}

const TodoItem = React.memo(({ todo, onToggle }) => {
  console.log('render', todo.text) // Вызывается КАЖДЫЙ раз при изменении filter!
  return (
    <li onClick={onToggle}>
      {todo.done ? '✓' : '○'} {todo.text}
    </li>
  )
})
```

### ✅ Решение: useCallback + React.memo

```tsx
function TodoList() {
  const [todos, setTodos] = useState([...])
  const [filter, setFilter] = useState('all')

  // ✅ Стабильная ссылка на функцию
  const handleToggle = useCallback((id: string) => {
    setTodos(prev => prev.map(t => 
      t.id === id ? { ...t, done: !t.done } : t
    ))
  }, [])

  return (
    <ul>
      {todos.map(todo => (
        // ✅ Передаём стабильную функцию + id отдельно
        <TodoItem 
          key={todo.id} 
          todo={todo} 
          onToggle={handleToggle}
        />
      ))}
    </ul>
  )
}

const TodoItem = React.memo(({ todo, onToggle }) => {
  console.log('render', todo.text) // Вызывается только при изменении todo
  return (
    <li onClick={() => onToggle(todo.id)}>
      {todo.done ? '✓' : '○'} {todo.text}
    </li>
  )
})
```

---

## Когда НЕ нужна мемоизация

1. **Компонент лёгкий и быстро рендерится**
   ```tsx
   // Не нужно memo для простых компонентов
   const Label = ({ text }) => <span>{text}</span>
   ```

2. **Список маленький (3-5 элементов)**
   ```tsx
   // 3 элемента — не проблема
   {navLinks.map(link => <NavLink {...link} />)}
   ```

3. **Пропсы меняются при каждом рендере всё равно**
   ```tsx
   // Бесполезно, если data меняется каждый раз
   <Chart data={realtimeData} />
   ```

4. **Преждевременная оптимизация**
   — Сначала измерь с React DevTools Profiler!

---

## Чеклист: когда использовать что

### useRef нужен если:
- [ ] Нужен доступ к DOM-элементу (измерить размер, установить фокус)
- [ ] Нужно хранить значение между рендерами БЕЗ ре-рендера
- [ ] Нужно запомнить предыдущее значение
- [ ] Нужен флаг/счётчик для throttle/debounce

### useCallback нужен если:
- [ ] Функция передаётся в компонент с `React.memo`
- [ ] Функция в массиве зависимостей `useEffect`
- [ ] Функция в массиве зависимостей `useMemo` или другого `useCallback`

### useMemo нужен если:
- [ ] Тяжёлое вычисление (сортировка/фильтрация большого массива)
- [ ] Объект/массив передаётся в компонент с `React.memo`
- [ ] Значение в массиве зависимостей другого хука

### React.memo нужен если:
- [ ] Компонент рендерится часто с теми же пропсами
- [ ] Компонент "тяжёлый" (сложный JSX, много дочерних элементов)
- [ ] Родитель часто ре-рендерится, но пропсы дочернего не меняются

---

## Главное правило

> **Мемоизация — это оптимизация. Оптимизируй только то, что тормозит.**
> 
> Измерь → Найди проблему → Оптимизируй → Измерь снова

Используй React DevTools Profiler для поиска реальных проблем с производительностью.
