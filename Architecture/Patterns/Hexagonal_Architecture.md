# Hexagonal Architecture (Гексагональная Архитектура)

Привет! Давай разберём **Hexagonal Architecture** — она же **Ports & Adapters** (Порты и Адаптеры). Этот паттерн придумал Алистер Кокбёрн в 2005 году, чтобы решить главную проблему: **как изолировать бизнес-логику от внешнего мира**.

Представь, что твоё приложение — это **крепость** 🏰. Внутри — ценная бизнес-логика. Снаружи — враги (базы данных, API, UI), которые хотят её "загрязнить". Порты — это **ворота**, а адаптеры — **мосты** к внешнему миру.

---

## 🎯 Главная идея

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   Внешний мир НЕ ДОЛЖЕН диктовать, как работает твоя бизнес-логика.         │
│   Бизнес-логика сама определяет, ЧТО ей нужно (через порты).                │
│   Внешний мир лишь ПОДСТРАИВАЕТСЯ (через адаптеры).                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🖼 Визуальная схема

```
                         ┌─────────────────┐
                         │   REST API      │
                         │   (Adapter)     │
                         └────────┬────────┘
                                  │
                                  ▼
┌─────────────┐           ┌──────────────┐           ┌─────────────┐
│   CLI       │           │              │           │  PostgreSQL │
│  (Adapter)  │──────────►│    PORT      │◄──────────│  (Adapter)  │
└─────────────┘           │  (Interface) │           └─────────────┘
                          └──────┬───────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │                        │
                    │      APPLICATION       │
                    │         CORE           │
                    │                        │
                    │  ┌──────────────────┐  │
                    │  │     DOMAIN       │  │
                    │  │   (бизнес-логика)│  │
                    │  └──────────────────┘  │
                    │                        │
                    └────────────────────────┘
                                 │
                                 ▼
                          ┌──────────────┐
                          │    PORT      │
                          │  (Interface) │
                          └──────┬───────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        ▼                        ▼                        ▼
┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│    Email    │          │    Redis    │          │   Stripe    │
│   (Adapter) │          │  (Adapter)  │          │  (Adapter)  │
└─────────────┘          └─────────────┘          └─────────────┘
```

---

## 🧩 Три ключевых понятия

### 1. Application Core (Ядро приложения)

📍 **Где находится:** В самом центре.
❤️ **Что внутри:** Domain (сущности, бизнес-правила) + Application Services (use cases).

Это "крепость". Здесь живёт вся бизнес-логика. Ядро **ничего не знает** о базах данных, HTTP, email-сервисах. Оно знает только о **портах** (интерфейсах).

```
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION CORE                           │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                      DOMAIN                              │    │
│  │                                                         │    │
│  │   class Order:                                          │    │
│  │       def __init__(self, items, customer):              │    │
│  │           if len(items) == 0:                           │    │
│  │               raise ValueError("Заказ не может быть     │    │
│  │                                 пустым")                │    │
│  │           self.items = items                            │    │
│  │           self.customer = customer                      │    │
│  │           self.status = "new"                           │    │
│  │                                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   APPLICATION SERVICES                   │    │
│  │                                                         │    │
│  │   class CreateOrderUseCase:                             │    │
│  │       def __init__(self, order_repo, email_service):    │    │
│  │           self.order_repo = order_repo  # ← PORT!       │    │
│  │           self.email_service = email_service  # ← PORT! │    │
│  │                                                         │    │
│  │       def execute(self, dto):                           │    │
│  │           order = Order(dto.items, dto.customer)        │    │
│  │           self.order_repo.save(order)                   │    │
│  │           self.email_service.send_confirmation(order)   │    │
│  │           return order                                  │    │
│  │                                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

### 2. Ports (Порты)

📍 **Что это:** Интерфейсы (абстракции), которые определяет ядро.
🚪 **Роль:** "Ворота" в крепость. Ядро говорит: "Мне нужен кто-то, кто умеет сохранять заказы", но не говорит КАК.

**Два типа портов:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   DRIVING PORTS (Входящие)              DRIVEN PORTS (Исходящие)            │
│   "Кто может меня вызвать?"             "Что мне нужно для работы?"         │
│                                                                             │
│   ┌─────────────────────────┐           ┌─────────────────────────┐         │
│   │ interface OrderService  │           │ interface OrderRepo     │         │
│   │ {                       │           │ {                       │         │
│   │   createOrder(dto)      │           │   save(order)           │         │
│   │   cancelOrder(id)       │           │   findById(id)          │         │
│   │ }                       │           │ }                       │         │
│   └─────────────────────────┘           └─────────────────────────┘         │
│                                                                             │
│   Используют:                           Используют:                         │
│   - REST Controller                     - PostgreSQL Adapter                │
│   - CLI Command                         - MongoDB Adapter                   │
│   - GraphQL Resolver                    - InMemory Adapter (для тестов)     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Пример порта (Python):**

```python
# ports/order_repository.py
from abc import ABC, abstractmethod

