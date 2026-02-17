# Runtime: Node.js, Deno, Bun

## Уровень 1: Что такое Runtime?

**Runtime** — это окружение, в котором выполняется твой код. Это как операционная система для твоей программы.

### Простая аналогия

Представь, что твой код — это актёр:
- **Runtime** — это сцена, декорации, свет, звук
- **V8** — это талант актёра (умение играть)
- **API** — это реквизит (мебель, костюмы)

Актёр (код) может играть на разных сценах (runtime), но ему нужны декорации (API).

---

## Уровень 2: Что входит в Runtime?

Runtime = **JavaScript Engine** + **API** + **Инструменты**

```
┌─────────────────────────────────────┐
│          RUNTIME (Node.js)          │
├─────────────────────────────────────┤
│  JavaScript Engine (V8)             │
│  - Выполняет JS код                 │
│  - JIT-компиляция                   │
│  - Garbage Collection               │
├─────────────────────────────────────┤
│  API (что доступно в коде)          │
│  - fs (работа с файлами)            │
│  - http (создание серверов)         │
│  - process (информация о процессе)  │
│  - Buffer (работа с бинарными данными)│
├─────────────────────────────────────┤
│  Инструменты                        │
│  - npm (менеджер пакетов)           │
│  - node (запуск кода)               │
└─────────────────────────────────────┘
```

---

## Node.js — классика

### Что это?
**Node.js** — первый и самый популярный runtime для JavaScript вне браузера.

### Основные характеристики

| Характеристика | Описание |
|----------------|----------|
| **Engine** | V8 (от Google Chrome) |
| **Создан** | 2009 год |
| **Пакетный менеджер** | npm (самый большой реестр пакетов) |
| **Модули** | CommonJS (`require`) + ES Modules (`import`) |
| **TypeScript** | Нужны дополнительные инструменты |

### Пример кода

```javascript
// Работа с файлами
const fs = require('fs');

fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) throw err;
    console.log(data);
});

// Создание HTTP-сервера
const http = require('http');

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World!');
});

server.listen(3000);
```

### Плюсы
- ✅ Огромная экосистема (npm)
- ✅ Много библиотек и фреймворков
- ✅ Большое сообщество
- ✅ Стабильный и проверенный временем

### Минусы
- ❌ Медленный старт (долго загружается)
- ❌ Устаревшие API
- ❌ Нет встроенной поддержки TypeScript
- ❌ Проблемы с безопасностью (пакеты имеют полный доступ)

---

## Deno — безопасность и современность

### Что это?
**Deno** — современный runtime, созданный автором Node.js (Ryan Dahl), чтобы исправить ошибки Node.js.

### Основные характеристики

| Характеристика | Описание |
|----------------|----------|
| **Engine** | V8 (тот же, что в Node.js) |
| **Создан** | 2020 год |
| **Пакетный менеджер** | Нет! Импорт по URL |
| **Модули** | Только ES Modules (`import`) |
| **TypeScript** | Встроенная поддержка! |

### Пример кода

```typescript
// Работа с файлами (нужно разрешение!)
const text = await Deno.readTextFile("file.txt");
console.log(text);

// Создание HTTP-сервера
Deno.serve({ port: 3000 }, (req) => {
    return new Response("Hello World!");
});

// Импорт по URL (без npm!)
import { serve } from "https://deno.land/std@0.200.0/http/server.ts";
```

### Ключевые отличия от Node.js

#### 1. Безопасность по умолчанию

**Node.js:**
```javascript
// Код может делать ЧТО УГОДНО без разрешения!
const fs = require('fs');
fs.unlinkSync('/important-file.txt');  // Удалил файл!
```

**Deno:**
```typescript
// Нужно явное разрешение!
const text = await Deno.readTextFile("file.txt");
// ❌ ОШИБКА: Requires --allow-read permission
```

Запуск с разрешениями:
```bash
deno run --allow-read app.ts        # Только чтение
deno run --allow-net app.ts         # Только сеть
deno run --allow-all app.ts         # Всё (как Node.js)
```

#### 2. TypeScript из коробки

**Node.js:**
```bash
npm install -D typescript @types/node
npx tsc app.ts
node app.js
```

**Deno:**
```bash
deno run app.ts  # Просто работает! ✨
```

#### 3. Импорт по URL

**Node.js:**
```javascript
// Нужно устанавливать пакет
npm install express
const express = require('express');
```

**Deno:**
```typescript
// Импорт напрямую по URL
import { Application } from "https://deno.land/x/oak@v12.0.0/mod.ts";
```

### Плюсы
- ✅ Безопасность (разрешения)
- ✅ TypeScript из коробки
- ✅ Современные API (Promise-based)
- ✅ Встроенные инструменты (formatter, linter, test runner)
- ✅ Совместимость с Web API (fetch, WebSocket)

### Минусы
- ❌ Меньше пакетов, чем npm
- ❌ Не все Node.js пакеты работают
- ❌ Меньшее сообщество

