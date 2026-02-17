# Build-time vs Run-time

## Уровень 1: Самое простое объяснение

Представь, что ты готовишь пиццу:

### Build-time (время сборки) — это подготовка
- Замесить тесто
- Нарезать ингредиенты
- Разогреть духовку
- Собрать пиццу

**Это делается ОДИН РАЗ, до того как начнёшь есть.**

### Run-time (время выполнения) — это когда ешь
- Достаёшь кусок
- Кусаешь
- Жуёшь

**Это происходит КАЖДЫЙ РАЗ, когда ты ешь.**

---

## Уровень 2: В программировании

### Build-time — что происходит ДО запуска

```
Твой код (исходники)
      ↓
   СБОРКА (build)
      ↓
Готовое приложение
```

**Примеры:**
- Компиляция TypeScript → JavaScript
- Babel превращает современный JS → старый JS
- Webpack собирает все файлы в один
- Минификация (удаление пробелов, сокращение имён)
- Оптимизация изображений

### Run-time — что происходит ПОСЛЕ запуска

```
Готовое приложение
      ↓
   ЗАПУСК
      ↓
Программа работает
```

**Примеры:**
- Выполнение JavaScript в браузере
- Обработка HTTP-запросов
- Работа с базой данных
- Взаимодействие с пользователем

---

## Уровень 3: Где рождаются ошибки

### Build-time ошибки (ловятся ДО запуска)

Эти ошибки ты видишь **при сборке проекта**.

#### Пример 1: Синтаксическая ошибка

```typescript
// app.ts
const name = "Tamerlan"
console.log(name;  // ❌ Лишняя скобка
```

**Сборка:**
```bash
$ npm run build

❌ ERROR: SyntaxError: Unexpected token
Build failed!
```

**Программа НЕ запустится**, пока не исправишь ошибку.

#### Пример 2: Ошибка типов (TypeScript)

```typescript
// app.ts
function add(a: number, b: number): number {
    return a + b;
}

add(5, "10");  // ❌ "10" это не число!
```

**Сборка:**
```bash
$ npm run build

❌ ERROR: Argument of type 'string' is not assignable to parameter of type 'number'
Build failed!
```

#### Пример 3: Отсутствующий модуль

```javascript
// app.js
import { something } from './missing-file.js';  // ❌ Файл не существует
```

**Сборка:**
```bash
$ npm run build

❌ ERROR: Cannot find module './missing-file.js'
Build failed!
```

### Run-time ошибки (ловятся ПОСЛЕ запуска)

Эти ошибки ты видишь **когда программа уже работает**.

#### Пример 1: Деление на ноль

```javascript
function divide(a, b) {
    return a / b;
}

console.log(divide(10, 2));  // ✅ 5
console.log(divide(10, 0));  // ⚠️ Infinity (не ошибка в JS!)
```

**Сборка:** ✅ Успешно  
**Запуск:** ⚠️ Работает, но результат странный

#### Пример 2: Обращение к несуществующему свойству

```javascript
const user = { name: "Tamerlan" };
console.log(user.age.toString());  // ❌ user.age это undefined!
```

**Сборка:** ✅ Успешно  
**Запуск:**
```
❌ TypeError: Cannot read property 'toString' of undefined
```

#### Пример 3: Ошибка сети

```javascript
async function fetchData() {
    const response = await fetch('https://api.example.com/data');
    return response.json();
}

fetchData();  // ❌ Сервер недоступен!
```

**Сборка:** ✅ Успешно  
**Запуск:**
```
❌ Error: Failed to fetch
```

---

## Уровень 4: Практические примеры

### Пример 1: Типичный React проект

#### Исходный код (TypeScript + JSX)

```typescript
// Button.tsx
interface Props {
    label: string;
    onClick: () => void;
}

export function Button({ label, onClick }: Props) {
    return <button onClick={onClick}>{label}</button>;
}
```

#### Build-time (что происходит при `npm run build`)

```
1. TypeScript Compiler (tsc)
   - Проверяет типы ✅
   - Удаляет типы
   ↓
2. Babel / SWC
   - Превращает JSX в JavaScript
   - Транспилирует современный синтаксис
   ↓
3. Webpack / Vite
   - Собирает все файлы
   - Минифицирует код
   - Оптимизирует
   ↓
4. Результат: bundle.js (готовый файл)
```

**Результат (bundle.js):**
```javascript
// Минифицированный код
function Button({label,onClick}){return React.createElement("button",{onClick},label)}
```

#### Run-time (что происходит в браузере)

```
1. Браузер загружает bundle.js
   ↓
2. React создаёт виртуальный DOM
   ↓
3. Пользователь кликает на кнопку
   ↓
4. Вызывается onClick
   ↓
5. Обновляется интерфейс
```

### Пример 2: Next.js приложение

#### Код

```typescript
// app/page.tsx
export default async function Home() {
    const data = await fetch('https://api.example.com/data');
    const json = await data.json();
    
    return <div>{json.title}</div>;
}
```

#### Build-time (`npm run build`)

```
1. TypeScript проверяет типы
   ↓
2. Next.js компилирует страницы
   ↓
3. Создаёт статические HTML файлы (если возможно)
   ↓
4. Оптимизирует изображения
   ↓
5. Создаёт production bundle
```

