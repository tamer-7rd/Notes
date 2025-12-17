# Dependency Injection (DI) — для Junior

## 🎯 Что это простыми словами?

**Dependency Injection (DI)** = "Внедрение зависимостей".

**Идея:** Объект не создаёт то, от чего зависит, а получает это извне (через конструктор, параметры или сеттер).

**Аналогия:** 
- **Без DI:** Ты сам идёшь в магазин за продуктами каждый раз, когда готовишь.
- **С DI:** У тебя есть поставщик (или холодильник), который уже всё принёс. Ты готовишь, не думая, откуда продукты.

---

## ❌ Проблема: Жёсткая связанность (Без DI)

Допустим, у тебя есть сервис, который работает с базой данных:

```python
class UserService:
    def __init__(self):
        self.db = Database()  # ❌ Зависимость создаётся внутри!

    def get_user(self, user_id):
        return self.db.query(f"SELECT * FROM users WHERE id={user_id}")
```

**Что не так?**
- Ты жёстко привязан к конкретному `Database`.
- Хочешь заменить базу на `FakeDatabase` для тестов? Придётся менять код внутри класса.
- Хочешь завтра перейти с PostgreSQL на MongoDB? Опять менять `UserService`.
- Это называется **жёсткая связанность** (tight coupling).

---

## ✅ Решение: Dependency Injection

DI говорит: "Не создавай зависимости внутри, получай их снаружи".

```python
class UserService:
    def __init__(self, db):  # ✅ Зависимость приходит извне
        self.db = db

    def get_user(self, user_id):
        return self.db.query(f"SELECT * FROM users WHERE id={user_id}")
```

**Теперь:**
```python
# В продакшене
db = Database()
user_service = UserService(db)

# В тестах
fake_db = FakeDatabase()
user_service = UserService(fake_db)  # ✅ Легко подменить!

# Завтра переходим на MongoDB
mongo_db = MongoDB()
user_service = UserService(mongo_db)  # ✅ Код UserService не меняется!
```

**Преимущество:** `UserService` теперь не знает, какая именно база внутри. Он работает через абстракцию (контракт: у объекта есть метод `query`).

---

## 🔍 Почему это важно?

### 1. Тестируемость
Можно подставить фейковые зависимости (mock, stub) для тестов:
```python
class FakeDatabase:
    def query(self, sql):
        return {"id": 1, "name": "Test User"}

fake_db = FakeDatabase()
service = UserService(fake_db)
# Теперь можно тестировать без реальной базы!
```

### 2. Расширяемость
Легко заменить реализацию, не трогая бизнес-логику:
```python
# Сегодня PostgreSQL
service = UserService(PostgreSQLDatabase())

# Завтра MongoDB
service = UserService(MongoDBDatabase())
```

### 3. Слабая связанность
Классы меньше знают друг о друге → код живёт дольше и проще поддерживать.

### 4. Конфигурируемость
В бою одно, в тестах другое, в отладке третье:
```python
if ENV == "production":
    db = PostgreSQLDatabase()
elif ENV == "test":
    db = FakeDatabase()
else:
    db = InMemoryDatabase()
```

---

## 🛠️ Варианты внедрения зависимостей

### 1. Через конструктор (Constructor Injection) — РЕКОМЕНДУЕТСЯ
Самый популярный способ:
```python
class UserService:
    def __init__(self, db):  # ← Зависимость в конструкторе
        self.db = db
```

### 2. Через сеттеры (Setter Injection)
Передаёшь зависимость методом после создания объекта:
```python
class UserService:
    def __init__(self):
        self.db = None
    
    def set_db(self, db):  # ← Зависимость через сеттер
        self.db = db

# Использование
user_service = UserService()
user_service.set_db(Database())
```

### 3. Через параметры методов (Method Injection)
Для мелких зависимостей:
```python
def get_user(self, db, user_id):  # ← Зависимость как параметр
    return db.query("...")
```

### 4. Через контейнер (IoC Container)
Это когда есть "коробка", которая знает, как строить все зависимости, и сама их прокидывает:
```python
# Псевдокод (в реальности это делают фреймворки)
container = Container()
container.register(Database, PostgreSQLDatabase)
container.register(UserService)

service = container.get(UserService)  # Контейнер сам подставит Database
```

**Это уже уровень фреймворков:** NestJS, Spring, Angular делают это автоматически.

---

## 🔗 Связь с DIP (Dependency Inversion Principle)

**DI** = практический механизм, чтобы воплотить **DIP** (принцип из SOLID).

### Визуальная схема связи:

```
DIP (Принцип)
    ↓
"Завись от абстракций, а не от деталей"
    ↓
DI (Инструмент)
    ↓
"Передавай зависимости через конструктор"
    ↓
Результат: Гибкий, тестируемый код
```

### Пример связи:

**Без DIP и DI (нарушение):**
```python
class UserService:
    def __init__(self):
        self.db = PostgreSQLDatabase()  # ❌ Зависит от конкретной реализации
```
**Проблема:** Нарушает DIP (зависит от деталей, а не от абстракции).

**С DIP и DI (правильно):**
```python
# 1. Создаём АБСТРАКЦИЮ (DIP требует)
class Database:
    def query(self, sql):
        raise NotImplementedError

# 2. Конкретные реализации
class PostgreSQLDatabase(Database):
    def query(self, sql):
        pass

# 3. Используем DI (передаём через конструктор)
class UserService:
    def __init__(self, db: Database):  # ✅ Зависит от абстракции (DIP)
        self.db = db  # ✅ Получаем извне (DI)
```

**Теперь:**
- **DIP соблюдён:** `UserService` зависит от абстракции (`Database`), а не от детали (`PostgreSQLDatabase`).
- **DI применён:** Зависимость передаётся извне через конструктор.

**Итог:** DI — это способ реализовать DIP на практике. Без DI сложно соблюдать DIP, потому что зависимости будут создаваться внутри классов и привязываться к конкретным реализациям.

**Аналогия:**
- **DIP** = "Используй стандартные розетки" (принцип).
- **DI** = "Вставляй вилку в розетку снаружи" (способ).

Оба работают вместе для создания гибкого кода.

---

## 📋 Чек-лист: Когда использовать DI?

- [ ] Класс зависит от внешних ресурсов (БД, API, файловая система).
- [ ] Нужно тестировать без реальных зависимостей (mock/stub).
- [ ] Зависимость может меняться (PostgreSQL → MongoDB).
- [ ] Хочешь слабую связанность между классами.

**Если все "да" → используй DI через конструктор.**

---

## 🎓 Главное для Junior:

1. **DI** = передавать зависимости извне, а не создавать внутри класса.
2. **Самый простой способ** = через конструктор (`__init__(self, dependency)`).
3. **Зачем:** Чтобы код был тестируемым, расширяемым и гибким.
4. **DI помогает соблюдать DIP** (принцип инверсии зависимостей из SOLID).

**Запомни:** Если видишь `self.db = Database()` внутри класса → это нарушение DI. Передавай `db` через конструктор!
