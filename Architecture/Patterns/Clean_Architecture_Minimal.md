# Clean Architecture (Minimal Full-Stack Implementation)

Привет! Рад, что ты решил копнуть в эту тему. Спустя год разработки ты наверняка замечал, как проекты превращаются в "спагетти", где логика размазана по контроллерам, а смена базы данных звучит как ночной кошмар. Clean Architecture (Чистая Архитектура) Роберта Мартина — это вакцина от такой болезни.

Давай разберем минималистичный подход, без фанатизма и лишних слоев, который реально работает в продакшене.

---

## 1. 🧠 Ментальная модель (Аналогия из жизни)

Представь **современный смартфон**.

*   **Ядро (Business Logic):** Это сам процессор и операционная система. Им все равно, какой чехол на телефоне или какие наушники ты подключил. Они выполняют вычисления.
*   **Интерфейсы (Adapters):** Это порт зарядки (USB-C) или Bluetooth. Они определяют стандарт общения.
*   **Внешний мир (Details):** Это зарядное устройство, наушники, чехол. Ты можешь сменить зарядку с белой на черную, наушники с проводных на беспроводные — телефон (Ядро) при этом переделывать не нужно.

**Clean Architecture** делает твой код таким же: бизнес-логика (твое "золото") находится в центре и ничего не знает о внешнем мире (React, Express, Postgres, Redis). Внешний мир подключается к ней как аксессуары через стандартные разъемы (интерфейсы).

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ВНЕШНИЙ МИР (Details)                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                      ADAPTERS (Интерфейсы)                          │   │
│   │   ┌─────────────────────────────────────────────────────────────┐   │   │
│   │   │                  USE CASES (Сценарии)                       │   │   │
│   │   │   ┌─────────────────────────────────────────────────────┐   │   │   │
│   │   │   │                                                     │   │   │   │
│   │   │   │              ENTITIES (Сущности)                    │   │   │   │
│   │   │   │                 Бизнес-логика                       │   │   │   │
│   │   │   │                    ★ ЯДРО ★                         │   │   │   │
│   │   │   │                                                     │   │   │   │
│   │   │   └─────────────────────────────────────────────────────┘   │   │   │
│   │   │              RegisterUser, CreateOrder...                   │   │   │
│   │   └─────────────────────────────────────────────────────────────┘   │   │
│   │           Controllers, Repositories, Presenters                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│               React, Express, PostgreSQL, Redis, REST API                   │
└─────────────────────────────────────────────────────────────────────────────┘

    ═══════════════════════════════════════════════════════════════════════
                    ПРАВИЛО: Зависимости направлены ВНУТРЬ →
    ═══════════════════════════════════════════════════════════════════════
```

---

## 2. 💼 Зачем это нужно бизнесу и архитектуре

Бизнесу плевать на красоту кода, ему нужны **скорость изменений** и **стабильность**.

1.  **Независимость от Фреймворков:** React обновился с breaking changes? Переезжаем с Express на Fastify? Твоя бизнес-логика этого даже не заметит. Ты меняешь "обертку", а не "суть".
2.  **Тестируемость (Testability):** Ты можешь протестировать логику оформления заказа без поднятия базы данных и сервера. Просто запускаешь unit-тесты ядра за миллисекунды.
3.  **Гибкость (Decoupling):** Заказчик захотел сменить отправку SMS с Twilio на местный шлюз? Ты пишешь новый адаптер, не трогая код, который рассчитывает скидки.

---

## 3. 💻 Код: Было vs Стало

### ❌ Anti-pattern: "Слоеный пирог" с жесткими связями (Spaghetti Controller)

Типичный код, где контроллер знает слишком много. Он сам ходит в базу, сам считает логику, сам шлет ответы.

```javascript
// userController.js (Bad Practice)
const db = require('./database'); // Прямая зависимость от БД

