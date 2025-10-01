# Конспект: Фабричный метод (Factory Method)

## 1. Определение

Фабричный метод — порождающий паттерн проектирования, который:
- определяет интерфейс для создания объектов;
- делегирует выбор конкретного продукта подклассам или фабрике;
- отделяет код создания объектов от кода, который их использует.

**Идея простая:** вместо того чтобы писать new Client() в разных местах, мы выделяем «завод» (factory), который знает, как именно создать объект. Если завтра поменяются правила — меняем только фабрику, а не весь код.

---

## 2. Проблема без фабрики

### Пример с API-клиентами (Python)
```python
class GitHubClient:
    def __init__(self, token):
        self.token = token
    def get_user(self, username):
        return f"Fetching {username} from GitHub with token {self.token}"

class TwitterClient:
    def __init__(self, token):
        self.token = token
    def get_user(self, username):
        return f"Fetching {username} from Twitter with token {self.token}"

# Использование
if service == "github":
    client = GitHubClient("TOKEN123")
elif service == "twitter":
    client = TwitterClient("TOKEN456")
```

**Минус:** при добавлении новых клиентов (RedditClient, YouTubeClient) нужно менять логику во многих местах кода.

---

## 3. Решение через фабрику (Простая фабрика)

```python
class ClientFactory:
    def create_client(self, service):
        if service == "github":
            return GitHubClient("TOKEN123")
        elif service == "twitter":
            return TwitterClient("TOKEN456")
        else:
            raise ValueError("Unknown service")
```

**Использование:**
```python
factory = ClientFactory()
client = factory.create_client("github")
print(client.get_user("tamerlan"))
```

Теперь при добавлении RedditClient нужно править только фабрику.

---

## 4. Пример с доменной моделью

```python
class OnlineOrder: ...
class OfflineOrder: ...
class SubscriptionOrder: ...

class OrderFactory:
    def create_order(self, type):
        if type == "online":
            return OnlineOrder()
        elif type == "offline":
            return OfflineOrder()
        elif type == "subscription":
            return SubscriptionOrder()
```

Бизнес-код теперь думает только о том, **какой тип заказа нужен**, а не о том, **как именно он создаётся**.

---

## 5. Отличие от простой фабрики
- **Простая фабрика**: вся логика выбора в одном классе (или функции) через if/switch.
- **Factory Method по GoF**: фабричный метод абстрактного класса, который переопределяется в подклассах.

Оба варианта изолируют создание объектов, но «классический» паттерн строится на наследовании.

---

## 6. Классический Factory Method (GoF)

Пример (Python):

```python
# Продукты
class Product:
    def get_name(self): pass

class ConcreteProductA(Product):
    def get_name(self): return "ProductA"

class ConcreteProductB(Product):
    def get_name(self): return "ProductB"

# Создатели
class Creator:
    def factory_method(self) -> Product:
        pass

class ConcreteCreatorA(Creator):
    def factory_method(self) -> Product:
        return ConcreteProductA()

class ConcreteCreatorB(Creator):
    def factory_method(self) -> Product:
        return ConcreteProductB()

# Клиентский код
creators = [ConcreteCreatorA(), ConcreteCreatorB()]
for c in creators:
    product = c.factory_method()
    print(product.get_name())
```

**Суть GoF-версии:**
- есть абстрактный создатель (Creator) с методом factory_method,
- каждый подкласс (ConcreteCreatorA/B) сам решает, какой продукт создавать,
- клиентский код работает только с абстракциями.

---

## 7. Применение в жизни
- **API-клиенты**: GitHubClient, TwitterClient, RedditClient.
- **Финтех**: разные платёжные провайдеры (Visa, Stripe, PayPal).
- **Соцсети**: интеграции с Facebook, LinkedIn и др.
- **Микросервисы**: AuthService, BillingService, NotificationService.

---

## 8. Связь с DIP и DI-фреймворками
- Фабрика = ручная инкапсуляция логики создания.
- DI (Dependency Injection) = получение зависимостей «снаружи».
- DI-фреймворки (Spring, Angular, NestJS) = автоматизированные фабрики, которые сами выбирают и внедряют зависимости.

---

## 9. Главное
- **Простая фабрика** → практичный вариант с if.
- **GoF Factory Method** → строгий вариант через наследование.
- Клиент = код, который использует фабрику и продукты (не человек).
- Плюсы: меньше дублирования, гибкость при смене реализаций, чистая бизнес-логика.

---

## 10. Схема Factory Method

```
        [ Creator ]
     -----------------
     + factoryMethod()   <--- абстрактный метод (обещание "я умею создавать продукт")

          / \
           |
   -----------------
   |               |
[ConcreteCreatorA] [ConcreteCreatorB]
  factoryMethod()    factoryMethod()
     |                   |
     v                   v
[ConcreteProductA]   [ConcreteProductB]
```