---

## Bun — скорость превыше всего

### Что это?
**Bun** — самый быстрый runtime, написанный с нуля для максимальной производительности.

### Основные характеристики

| Характеристика | Описание |
|----------------|----------|
| **Engine** | JavaScriptCore (от Safari, не V8!) |
| **Создан** | 2022 год |
| **Пакетный менеджер** | Встроенный (совместим с npm) |
| **Модули** | CommonJS + ES Modules |
| **TypeScript** | Встроенная поддержка! |

### Пример кода

```typescript
// Работа с файлами
const file = Bun.file("file.txt");
const text = await file.text();
console.log(text);

// Создание HTTP-сервера
Bun.serve({
    port: 3000,
    fetch(req) {
        return new Response("Hello World!");
    },
});

// Импорт работает как в Node.js
import express from "express";
```

### Ключевые особенности

#### 1. Безумная скорость

**Установка пакетов:**
```bash
npm install express      # Node.js: ~10 секунд
bun install express      # Bun: ~0.5 секунды 🚀
```

**Запуск кода:**
```bash
node app.js              # Node.js: ~200ms старт
bun app.js               # Bun: ~20ms старт ⚡
```

#### 2. Всё в одном

```bash
bun run app.ts           # Запуск TypeScript
bun test                 # Запуск тестов
bun install              # Установка пакетов
bun build app.ts         # Сборка для продакшена
```

#### 3. Совместимость с Node.js

```javascript
// Node.js код работает в Bun!
const fs = require('fs');
const http = require('http');

// Всё работает! ✅
```

### Плюсы
- ✅ Очень быстрый (в 2-3 раза быстрее Node.js)
- ✅ TypeScript из коробки
- ✅ Совместим с npm пакетами
- ✅ Встроенные инструменты (bundler, test runner)
- ✅ Низкое потребление памяти

### Минусы
- ❌ Молодой проект (могут быть баги)
- ❌ Не все Node.js API реализованы
- ❌ Меньше документации

---

## Сравнение: Node.js vs Deno vs Bun

| Критерий | Node.js | Deno | Bun |
|----------|---------|------|-----|
| **Engine** | V8 | V8 | JavaScriptCore |
| **Скорость старта** | Медленно | Средне | Очень быстро ⚡ |
| **Скорость выполнения** | Быстро | Быстро | Очень быстро ⚡ |
| **TypeScript** | Нужна настройка | Из коробки ✅ | Из коробки ✅ |
| **Пакеты** | npm (огромный) | URL + npm | npm ✅ |
| **Безопасность** | Нет | Разрешения ✅ | Нет |
| **Совместимость** | - | Частичная | Высокая ✅ |
| **Экосистема** | Огромная ✅ | Растущая | Растущая |
| **Стабильность** | Очень стабильно ✅ | Стабильно | В разработке ⚠️ |

---

## Когда что использовать?

### Node.js — выбирай, если:
- ✅ Нужна максимальная стабильность
- ✅ Используешь много npm пакетов
- ✅ Работаешь в команде (все знают Node.js)
- ✅ Продакшен-проект

**Примеры:** Express, NestJS, Next.js

### Deno — выбирай, если:
- ✅ Важна безопасность
- ✅ Хочешь современные API
- ✅ Пишешь на TypeScript
- ✅ Новый проект без зависимостей от Node.js

**Примеры:** API сервисы, CLI инструменты, скрипты

### Bun — выбирай, если:
- ✅ Нужна максимальная скорость
- ✅ Разработка (быстрая итерация)
- ✅ Хочешь всё в одном инструменте
- ✅ Готов к экспериментам

**Примеры:** Разработка, тестирование, прототипы

---

## Практический пример: один код, три runtime

### Код (app.ts)

```typescript
// Простой HTTP-сервер
const port = 3000;

// Node.js
import http from 'http';
http.createServer((req, res) => {
    res.end('Hello from Node.js!');
}).listen(port);

// Deno
Deno.serve({ port }, () => new Response('Hello from Deno!'));

// Bun
Bun.serve({
    port,
    fetch: () => new Response('Hello from Bun!'),
});
```

### Запуск

```bash
# Node.js
node app.js

# Deno
deno run --allow-net app.ts

# Bun
bun app.ts
```

---

## Главные выводы

### Runtime — это:
- JavaScript Engine (V8 или JavaScriptCore)
- API для работы с системой (файлы, сеть, и т.д.)
- Инструменты (пакетный менеджер, тестирование)

### Три основных runtime:
1. **Node.js** — стабильный, популярный, огромная экосистема
2. **Deno** — безопасный, современный, TypeScript из коробки
3. **Bun** — быстрый, всё в одном, совместим с Node.js

### Выбор зависит от:
- Требований проекта
- Команды (что знают)
- Приоритетов (скорость vs стабильность vs безопасность)

**Для обучения:** Начни с Node.js, потом попробуй Bun для скорости!
