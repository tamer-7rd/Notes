# Factory Method (Фабричный метод) — для Junior

## 🎯 Что это простыми словами?

**Factory Method** = паттерн, который создаёт объекты через специальный "завод" (фабрику), а не напрямую через `new Class()`.

**Идея:** Вместо того чтобы писать `if service == "github": client = GitHubClient()` в разных местах, мы создаём фабрику, которая знает, как создать нужный объект. Если завтра появятся новые типы — меняем только фабрику, а не весь код.

**Аналогия:** Как фабрика по производству машин. Ты говоришь "хочу седан", а фабрика сама знает, как его собрать. Тебе не нужно знать детали производства.

---

## ❌ Проблема: Дублирование логики создания

Представь, у тебя есть несколько API-клиентов, и ты создаёшь их в разных местах:

```python
class GitHubClient:
    def __init__(self, token):
        self.token = token
    def get_user(self, username):
        return f"Fetching {username} from GitHub"

class TwitterClient:
    def __init__(self, token):
        self.token = token
    def get_user(self, username):
        return f"Fetching {username} from Twitter"

# Проблема: логика создания размазана по коду
if service == "github":
    client = GitHubClient("TOKEN123")
elif service == "twitter":
    client = TwitterClient("TOKEN456")

# В другом месте кода — опять та же логика!
if platform == "github":
    api = GitHubClient("TOKEN123")
elif platform == "twitter":
    api = TwitterClient("TOKEN456")
```

**Проблемы:**
- Логика создания дублируется в разных местах.
- При добавлении нового клиента (RedditClient) нужно править код везде.
- Если изменится способ создания (нужен другой токен) — править во всех местах.

---

## ✅ Решение: Простая фабрика

Выносим логику создания в один класс:

```python
class ClientFactory:
    def create_client(self, service):
        if service == "github":
            return GitHubClient("TOKEN123")
        elif service == "twitter":
            return TwitterClient("TOKEN456")
        elif service == "reddit":
            return RedditClient("TOKEN789")
        else:
            raise ValueError(f"Unknown service: {service}")

# Использование везде одинаковое
factory = ClientFactory()
client = factory.create_client("github")
print(client.get_user("tamerlan"))
```

**Преимущества:**
- Логика создания в одном месте.
- Добавить новый клиент → правим только фабрику.
- Изменить способ создания → правим только фабрику.

---

## 🏭 Два вида фабрик

### 1. Простая фабрика (Simple Factory) — РЕКОМЕНДУЕТСЯ для Junior

Вся логика выбора в одном классе через `if/elif`:

```python
class PaymentFactory:
    def create_payment(self, method):
        if method == "paypal":
            return PayPalPayment()
        elif method == "stripe":
            return StripePayment()
        elif method == "crypto":
            return CryptoPayment()
        else:
            raise ValueError("Unknown payment method")
```

**Когда использовать:** В 90% случаев этого достаточно. Просто, понятно, работает.

---

### 2. Factory Method (GoF) — для сложных случаев (Что бы не нарушать open/close principle "O in SOLID")

Абстрактный класс с методом, который переопределяется в подклассах:

```python
from abc import ABC, abstractmethod

# Абстрактный создатель
class PaymentCreator(ABC):
    @abstractmethod
    def create_payment(self):
        pass
    
    def process_payment(self, amount):
        payment = self.create_payment()  # Вызываем фабричный метод
        return payment.pay(amount)

# Конкретные создатели
class PayPalCreator(PaymentCreator):
    def create_payment(self):
        return PayPalPayment()

class StripeCreator(PaymentCreator):
    def create_payment(self):
        return StripePayment()

# Использование
paypal_creator = PayPalCreator()
paypal_creator.process_payment(100)
```

**Когда использовать:** Когда нужна дополнительная логика вокруг создания (например, `process_payment` делает что-то до и после создания).

---

## 🔍 Живой пример: Заказы в интернет-магазине

### ❌ Плохо (Без фабрики):

```python
class OnlineOrder:
    def __init__(self):
        self.type = "online"
        self.shipping = "courier"

class OfflineOrder:
    def __init__(self):
        self.type = "offline"
        self.shipping = "pickup"

class SubscriptionOrder:
    def __init__(self):
        self.type = "subscription"
        self.shipping = "digital"

# Логика создания размазана
def create_order(order_type):
    if order_type == "online":
        return OnlineOrder()
    elif order_type == "offline":
        return OfflineOrder()
    elif order_type == "subscription":
        return SubscriptionOrder()

# В другом месте — опять та же логика!
def process_order(order_type):
    if order_type == "online":
        order = OnlineOrder()
    elif order_type == "offline":
        order = OfflineOrder()
    # ...
```

### ✅ Хорошо (С фабрикой):

```python
class OrderFactory:
    def create_order(self, order_type):
        if order_type == "online":
            return OnlineOrder()
        elif order_type == "offline":
            return OfflineOrder()
        elif order_type == "subscription":
            return SubscriptionOrder()
        else:
            raise ValueError(f"Unknown order type: {order_type}")

# Использование везде одинаковое
factory = OrderFactory()
order = factory.create_order("online")
```

---

## 🎯 Когда использовать Factory?

**Используй фабрику, если:**

- ✅ Нужно создавать объекты разных типов в зависимости от параметра.
- ✅ Логика создания сложная (много `if/elif`).
- ✅ Одна и та же логика создания повторяется в разных местах.
- ✅ В будущем могут появиться новые типы объектов.

**Не нужна фабрика, если:**

- ❌ Создаёшь объекты одного типа (просто `User()`).
- ❌ Логика создания простая (1-2 варианта, которые не меняются).
- ❌ Создание происходит в одном месте и не дублируется.

---

## 🔗 Связь с DIP и DI

**Factory Method помогает соблюдать DIP:**

```python
# Без фабрики (нарушение DIP)
class OrderService:
    def process(self, order_type):
        if order_type == "online":
            order = OnlineOrder()  # ❌ Зависит от конкретного класса
        # ...

# С фабрикой (соблюдение DIP)
class OrderService:
    def __init__(self, factory: OrderFactory):  # ✅ Зависит от абстракции
        self.factory = factory
    
    def process(self, order_type):
        order = self.factory.create_order(order_type)  # ✅ Работает через интерфейс
```

**DI-фреймворки (Spring, NestJS, Angular)** — это автоматизированные фабрики:
- Они сами знают, как создать объекты.
- Автоматически внедряют зависимости.
- Ты просто говоришь "мне нужен UserService", а фреймворк сам создаёт его со всеми зависимостями.

---

## 📋 Чек-лист: Когда использовать Factory?

- [ ] Нужно создавать объекты разных типов в зависимости от параметра?
- [ ] Логика создания повторяется в разных местах кода?
- [ ] В будущем могут появиться новые типы объектов?
- [ ] Логика создания сложная (много условий)?

**Если "да" хотя бы на 2 пункта → используй фабрику.**

---

## 🎓 Главное для Junior:

1. **Простая фабрика** (с `if/elif`) — используй в 90% случаев. Это достаточно.
2. **Factory Method (GoF)** — только если нужна дополнительная логика вокруг создания.
3. **Фабрика изолирует создание** — меняешь только фабрику, остальной код не трогаешь.
4. **Фабрика помогает соблюдать DIP** — код зависит от интерфейса фабрики, а не от конкретных классов.

**Запомни:** Если видишь длинный `if/elif` для создания объектов в разных местах → вынеси в фабрику!