class OrderRepository(ABC):
    """ПОРТ: Ядро определяет, ЧТО ему нужно"""
    
    @abstractmethod
    def save(self, order: Order) -> None:
        pass
    
    @abstractmethod
    def find_by_id(self, order_id: str) -> Order:
        pass
```

---

### 3. Adapters (Адаптеры)

📍 **Что это:** Конкретные реализации портов.
🌉 **Роль:** "Мосты" между ядром и внешним миром.

**Два типа адаптеров:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   PRIMARY ADAPTERS (Входящие)           SECONDARY ADAPTERS (Исходящие)      │
│   "Как внешний мир вызывает ядро"       "Как ядро взаимодействует с миром"  │
│                                                                             │
│   ┌─────────────────────────┐           ┌─────────────────────────┐         │
│   │                         │           │                         │         │
│   │   REST Controller       │           │   PostgresOrderRepo     │         │
│   │   GraphQL Resolver      │           │   MongoOrderRepo        │         │
│   │   CLI Command           │           │   RedisCache            │         │
│   │   Message Consumer      │           │   SmtpEmailService      │         │
│   │   gRPC Handler          │           │   StripePayment         │         │
│   │                         │           │                         │         │
│   └─────────────────────────┘           └─────────────────────────┘         │
│                                                                             │
│   Вызывают ПОРТЫ ядра                   Реализуют ПОРТЫ ядра                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Пример адаптера (Python):**

```python
# adapters/postgres_order_repository.py
from ports.order_repository import OrderRepository

class PostgresOrderRepository(OrderRepository):
    """АДАПТЕР: Реализует порт для PostgreSQL"""
    
    def __init__(self, connection):
        self.connection = connection
    
    def save(self, order: Order) -> None:
        # SQL-специфичный код
        self.connection.execute(
            "INSERT INTO orders (id, customer, status) VALUES (?, ?, ?)",
            (order.id, order.customer, order.status)
        )
    
    def find_by_id(self, order_id: str) -> Order:
        row = self.connection.execute(
            "SELECT * FROM orders WHERE id = ?", (order_id,)
        ).fetchone()
        return Order(row['id'], row['customer'], row['status'])
```

---

## 📁 Пример структуры папок

```
src/
├── core/                          # 🏰 ЯДРО (не зависит ни от чего!)
│   ├── domain/
│   │   ├── entities/
│   │   │   └── order.py           # Сущность Order
│   │   └── value_objects/
│   │       └── money.py           # Value Object Money
│   │
│   ├── ports/                     # 🚪 ПОРТЫ (интерфейсы)
│   │   ├── inbound/               # Входящие порты
│   │   │   └── order_service.py   # interface OrderService
│   │   └── outbound/              # Исходящие порты
│   │       ├── order_repository.py
│   │       └── email_service.py
│   │
│   └── application/               # Use Cases
│       └── create_order_use_case.py
│
├── adapters/                      # 🌉 АДАПТЕРЫ (реализации)
│   ├── inbound/                   # Входящие адаптеры
│   │   ├── rest/
│   │   │   └── order_controller.py
│   │   └── cli/
│   │       └── order_commands.py
│   │
│   └── outbound/                  # Исходящие адаптеры
│       ├── persistence/
│       │   ├── postgres_order_repo.py
│       │   └── in_memory_order_repo.py  # Для тестов!
│       └── external/
│           ├── smtp_email_service.py
│           └── stripe_payment.py
│
└── main.py                        # Сборка: подключаем адаптеры к портам
```

---

## 📝 Пример потока "Создание заказа"

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  👤 Пользователь отправляет POST /orders                                     │
└──────────────────────┬───────────────────────────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  PRIMARY ADAPTER     │  OrderController (REST)                               │
│  (Входящий)          │  Преобразует HTTP Request → DTO                       │
│                      │  Вызывает порт: order_service.create_order(dto)       │
└──────────────────────┼───────────────────────────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  DRIVING PORT        │  interface OrderService                               │
│  (Входящий порт)     │  create_order(dto) → Order                            │
└──────────────────────┼───────────────────────────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  APPLICATION CORE    │  CreateOrderUseCase.execute(dto)                      │
│                      │                                                       │
│                      │  1. Создаёт Order (Domain)                            │
│                      │  2. Вызывает order_repo.save(order)     ← DRIVEN PORT │
│                      │  3. Вызывает email_service.send(order)  ← DRIVEN PORT │
└──────────────────────┼───────────────────────────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  DRIVEN PORT         │  interface OrderRepository                            │
│  (Исходящий порт)    │  save(order)                                          │
└──────────────────────┼───────────────────────────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│  SECONDARY ADAPTER   │  PostgresOrderRepository                              │
│  (Исходящий)         │  INSERT INTO orders...                                │
└──────────────────────┴───────────────────────────────────────────────────────┘
```

