# Что происходит при `npm start`

## Уровень 1: Самое простое объяснение

Когда ты пишешь `npm start`, ты говоришь компьютеру: **"Запусти моё приложение!"**

Но за этой простой командой скрывается целая цепочка событий. Давай разберём её пошагово.

---

## Уровень 2: Пошаговый разбор

### Шаг 1: npm читает package.json

```bash
$ npm start
```

**Что происходит:**
1. npm ищет файл `package.json` в текущей папке
2. Находит секцию `"scripts"`
3. Ищет скрипт с именем `"start"`

**Пример package.json:**
```json
{
  "name": "my-app",
  "scripts": {
    "start": "node server.js",
    "dev": "vite",
    "build": "tsc && vite build"
  }
}
```

npm видит: `"start": "node server.js"` и запускает эту команду.

---

### Шаг 2: Запуск команды

В зависимости от того, что написано в скрипте, происходит разное.

#### Вариант A: Простой Node.js сервер

```json
"start": "node server.js"
```

**Что происходит:**
```
1. Запускается Node.js
   ↓
2. Node.js загружает V8 engine
   ↓
3. V8 читает server.js
   ↓
4. Парсинг кода → AST
   ↓
5. Создание bytecode
   ↓
6. Выполнение кода
   ↓
7. Сервер запускается и слушает порт
```

#### Вариант B: Development сервер (Vite, Webpack)

```json
"start": "vite"
```

**Что происходит:**
```
1. Запускается Vite
   ↓
2. Vite читает конфигурацию (vite.config.js)
   ↓
3. Сканирует исходники (src/)
   ↓
4. Запускает dev server (обычно на порту 5173)
   ↓
5. Открывает браузер
   ↓
6. Следит за изменениями файлов (hot reload)
```

#### Вариант C: Production сервер (Next.js)

```json
"start": "next start"
```

**Что происходит:**
```
1. Next.js ищет папку .next (результат build)
   ↓
2. Запускает production сервер
   ↓
3. Загружает скомпилированные страницы
   ↓
4. Слушает порт (обычно 3000)
   ↓
5. Готов обрабатывать запросы
```

---

## Уровень 3: Детальный разбор (на примере Vite + React)

### Исходная структура проекта

```
my-app/
├── package.json
├── vite.config.js
├── index.html
└── src/
    ├── main.tsx
    └── App.tsx
```

### package.json

```json
{
  "name": "my-app",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "start": "vite"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "typescript": "^5.0.0"
  }
}
```

### Команда: `npm start`

```bash
$ npm start
```

---

### Этап 1: npm запускает Vite

```
Terminal:
$ npm start

> my-app@1.0.0 start
> vite

  VITE v5.0.0  ready in 234 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
```

**Что произошло:**
1. npm прочитал `package.json`
2. Нашёл `"start": "vite"`
3. Запустил команду `vite`

---

### Этап 2: Vite инициализируется

**Что делает Vite:**

```
1. Читает vite.config.js
   ↓
2. Определяет корневую папку (root)
   ↓
3. Находит index.html (точка входа)
   ↓
4. Сканирует зависимости (node_modules)
   ↓
5. Создаёт HTTP-сервер
   ↓
6. Запускает WebSocket (для hot reload)
```

**vite.config.js:**
```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    open: true  // Автоматически открыть браузер
  }
})
```

---

### Этап 3: Браузер открывается

Vite автоматически открывает браузер на `http://localhost:5173/`

**Что происходит в браузере:**

```
1. Браузер запрашивает http://localhost:5173/
   ↓
2. Vite отдаёт index.html
   ↓
3. Браузер парсит HTML
   ↓
4. Находит <script type="module" src="/src/main.tsx">
   ↓
5. Запрашивает /src/main.tsx
```

---

### Этап 4: Vite обрабатывает запросы

**Запрос 1: index.html**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

**Запрос 2: /src/main.tsx**

**Исходный код:**
```typescript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <App />
)
```

**Что делает Vite:**
```
1. Читает main.tsx
   ↓
2. Превращает TypeScript → JavaScript (с помощью esbuild)
   ↓
3. Превращает JSX → JavaScript
   ↓
4. Разрешает импорты (import React from 'react')
   ↓
5. Отдаёт браузеру готовый JavaScript
```

**Результат (упрощённо):**
```javascript
import React from '/node_modules/react/index.js'
import ReactDOM from '/node_modules/react-dom/client.js'
import App from '/src/App.tsx'

ReactDOM.createRoot(document.getElementById('root')).render(
  React.createElement(App)
)
```

---

### Этап 5: Браузер выполняет код

```
1. Загружает React из node_modules
   ↓
2. Загружает ReactDOM
   ↓
3. Загружает App.tsx (и превращает в JS)
   ↓
4. Выполняет main.tsx:
   - Находит элемент #root
   - Создаёт React root
   - Рендерит <App />
   ↓
5. React создаёт виртуальный DOM
   ↓
6. Обновляет реальный DOM
   ↓
7. Пользователь видит интерфейс! 🎉
```

---

### Этап 6: Hot Module Replacement (HMR)

Vite следит за изменениями в файлах.

**Ты редактируешь App.tsx:**
```typescript
export default function App() {
  return <h1>Hello, World!</h1>  // Изменил текст
}
```