async function registerUser(req, res) {
  const { email, password } = req.body;

  // 1. Валидация в контроллере (смешивание ответственности)
  if (!email.includes('@')) return res.status(400).send('Invalid email');

  // 2. Бизнес-логика смешана с инфраструктурой
  const existingUser = await db.query('SELECT * FROM users WHERE email = $1', [email]);
  if (existingUser.rows.length > 0) {
    return res.status(409).send('User exists');
  }

  // 3. Прямой вызов SQL
  await db.query('INSERT INTO users (email, password) VALUES ($1, $2)', [email, password]);

  res.json({ success: true });
}
```

### ✅ Best Practice: Clean Architecture (Minimal)

Мы разделяем слои. Контроллер тупой. UseCase (Интерактор) умный. Repository — просто кладовщик.

Структура папок:

```
src/
├── domain/                    # ★ ЯДРО (Чистый код, 0 зависимостей)
│   └── entities/
│       └── User.ts            # Сущность + бизнес-правила
│
├── application/               # USE CASES (Сценарии)
│   ├── use-cases/
│   │   └── RegisterUser.ts    # Логика "Зарегистрировать пользователя"
│   └── interfaces/
│       └── IUserRepository.ts # Контракт (что нужно от хранилища)
│
├── infrastructure/            # ДЕТАЛИ (Всё, что можно заменить)
│   ├── repositories/
│   │   └── PostgresUserRepository.ts  # Реализация контракта
│   ├── http/
│   │   └── UserController.ts  # Express/Fastify
│   └── db/
│       └── connection.ts
│
└── main.ts                    # Точка входа, Composition Root (DI)
```

```typescript
// 1. Domain (Ядро) - Чистый JS/TS, никаких зависимостей!
// entities/User.ts
export class User {
  constructor(public id: string, public email: string) {
    if (!email.includes('@')) throw new Error("Invalid email");
  }
}

// 2. Application (Use Case) - Сценарий использования
// use-cases/RegisterUser.ts
import { IUserRepository } from '../interfaces/IUserRepository';

export class RegisterUser {
  constructor(private userRepo: IUserRepository) {} // Dependency Injection!

  async execute(email: string, password: string): Promise<User> {
    const exists = await this.userRepo.findByEmail(email);
    if (exists) throw new Error("User already exists");

    const newUser = new User(Date.now().toString(), email);
    // ... тут может быть хеширование пароля через сервис ...
    await this.userRepo.save(newUser);
    return newUser;
  }
}