---

## 🔧 Как собрать всё вместе (Dependency Injection)

```python
# main.py — точка сборки приложения

from core.application.create_order_use_case import CreateOrderUseCase
from adapters.outbound.persistence.postgres_order_repo import PostgresOrderRepository
from adapters.outbound.external.smtp_email_service import SmtpEmailService
from adapters.inbound.rest.order_controller import OrderController

# 1. Создаём исходящие адаптеры
order_repo = PostgresOrderRepository(db_connection)
email_service = SmtpEmailService(smtp_config)

# 2. Создаём use case, внедряя адаптеры через порты
create_order_use_case = CreateOrderUseCase(
    order_repo=order_repo,        # Адаптер реализует порт OrderRepository
    email_service=email_service   # Адаптер реализует порт EmailService
)

# 3. Создаём входящий адаптер
controller = OrderController(create_order_use_case)

# 4. Запускаем сервер
app.register(controller)
app.run()
```

---

## ✅ Плюсы и ❌ Минусы

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ✅ ПЛЮСЫ                                                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  • ТЕСТИРУЕМОСТЬ                                                            │
│    Можно тестировать ядро без БД, HTTP, email.                              │
│    Просто подставляешь InMemoryOrderRepo вместо PostgresOrderRepo.          │
│                                                                             │
│  • ЗАМЕНЯЕМОСТЬ                                                             │
│    Хочешь сменить PostgreSQL на MongoDB? Пишешь новый адаптер.              │
│    Ядро не трогаешь вообще.                                                 │
│                                                                             │
│  • НЕЗАВИСИМОСТЬ ОТ ФРЕЙМВОРКОВ                                             │
│    Ядро не знает про Flask, Django, FastAPI.                                │
│    Можешь сменить фреймворк, переписав только адаптеры.                     │
│                                                                             │
│  • ПАРАЛЛЕЛЬНАЯ РАЗРАБОТКА                                                  │
│    Один разработчик пишет ядро, другой — адаптеры.                          │
│    Договорились о портах (интерфейсах) и работают независимо.               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│  ❌ МИНУСЫ                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  • СЛОЖНОСТЬ                                                                │
│    Много файлов, папок, интерфейсов. Для простого CRUD — overkill.          │
│                                                                             │
│  • BOILERPLATE                                                              │
│    Порт + Адаптер + Injection = много кода для простых вещей.               │
│                                                                             │
│  • КРИВАЯ ОБУЧЕНИЯ                                                          │
│    Junior-у сложно понять, зачем столько абстракций.                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## ⚠️ Частые ошибки Junior-ов

### Ошибка 1: Ядро импортирует адаптер

```python
# ❌ ПЛОХО — ядро знает про PostgreSQL
# core/application/create_order_use_case.py

from adapters.outbound.persistence.postgres_order_repo import PostgresOrderRepository  # НЕТ!

class CreateOrderUseCase:
    def __init__(self):
        self.repo = PostgresOrderRepository()  # ❌ Жёсткая зависимость!
```

```python
# ✅ ХОРОШО — ядро знает только про порт
# core/application/create_order_use_case.py

from core.ports.outbound.order_repository import OrderRepository  # Порт!

class CreateOrderUseCase:
    def __init__(self, repo: OrderRepository):  # ✅ Принимает интерфейс
        self.repo = repo
```

