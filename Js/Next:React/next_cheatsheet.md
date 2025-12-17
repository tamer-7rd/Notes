
# 🚀 Шпаргалка по Next.js (база для новичка)

---

## 1. Что такое Next.js
**Next.js** — это full-stack фреймворк для веб-приложений, который использует **React** как библиотеку для рендеринга UI.

**Технически**: Next.js написан на Node.js/JavaScript и использует React как **зависимость** (dependency). Когда ты создаёшь Next.js проект, React автоматически устанавливается в `node_modules`. Next.js предоставляет инструменты и инфраструктуру, которые работают **с** React компонентами, но сам фреймворк — это отдельный код, который:
- обрабатывает роутинг,
- запускает серверный рендеринг React компонентов,
- управляет сборкой и оптимизацией,
- предоставляет API для разработчиков.

**React** — это библиотека для построения пользовательских интерфейсов (UI). Она работает на клиенте (в браузере) и позволяет создавать компоненты с состоянием, но сама по себе не предоставляет:
- систему маршрутизации (роутинг),
- серверную инфраструктуру,
- оптимизацию производительности,
- инструменты для SEO.

**Next.js** использует React для рендеринга UI, но добавляет полную инфраструктуру для production-ready веб-приложений:
- **File-based routing** — маршрутизация на основе файловой структуры (без React Router),
- **Server-Side Rendering (SSR)** — рендеринг компонентов на сервере для лучшего SEO и производительности,
- **Static Site Generation (SSG)** — предгенерация статических страниц на этапе сборки,
- **Incremental Static Regeneration (ISR)** — обновление статических страниц по требованию,
- **API Routes** — встроенный бэкенд (серверные эндпоинты прямо в проекте),
- **Автоматическая оптимизация** — изображения (`next/image`), шрифты (`next/font`), кэширование, code splitting,
- **Server Components** — компоненты, выполняющиеся на сервере (по умолчанию в App Router).

👉 **Аналогия**: React = кирпичи для строительства, Next.js = готовый дом с коммуникациями, фундаментом и всеми системами.

---

## 2. Структура проекта (App Router)

### Полная структура с пометками:

```
📁 my-nextjs-app/
│
├── 📁 .next/                    # 🔷 АВТО (генерируется Next.js)
│                                # ❌ НЕ КОММИТИТЬ в git
│
├── 📁 node_modules/             # 🔷 АВТО (зависимости)
│                                # ❌ НЕ КОММИТИТЬ в git
│
├── 📁 public/                   # 🔷 ОБЯЗАТЕЛЬНО (статические файлы)
│   ├── favicon.ico              # ⚪ опционально
│   ├── images/                  # ⚪ опционально
│   └── fonts/                   # ⚪ опционально
│
├── 📁 src/                      # ⚪ ОПЦИОНАЛЬНО (можно без src/)
│   │
│   ├── 📁 app/                  # 🔷 ОБЯЗАТЕЛЬНО (App Router)
│   │   │
│   │   ├── layout.tsx           # 🔷 ОБЯЗАТЕЛЬНО (Root Layout)
│   │   ├── page.tsx             # 🔷 ОБЯЗАТЕЛЬНО (главная /)
│   │   ├── globals.css          # ⚪ опционально (глобальные стили)
│   │   ├── loading.tsx          # ⚪ опционально (Loading UI)
│   │   ├── error.tsx            # ⚪ опционально (Error Boundary)
│   │   ├── not-found.tsx        # ⚪ опционально (404)
│   │   ├── template.tsx         # ⚪ опционально (пересоздаётся при навигации)
│   │   │
│   │   ├── 📁 api/              # ⚪ опционально (API Routes)
│   │   │   └── 📁 users/
│   │   │       └── route.ts     # GET/POST /api/users
│   │   │
│   │   ├── 📁 blog/             # ⚪ опционально (любые роуты)
│   │   │   ├── page.tsx         # /blog
│   │   │   └── 📁 [slug]/       # динамический роут
│   │   │       └── page.tsx     # /blog/:slug
│   │   │
│   │   └── 📁 (auth)/           # ⚪ Route Group (не влияет на URL)
│   │       ├── 📁 login/
│   │       │   └── page.tsx     # /login
│   │       └── 📁 register/
│   │           └── page.tsx     # /register
│   │
│   ├── 📁 components/           # ⚪ опционально (React компоненты)
│   │   ├── 📁 ui/               # UI примитивы (Button, Input)
│   │   ├── 📁 layout/           # Header, Footer, Sidebar
│   │   └── 📁 features/         # Бизнес-компоненты
│   │
│   ├── 📁 lib/                  # ⚪ опционально (утилиты)
│   │   ├── utils.ts
│   │   ├── api.ts
│   │   └── db.ts
│   │
│   ├── 📁 hooks/                # ⚪ опционально (Custom hooks)
│   ├── 📁 types/                # ⚪ опционально (TypeScript типы)
│   └── 📁 styles/               # ⚪ опционально (CSS/SCSS)
│
├── .env                         # ⚪ опционально (переменные окружения)
├── .env.local                   # ⚪ опционально (локальные, не в git)
├── .gitignore                   # 🔷 ОБЯЗАТЕЛЬНО
├── next.config.js               # 🔷 ОБЯЗАТЕЛЬНО (конфигурация)
├── package.json                 # 🔷 ОБЯЗАТЕЛЬНО
├── tsconfig.json                # ⚪ если TypeScript
├── tailwind.config.js           # ⚪ если Tailwind
└── README.md                    # ⚪ рекомендуется
```