// 3. Interface (Контракт для внешнего мира)
// interfaces/IUserRepository.ts
export interface IUserRepository {
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// 4. Infrastructure (Реализация деталей)
// repositories/PostgresUserRepository.ts
import { IUserRepository } from '../../application/interfaces/IUserRepository';
import { User } from '../../domain/entities/User';
import db from './db';

export class PostgresUserRepository implements IUserRepository {
  async findByEmail(email: string): Promise<User | null> {
    const res = await db.query('SELECT * FROM users WHERE email = $1', [email]);
    return res.rows[0] ? new User(res.rows[0].id, res.rows[0].email) : null;
  }
  async save(user: User): Promise<void> {
    await db.query('INSERT INTO users ...', [user.id, user.email]);
  }
}

// 5. Controller (Склейка)
// controllers/UserController.ts
const userRepo = new PostgresUserRepository(); // В реальности используем DI Container
const registerUseCase = new RegisterUser(userRepo);

export async function register(req, res) {
  try {
    const user = await registerUseCase.execute(req.body.email, req.body.password);
    res.json(user);
  } catch (e) {
    res.status(400).json({ error: e.message });
  }
}
```

**В чем разница?**
`RegisterUser` (Use Case) ничего не знает про SQL или Express. Он работает с *абстракцией* `IUserRepository`. Мы можем подменить `PostgresUserRepository` на `InMemoryUserRepository` для тестов за 1 минуту.

---

## 4. 🛠 Under the Hood (Как это работает внутри)

Ключевой механизм здесь — **Dependency Inversion Principle (DIP)** из SOLID.

### ❌ Обычная архитектура (Зависимость от деталей)

```
┌──────────────────┐         ┌──────────────────┐
│   RegisterUser   │ ──────▶ │  PostgresRepo    │
│   (Use Case)     │ import  │  (Конкретный)    │
└──────────────────┘         └──────────────────┘
        │                            │
        │   ПРОБЛЕМА: Жёсткая        │
        │   связь. Нельзя           │
        │   подменить БД.           │
        ▼                            ▼
  "Высокий уровень"           "Низкий уровень"
   зависит от низкого
```

### ✅ Clean Architecture (Инверсия зависимостей)

```
                        АБСТРАКЦИЯ
                   ┌──────────────────┐
                   │ IUserRepository  │  ← interface (контракт)
                   │  (Интерфейс)     │
                   └────────▲─────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          │ implements      │ uses            │ implements
          │                 │                 │
┌─────────┴────────┐  ┌─────┴──────┐  ┌───────┴──────────┐
│ PostgresUserRepo │  │ RegisterUser│  │ InMemoryUserRepo │
│ (Production)     │  │ (Use Case) │  │ (Для тестов)     │
└──────────────────┘  └────────────┘  └──────────────────┘

═══════════════════════════════════════════════════════════
  ОБЕ стороны зависят от абстракции.
  Use Case не знает, откуда данные — из Postgres или памяти.
═══════════════════════════════════════════════════════════
```

Получается, что и бизнес-логика, и БД зависят от *абстракции*, которой владеет бизнес-логика.

Это работает как плагин. Твое приложение определяет "гнездо" (интерфейс), а база данных — это "вилка", которая в него втыкается. Стрелки зависимостей в коде всегда направлены **ВНУТРЬ** круга (к домену).

---

## 5. 🔥 Level Up: Production Ready (Самая важная часть)

В реальном мире (HighLoad, Enterprise) ты столкнешься с тем, о чем молчат туториалы.

### Типичные ошибки (Common Pitfalls)
1.  **Anemic Domain Model:** Создание сущностей, которые просто хранят данные (getters/setters), а вся логика лежит в сервисах. Старайся держать логику валидации и поведения ВНУТРИ сущностей (`User.changePassword(...)`, а не `UserService.changePassword(user, ...)`).
2.  **Leaking Details:** Когда типы из БД (например, `ObjectId` из Mongo или `RowDataPacket` из MySQL) просачиваются в UseCase или Domain. Всегда делай маппинг (Mapper) на границах слоев.

### Надежность и Масштабирование
*   **Транзакции:** Самая большая боль. UseCase должен быть атомарным. Но как управлять транзакцией, если UseCase не должен знать про БД?
    *   *Решение:* Паттерн `UnitOfWork`. UseCase получает UoW, делает работу и вызывает `uow.commit()`.
*   **DTO Hell:** Тебе придется писать много классов для перекладывания данных.
    *   *Решение:* Используй мапперы (AutoMapper) или смирись. Это цена за чистоту. В простых случаях для чтения (Read Model) можно ходить в базу напрямую (CQRS), минуя сложную логику записи.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Поток данных (Data Flow)                            │
└─────────────────────────────────────────────────────────────────────────────┘

  HTTP Request                                                  HTTP Response
       │                                                              ▲
       ▼                                                              │
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌────────┴─────┐
│  Controller  │───▶│  RequestDTO  │───▶│   UseCase    │───▶│ ResponseDTO  │
│  (Adapter)   │    │  (Mapping)   │    │ (Application)│    │  (Mapping)   │
└──────────────┘    └──────────────┘    └──────┬───────┘    └──────────────┘
                                               │
                                               ▼
                                        ┌──────────────┐
                                        │   Entity     │
                                        │   (Domain)   │
                                        └──────┬───────┘
                                               │
                                               ▼
                                        ┌──────────────┐
                                        │  Repository  │──────▶ Database
                                        │ (Interface)  │
                                        └──────────────┘

 ════════════════════════════════════════════════════════════════════════════
  На каждой границе слоёв: Mapping!  RequestDTO ≠ Entity ≠ DB Row
 ════════════════════════════════════════════════════════════════════════════
```

### Краевые случаи (Edge Cases)
*   **Идемпотентность:** Если UseCase вызывается дважды (клиент нажал кнопку два раза), Clean Architecture сама по себе не спасет. Тебе нужно продумать механизм идемпотентности (например, передавать уникальный `requestId` в UseCase).
*   **Race Conditions:** Проверка `if (exists)` и последующая запись `save` в примере выше — это дыра для гонки состояний. В HighLoad это решается уникальными индексами в БД (на уровне Infrastructure) или блокировками, но UseCase должен быть готов обработать ошибку дубликата.

---

## 6. ⚖️ Когда НЕ нужно это использовать

Clean Architecture — это "тяжелая артиллерия". Не стреляй из пушки по воробьям.

**Не используй, если:**
1.  **Простой CRUD:** Если твое приложение просто перекладывает JSON из запроса в базу и обратно без сложной логики. Ты просто утонешь в создании интерфейсов и классов.
2.  **MVP / Прототип:** Когда нужно проверить гипотезу за 2 дня. Пиши "грязно", в один файл. Потом выкинешь или отрефакторишь (если выживет).
3.  **Скрипты и утилиты:** Тут важна краткость.

**Используй, когда:**
*   Приложение планирует жить долго (годы).
*   Сложная бизнес-логика (правила скидок, транзакции, сложные статусы заказов).
*   Команда растет (четкое разделение ответственности помогает не наступать друг другу на ноги).