```
❌ НЕПРАВИЛЬНО:                          ✅ ПРАВИЛЬНО:

┌────────────┐                           ┌────────────┐
│    CORE    │                           │    CORE    │
│            │                           │            │
│  UseCase ──┼──► PostgresRepo           │  UseCase ──┼──► PORT (interface)
│            │    (adapter)              │            │         ▲
└────────────┘                           └────────────┘         │
                                                                │
     Ядро зависит                              ┌────────────────┘
     от адаптера!                              │
                                         ┌─────┴─────┐
                                         │PostgresRepo│
                                         │ (adapter)  │
                                         └───────────┘
                                         Адаптер зависит от порта!
```

---

### Ошибка 2: Адаптер содержит бизнес-логику

```python
# ❌ ПЛОХО — бизнес-логика в адаптере
# adapters/inbound/rest/order_controller.py

class OrderController:
    def create_order(self, request):
        data = request.json
        
        # ❌ Бизнес-логика в контроллере!
        if data['total'] < 500:
            return {"error": "Минимальная сумма 500"}, 400
        
        order = Order(data)
        self.repo.save(order)
        return {"id": order.id}
```

```python
# ✅ ХОРОШО — адаптер только преобразует данные
# adapters/inbound/rest/order_controller.py

class OrderController:
    def __init__(self, create_order_use_case):
        self.use_case = create_order_use_case
    
    def create_order(self, request):
        dto = CreateOrderDTO.from_json(request.json)  # Преобразование
        order = self.use_case.execute(dto)            # Делегирование ядру
        return {"id": order.id}

# Бизнес-логика в ЯДРЕ:
# core/domain/order.py
class Order:
    def __init__(self, total):
        if total < 500:
            raise ValueError("Минимальная сумма 500")  # ✅ Здесь!
```

---

### Ошибка 3: Порт знает про детали реализации

```python
# ❌ ПЛОХО — порт знает про SQL
# core/ports/outbound/order_repository.py

class OrderRepository(ABC):
    @abstractmethod
    def execute_sql(self, query: str):  # ❌ SQL — это деталь реализации!
        pass
```

```python
# ✅ ХОРОШО — порт описывает ЧТО, а не КАК
# core/ports/outbound/order_repository.py

class OrderRepository(ABC):
    @abstractmethod
    def save(self, order: Order) -> None:  # ✅ Абстрактно
        pass
    
    @abstractmethod
    def find_by_id(self, order_id: str) -> Order:  # ✅ Абстрактно
        pass
```

---

## 🎭 "Боль" гексагональной архитектуры: Mapping (Дублирование)

Новички часто спрашивают: **"Зачем мне 3 класса Order? Это же дублирование кода!"**

Действительно, у тебя будут:
1. `OrderDTO` (в Presentation слое) — простой объект для JSON.
2. `Order` (в Domain слое) — богатая модель с поведением и проверками.
3. `OrderModel` (в Infrastructure слое) — объект для базы данных (SQLAlchemy/TypeORM).

**Зачем страдать?**

```
           DTO                   DOMAIN ENTITY                  DB MODEL
      (Presentation)              (Core/Domain)              (Infrastructure)
    ┌────────────────┐          ┌───────────────┐          ┌──────────────────┐
    │  OrderDTO      │          │    Order      │          │   OrderModel     │
    │                │  map()   │               │  map()   │                  │
    │  - user_id     │ ───────► │  - customer   │ ───────► │  - id (PK)       │
    │  - items_ids   │          │  - items      │          │  - customer_id   │
    │                │          │               │          │  - created_at    │
    │ (Просто данные)│          │ (Бизнес-логика│          │ (Типы данных БД, │
    │                │          │  проверки)    │          │  Foreign Keys)   │
    └────────────────┘          └───────────────┘          └──────────────────┘
```

**Ответ:** Они меняются по разным причинам!
- Если API изменится (клиент хочет слать XML) — меняем DTO. Domain не трогаем.
- Если бизнес-правила изменятся — меняем Domain. БД не трогаем.
- Если сменим Postgres на Mongo — меняем DB Model. Domain не трогаем.

**Правило:** Лучше написать маппер (конвертер), чем смешать всё в одну кучу и потом страдать при рефакторинге.

---

## ✨ Магия тестирования (Почему это круто)

Самый кайф Hexagonal Architecture — ты можешь протестировать всю бизнес-логику **без базы данных** и **без поднятия сервера**.

Пример теста (Pytest):