**Ошибки на этом этапе:**
- ❌ Синтаксические ошибки
- ❌ Ошибки типов
- ❌ Отсутствующие импорты

#### Run-time (когда пользователь открывает сайт)

```
1. Сервер получает запрос
   ↓
2. Выполняется fetch (запрос к API)
   ↓
3. Рендерится HTML
   ↓
4. Отправляется пользователю
```

**Ошибки на этом этапе:**
- ❌ API недоступен
- ❌ Неправильный формат данных
- ❌ Ошибки в логике

---

## Уровень 5: Как ловить ошибки на разных этапах

### Build-time: TypeScript + ESLint

```typescript
// ✅ Хороший код — ошибки ловятся при сборке

interface User {
    name: string;
    age: number;
}

function greet(user: User): string {
    return `Hello, ${user.name}!`;
}

// ❌ Ошибка типа — поймается при сборке!
greet({ name: "Tamerlan" });  // Нет поля age!
```

**Сборка:**
```bash
❌ ERROR: Property 'age' is missing in type '{ name: string; }'
```

### Run-time: try-catch + валидация

```typescript
// ✅ Хороший код — ошибки обрабатываются в runtime

async function fetchUser(id: number) {
    try {
        const response = await fetch(`/api/users/${id}`);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        
        // Валидация данных
        if (!data.name || typeof data.age !== 'number') {
            throw new Error('Invalid user data');
        }
        
        return data;
    } catch (error) {
        console.error('Failed to fetch user:', error);
        return null;  // Возвращаем null вместо краша
    }
}
```

---

## Таблица: Build-time vs Run-time

| Аспект | Build-time | Run-time |
|--------|------------|----------|
| **Когда** | При сборке (`npm run build`) | При выполнении (в браузере/сервере) |
| **Частота** | Один раз перед деплоем | Каждый раз при использовании |
| **Инструменты** | TypeScript, Babel, Webpack, ESLint | V8, браузер, Node.js |
| **Ошибки** | Синтаксис, типы, импорты | Логика, сеть, данные |
| **Исправление** | Обязательно (не соберётся) | Желательно (может крашнуться) |
| **Производительность** | Не важна (делается редко) | Критична (влияет на пользователя) |

---

## Практический пример: npm start

### Что происходит, когда ты пишешь `npm start`?

#### Вариант 1: Development режим

```bash
$ npm start
```

**Что происходит:**

```
1. npm читает package.json
   ↓
2. Находит скрипт "start": "vite"
   ↓
3. Запускает Vite dev server
   ↓
4. Vite:
   - Парсит код (Build-time)
   - Превращает TypeScript → JavaScript (Build-time)
   - Запускает сервер (Run-time)
   - Открывает браузер (Run-time)
   ↓
5. Браузер:
   - Загружает JavaScript (Run-time)
   - Выполняет код (Run-time)
   - Рендерит интерфейс (Run-time)
```

**Ошибки:**
- Build-time: Синтаксис, типы → видишь в терминале
- Run-time: Логика, данные → видишь в консоли браузера

#### Вариант 2: Production режим

```bash
$ npm run build
$ npm start
```

**Что происходит:**

```
1. npm run build (Build-time):
   - TypeScript → JavaScript
   - Babel транспилирует
   - Webpack собирает bundle
   - Минифицирует код
   - Оптимизирует
   ↓
2. npm start (Run-time):
   - Запускает production сервер
   - Отдаёт готовые файлы
   - Пользователь видит сайт
```

---

## Оптимизация: что делать где?

### Build-time оптимизации (делаются один раз)

```javascript
// ❌ Плохо — делать в runtime
function Component() {
    const config = JSON.parse(configString);  // Парсинг каждый раз!
    return <div>{config.title}</div>;
}

// ✅ Хорошо — сделать в build-time
const config = JSON.parse(configString);  // Парсинг один раз при сборке

function Component() {
    return <div>{config.title}</div>;
}
```

### Run-time оптимизации (делаются каждый раз)

```javascript
// ❌ Плохо — медленно
function Component({ items }) {
    return (
        <div>
            {items.map(item => (
                <ExpensiveComponent key={item.id} data={item} />
            ))}
        </div>
    );
}

// ✅ Хорошо — используем мемоизацию
const MemoizedComponent = React.memo(ExpensiveComponent);

function Component({ items }) {
    return (
        <div>
            {items.map(item => (
                <MemoizedComponent key={item.id} data={item} />
            ))}
        </div>
    );
}
```

---

## Главные выводы

### Build-time
- **Когда:** При сборке проекта (`npm run build`)
- **Инструменты:** TypeScript, Babel, Webpack, Vite
- **Ошибки:** Синтаксис, типы, импорты
- **Цель:** Подготовить код для выполнения

### Run-time
- **Когда:** При выполнении программы
- **Инструменты:** V8, браузер, Node.js
- **Ошибки:** Логика, сеть, данные
- **Цель:** Выполнить код и дать результат

### Золотое правило
- **Делай в build-time** всё, что можно сделать заранее
- **Делай в run-time** только то, что зависит от пользователя

### Как ловить ошибки
- **Build-time:** TypeScript, ESLint, тесты
- **Run-time:** try-catch, валидация, мониторинг

**Чем больше ошибок поймаешь в build-time, тем меньше крашей в production!** 🎯