### Легенда:
```
🔷 ОБЯЗАТЕЛЬНО  — Next.js не работает без этого
⚪ опционально  — добавляй по необходимости
❌ НЕ КОММИТИТЬ — должно быть в .gitignore
```

### Минимальная структура (Next.js будет работать):
```
my-app/
├── app/
│   ├── layout.tsx    # 🔷 Root Layout
│   └── page.tsx      # 🔷 Главная страница
├── next.config.js    # 🔷 Конфиг
└── package.json      # 🔷 Зависимости
```

### Специальные файлы в папке app/:

```
+----------------+-----------------------------------------------------+
| Файл           | Назначение                                          |
+----------------+-----------------------------------------------------+
| page.tsx       | Страница (URL = путь к папке)                       |
| layout.tsx     | Общий каркас (шапка, футер), сохраняется            |
| template.tsx   | Как layout, но пересоздаётся при навигации          |
| loading.tsx    | UI загрузки (Suspense fallback)                     |
| error.tsx      | UI ошибки (Error Boundary)                          |
| not-found.tsx  | Страница 404                                        |
| route.ts       | API endpoint (вместо page.tsx)                      |
+----------------+-----------------------------------------------------+
```

### Динамические роуты:

```
+---------------+-------------------+-------------------------------+
| Паттерн       | Пример URL        | Как получить                  |
+---------------+-------------------+-------------------------------+
| [slug]        | /blog/hello       | params.slug = "hello"         |
| [...slug]     | /blog/a/b/c       | params.slug = ["a","b","c"]   |
| [[...slug]]   | /blog или /blog/a | опциональный catch-all        |
+---------------+-------------------+-------------------------------+
```

### Route Groups (группировка без влияния на URL):
```
app/
├── (marketing)/      # Группа для маркетинга
│   ├── about/page.tsx    # → /about
│   └── contact/page.tsx  # → /contact
└── (shop)/           # Группа для магазина
    ├── cart/page.tsx     # → /cart
    └── checkout/page.tsx # → /checkout
```

---

## 3. Серверные и клиентские компоненты
- **Серверные (по умолчанию):**
  - выполняются на сервере,
  - отдают готовый HTML,
  - быстрые и хорошие для SEO,
  - можно сразу делать `fetch` к API.

- **Клиентские ("use client"):**
  - выполняются в браузере,
  - нужны для интерактива (поиск, кнопки, лайки),
  - позволяют использовать `useState`, `useEffect`.

Пример:
```tsx
// app/page.tsx (серверный)
import SearchBar from "@/components/SearchBar";
export default function HomePage() {
  return <><h1>Новости</h1><SearchBar /></>;
}

// components/SearchBar.tsx (клиентский)
"use client";
import { useState } from "react";
export default function SearchBar() {
  const [q, setQ] = useState("");
  return <input value={q} onChange={(e) => setQ(e.target.value)} placeholder="Поиск..." />;
}
```

---

## 4. Загрузка данных (Data Fetching)
Пример серверного компонента с загрузкой данных:
```tsx
export default async function Page() {
  const res = await fetch("https://api.example.com/articles", { cache: "no-store" });
  const articles = await res.json();
  return (
    <ul>{articles.map((a: any) => <li key={a.id}>{a.title}</li>)}</ul>
  );
}
```
- `cache: "no-store"` → всегда свежие данные (SSR).  
- `export const revalidate = 300;` → ISR (перегенерация раз в 5 минут).  

---

## 5. API Routes (бэкенд прямо в Next)
```ts
// app/api/subscribe/route.ts
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const { email } = await req.json();
  return NextResponse.json({ ok: true, email });
}
```
Запрос с клиента:
```ts
await fetch("/api/subscribe", {
  method: "POST",
  body: JSON.stringify({ email: "test@mail.com" }),
});
```

---

## 6. SEO и метаданные
```tsx
export const metadata = {
  title: "Главная страница",
  description: "Последние новости об искусственном интеллекте",
};
```

---

## 7. Картинки и шрифты
- `<Image>` из `next/image` оптимизирует изображения.  
- `next/font` подключает шрифты без скачков.  

---

## 8. Tailwind в Next
Подключается при создании проекта.  
Пример:
```tsx
<h1 className="text-2xl font-bold text-blue-500">Заголовок</h1>
```

---

## 9. Ошибки и загрузка
- `error.tsx` → UI при ошибках.  
- `loading.tsx` → экран загрузки.  
- `not-found.tsx` → 404.  

---

## 10. Мини-план для старта
1. Создай проект:
```bash
npx create-next-app@latest
```
Выбери TypeScript + Tailwind.

2. В `app/page.tsx` напиши:
```tsx
export default function Page() {
  return <h1>Привет, Next!</h1>;
}
```

3. Создай папку `article/[slug]` и файл `page.tsx`.  
   Теперь `/article/123` работает.

4. Добавь компонент `ArticleCard`.  
5. Настрой `revalidate = 300`, чтобы лента обновлялась каждые 5 минут.

---

✅ Теперь у тебя есть база для новостного сайта: главная лента, страницы статей, API для подписки и стили через Tailwind.
