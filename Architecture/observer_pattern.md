# Observer Pattern (Паттерн Наблюдатель) — для Junior

## 🎯 Что это простыми словами?

**Observer Pattern** = паттерн, где один объект (Издатель) уведомляет множество других объектов (Наблюдателей) об изменениях.

**Идея:** Когда что-то происходит (событие), все, кто подписался на это событие, автоматически получают уведомление и могут отреагировать.

**Аналогия:** Как подписка на YouTube. Когда канал публикует видео (событие), все подписчики (наблюдатели) автоматически получают уведомление. Канал не знает, кто именно подписан — он просто объявляет о новом видео.

---

## ❌ Проблема: Жёсткая связанность (Без Observer)

Представь, у тебя есть система регистрации пользователей. При регистрации нужно:
1. Отправить email-письмо.
2. Записать в лог.
3. Обновить статистику.

**Плохой вариант (жёсткая связность):**

```python
class UserService:
    def __init__(self):
        self.emailer = EmailSender()
        self.logger = Logger()
        self.analytics = Analytics()
    
    def register_user(self, user):
        # Сохраняем пользователя
        self.save_user(user)
        
        # ❌ Проблема: жёстко привязаны к конкретным классам
        self.emailer.send_welcome_email(user)
        self.logger.log(f"User {user.name} registered")
        self.analytics.track_registration(user)
```

**Проблемы:**
- При добавлении нового действия (например, SMS) нужно менять `UserService`.
- Код размазан — логика регистрации смешана с уведомлениями.
- Сложно тестировать — нужно мокать все зависимости.

---

## ✅ Решение: Observer Pattern

Издатель (Subject) объявляет событие, а наблюдатели (Observers) сами решают, что делать.

### Структура:

```
+---------------------+
|  Subject (Издатель) |
+----------+----------+
           |
           | notify()
           |
     +-----+-----+---------+
     |           |         |
     v           v         v
+----------+ +--------+ +-----------+
| Observer1| |Observer2| | Observer3 |
| (Email)  | | (Logger)| | (Analytics)|
+----------+ +--------+ +-----------+
```

---

## 🔧 Как это работает?

### 1. Создаём интерфейс Observer:

```python
from abc import ABC, abstractmethod

class Observer(ABC):
    @abstractmethod
    def update(self, event_data):
        pass
```

### 2. Создаём конкретных наблюдателей:

```python
class EmailNotifier(Observer):
    def update(self, event_data):
        user = event_data['user']
        print(f"📧 Отправляю email пользователю {user['name']}")

class Logger(Observer):
    def update(self, event_data):
        user = event_data['user']
        print(f"📝 Логирую: пользователь {user['name']} зарегистрирован")

class Analytics(Observer):
    def update(self, event_data):
        user = event_data['user']
        print(f"📊 Обновляю статистику для {user['name']}")
```

### 3. Создаём Subject (Издатель):

```python
class UserService:
    def __init__(self):
        self._observers = []  # Список подписчиков
    
    def add_observer(self, observer: Observer):
        self._observers.append(observer)
    
    def remove_observer(self, observer: Observer):
        self._observers.remove(observer)
    
    def notify_observers(self, event_data):
        # Уведомляем всех подписчиков
        for observer in self._observers:
            observer.update(event_data)
    
    def register_user(self, user):
        # Сохраняем пользователя
        self.save_user(user)
        
        # ✅ Уведомляем всех наблюдателей
        self.notify_observers({'event': 'user_registered', 'user': user})
```

### 4. Использование:

```python
# Создаём сервис
user_service = UserService()

# Подписываем наблюдателей
user_service.add_observer(EmailNotifier())
user_service.add_observer(Logger())
user_service.add_observer(Analytics())

# Регистрируем пользователя
user_service.register_user({'name': 'Tamerlan', 'email': 'tamerlan@example.com'})

# Вывод:
# 📧 Отправляю email пользователю Tamerlan
# 📝 Логирую: пользователь Tamerlan зарегистрирован
# 📊 Обновляю статистику для Tamerlan
```

**Преимущества:**
- Добавить новое действие → просто создаёшь новый Observer и подписываешь.
- Код `UserService` не меняется при добавлении новых наблюдателей.
- Легко тестировать — можно подставить фейковых наблюдателей.

---

## 🎯 Когда использовать Observer?

**Используй Observer, если:**

- ✅ Одно событие должно вызывать несколько разных действий.
- ✅ Нужно разделить бизнес-логику (регистрация отдельно от уведомлений).
- ✅ Количество наблюдателей может меняться во время работы программы.
- ✅ Хочешь слабую связанность между компонентами.

**Не нужен Observer, если:**

- ❌ Простые отношения (один объект вызывает один метод).
- ❌ Фиксированное количество действий, которые не меняются.
- ❌ Порядок выполнения критичен (Observer не гарантирует порядок).

---

## 🔗 Связь с SOLID (OCP)

**Observer помогает соблюдать OCP (Open/Closed Principle):**

```python
# Без Observer (нарушение OCP)
class UserService:
    def register_user(self, user):
        self.save_user(user)
        self.emailer.send()  # ❌ При добавлении SMS нужно менять код
        self.logger.log()

# С Observer (соблюдение OCP)
class UserService:
    def register_user(self, user):
        self.save_user(user)
        self.notify_observers(...)  # ✅ Добавляем нового Observer — код не меняется
```

**Суть:** Чтобы добавить новую реакцию на событие, ты расширяешь систему (добавляешь новый Observer), а не модифицируешь существующий код.

---

## 💡 Примеры из реальной жизни

### 1. JavaScript Event Listeners:

```javascript
// Издатель (DOM элемент)
const button = document.getElementById('myButton');

// Наблюдатели (обработчики событий)
button.addEventListener('click', () => console.log('Клик 1'));
button.addEventListener('click', () => console.log('Клик 2'));

// При клике все наблюдатели получают уведомление
```

### 2. Node.js EventEmitter:

```javascript
const EventEmitter = require('events');

class UserService extends EventEmitter {
    registerUser(user) {
        // Сохраняем пользователя
        this.saveUser(user);
        
        // Уведомляем всех подписчиков
        this.emit('user_registered', user);
    }
}

const userService = new UserService();

// Подписываемся на событие
userService.on('user_registered', (user) => {
    console.log(`Email отправлен ${user.name}`);
});

userService.on('user_registered', (user) => {
    console.log(`Лог записан для ${user.name}`);
});
```

### 3. Django Signals (Python):

```python
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=User)
def send_welcome_email(sender, instance, **kwargs):
    # Автоматически вызывается при сохранении User
    send_email(instance)
```

---

## 📋 Чек-лист: Когда использовать Observer?

- [ ] Одно событие должно вызывать несколько разных действий?
- [ ] Количество действий может меняться во время работы программы?
- [ ] Нужно разделить бизнес-логику (событие отдельно от реакций)?
- [ ] Хочешь слабую связанность между компонентами?

**Если "да" хотя бы на 2 пункта → используй Observer.**

---

## 🎓 Главное для Junior:

1. **Observer** = один объект уведомляет множество других об изменениях.
2. **Subject (Издатель)** = тот, кто объявляет событие.
3. **Observer (Наблюдатель)** = тот, кто реагирует на событие.
4. **Observer помогает соблюдать OCP** — добавляешь новых наблюдателей без изменения кода издателя.
5. **В JavaScript/Node.js** Observer уже встроен (`addEventListener`, `EventEmitter`).

**Запомни:** Если видишь, что одно событие вызывает несколько разных действий, и они размазаны по коду → используй Observer Pattern!