```python
# tests/test_create_order.py

class InMemoryOrderRepository(OrderRepository):
    """Фейковая база данных (просто список в памяти)"""
    def __init__(self):
        self.orders = []
        
    def save(self, order):
        self.orders.append(order)
        print("Сохранено в память (Fake DB)")

def test_create_order_success():
    # 1. Подготовка (Arrange)
    fake_repo = InMemoryOrderRepository()  # Используем фейк!
    fake_email = FakeEmailService()
    use_case = CreateOrderUseCase(fake_repo, fake_email)
    
    dto = CreateOrderDTO(items=["book"], total=600)
    
    # 2. Действие (Act)
    order = use_case.execute(dto)
    
    # 3. Проверка (Assert)
    assert order.total == 600
    assert len(fake_repo.orders) == 1  # Проверяем, что сохранилось в фейк
    assert fake_repo.orders[0].status == "new"

def test_cannot_create_order_under_500():
    fake_repo = InMemoryOrderRepository()
    use_case = CreateOrderUseCase(fake_repo, FakeEmailService())
    
    dto = CreateOrderDTO(items=["pen"], total=100)  # Мало денег!
    
    # Проверяем, что бизнес-логика выбросила ошибку
    with pytest.raises(ValueError, match="Минимальная сумма 500"):
        use_case.execute(dto)
```

**Результат:** Тесты бегут за 0.001 секунды, потому что не ходим в реальную базу! 🚀

---

## 🆚 Сравнение с Layered Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           LAYERED                                           │
│                                                                             │
│    ┌──────────────────┐                                                     │
│    │        UI        │     Зависимости идут ВНИЗ                           │
│    ├──────────────────┤     UI → App → Domain → Infra                       │
│    │   Application    │                                                     │
│    ├──────────────────┤     Domain МОЖЕТ зависеть от Infrastructure         │
│    │      Domain      │                                                     │
│    ├──────────────────┤                                                     │
│    │  Infrastructure  │                                                     │
│    └──────────────────┘                                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                          HEXAGONAL                                          │
│                                                                             │
│       REST ◄────┐                      ┌────► PostgreSQL                    │
│                 │                      │                                    │
│       CLI ◄─────┼── PORTS ── CORE ── PORTS ──┼────► Redis                   │
│                 │                      │                                    │
│       gRPC ◄────┘                      └────► Stripe                        │
│                                                                             │
│    Зависимости идут К ЦЕНТРУ                                                │
│    Ядро НЕ ЗАВИСИТ ни от чего                                               │
│    Адаптеры зависят от портов                                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📋 Чек-лист: Когда использовать Hexagonal?

- [ ] Сложная бизнес-логика, которую нужно тестировать изолированно?
- [ ] Несколько способов взаимодействия (REST + CLI + Events)?
- [ ] Возможна смена БД или внешних сервисов в будущем?
- [ ] Команда больше 2-3 человек?
- [ ] Долгоживущий проект (> 1 года)?

**Если "да" на 3+ пункта → Hexagonal оправдан.**

**Если проект простой** (CRUD, мало бизнес-логики) → используй Layered, не усложняй.

---

## 🎓 Главное для Junior:

1. **Ядро (Core)** = Domain + Application Services. Не зависит ни от чего.
2. **Порты (Ports)** = Интерфейсы. Ядро говорит "мне нужен кто-то, кто умеет X".
3. **Адаптеры (Adapters)** = Реализации портов. "Я умею X через PostgreSQL/Redis/HTTP".
4. **Входящие (Primary)** = Кто вызывает ядро (REST, CLI, gRPC).
5. **Исходящие (Secondary)** = Что использует ядро (БД, Email, Payment).
6. **Зависимости к центру** = Адаптеры зависят от портов, не наоборот.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Запомни главное правило:                                      │
│                                                                 │
│   ЯДРО никогда не импортирует АДАПТЕРЫ.                         │
│   ЯДРО определяет ПОРТЫ (интерфейсы).                           │
│   АДАПТЕРЫ реализуют ПОРТЫ.                                     │
│                                                                 │
│   Если видишь import из adapters/ в core/ → это ошибка!         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Аналогия напоследок:** Ядро — это розетка 🔌 (стандарт). Адаптеры — это вилки разных устройств. Розетка не знает, что в неё воткнут (чайник, компьютер, телефон). Она просто предоставляет интерфейс (порт). Устройства (адаптеры) подстраиваются под розетку, а не наоборот.