**Что происходит:**
```
1. Vite замечает изменение в App.tsx
   ↓
2. Превращает TypeScript → JavaScript
   ↓
3. Отправляет обновление в браузер через WebSocket
   ↓
4. Браузер получает новый код
   ↓
5. React заменяет компонент БЕЗ перезагрузки страницы
   ↓
6. Ты видишь изменения мгновенно! ⚡
```

---

## Уровень 4: Что происходит на разных уровнях

### 1. Операционная система

```
$ npm start
   ↓
OS запускает процесс npm
   ↓
npm запускает дочерний процесс (vite)
   ↓
Vite запускает HTTP-сервер (слушает порт 5173)
```

### 2. Node.js / V8

```
Node.js загружает код Vite
   ↓
V8 парсит JavaScript код Vite
   ↓
V8 создаёт bytecode
   ↓
V8 выполняет код (JIT-компиляция для горячих участков)
   ↓
Vite работает как HTTP-сервер
```

### 3. Сеть (HTTP)

```
Браузер → GET http://localhost:5173/
   ↓
Vite → 200 OK + index.html
   ↓
Браузер → GET http://localhost:5173/src/main.tsx
   ↓
Vite → 200 OK + JavaScript (превращённый из TypeScript)
   ↓
Браузер → GET http://localhost:5173/node_modules/react/...
   ↓
Vite → 200 OK + React код
```

### 4. Браузер

```
Получает HTML
   ↓
Парсит HTML → DOM дерево
   ↓
Находит <script type="module">
   ↓
Загружает JavaScript модули
   ↓
V8 (в браузере) выполняет JavaScript
   ↓
React создаёт виртуальный DOM
   ↓
React обновляет реальный DOM
   ↓
Пользователь видит интерфейс
```

---

## Сравнение: Development vs Production

### Development (`npm start` или `npm run dev`)

```json
"start": "vite"
```

**Особенности:**
- ✅ Быстрый старт (не нужна полная сборка)
- ✅ Hot Module Replacement (мгновенные обновления)
- ✅ Source maps (легко дебажить)
- ❌ Медленнее выполнение (код не оптимизирован)
- ❌ Много файлов (каждый модуль отдельно)

**Процесс:**
```
Запуск → Dev server → Браузер запрашивает файлы → 
Vite превращает на лету → Отдаёт браузеру
```

### Production (`npm run build` + `npm start`)

```json
"build": "vite build",
"start": "vite preview"
```

**Особенности:**
- ✅ Быстрое выполнение (код оптимизирован)
- ✅ Маленький размер (минификация, tree-shaking)
- ✅ Один или несколько bundle файлов
- ❌ Долгая сборка (нужно собрать весь проект)
- ❌ Нет HMR (нужна пересборка для изменений)

**Процесс:**
```
npm run build:
  Vite читает все файлы →
  TypeScript → JavaScript →
  Babel транспилирует →
  Rollup собирает bundle →
  Минификация →
  Оптимизация →
  Создаёт dist/ папку

npm start:
  Запускает простой HTTP-сервер →
  Отдаёт готовые файлы из dist/
```

---

## Практический пример: полный цикл

### 1. Создание проекта

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
```

### 2. Запуск dev сервера

```bash
npm start
```

**Что происходит:**
```
1. npm читает package.json
2. Запускает "vite"
3. Vite:
   - Читает vite.config.js
   - Запускает dev server на :5173
   - Открывает браузер
4. Браузер:
   - Запрашивает index.html
   - Загружает модули
   - React рендерит интерфейс
5. Ты видишь приложение!
```

### 3. Редактирование кода

**Ты меняешь App.tsx:**
```typescript
export default function App() {
  return <h1>Hello, Tamerlan!</h1>
}
```

**Что происходит:**
```
1. Vite замечает изменение
2. Превращает TypeScript → JavaScript
3. Отправляет в браузер через WebSocket
4. React обновляет компонент
5. Ты видишь изменения БЕЗ перезагрузки!
```

### 4. Сборка для продакшена

```bash
npm run build
```

**Что происходит:**
```
1. TypeScript проверяет типы (tsc)
2. Vite собирает проект:
   - Читает все файлы
   - Превращает TypeScript → JavaScript
   - Превращает JSX → JavaScript
   - Собирает в bundle
   - Минифицирует
   - Оптимизирует
3. Создаёт dist/ папку с готовыми файлами
```

### 5. Запуск production сервера

```bash
npm run preview
```

**Что происходит:**
```
1. Vite запускает простой HTTP-сервер
2. Отдаёт файлы из dist/
3. Браузер загружает оптимизированный код
4. Приложение работает быстро!
```

---

## Главные выводы

### `npm start` делает:
1. **Читает** package.json
2. **Запускает** команду из скрипта "start"
3. **Инициализирует** dev server (Vite, Webpack, и т.д.)
4. **Обрабатывает** файлы на лету (TypeScript → JavaScript)
5. **Отдаёт** код браузеру
6. **Следит** за изменениями (HMR)

### Разница между dev и production:
- **Dev:** Быстрый старт, медленное выполнение, HMR
- **Production:** Долгая сборка, быстрое выполнение, оптимизация

### Где происходит магия:
- **Build-time:** TypeScript → JavaScript, оптимизация (в production)
- **Run-time:** Выполнение кода, рендеринг, взаимодействие с пользователем

**Теперь ты знаешь, что происходит под капотом!** 🚀
