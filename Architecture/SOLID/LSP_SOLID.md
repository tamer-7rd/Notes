# LSP (Liskov Substitution Principle) — для Junior (принцип подстановки Лискова) 

> "Если S является подтипом T, то объекты типа T могут быть заменены объектами типа S без изменения желаемых свойств программы."

## 🎯 Что это значит простыми словами?

**"Наследники должны работать так же, как родительский класс"**

Перевод: если у тебя есть функция, которая работает с базовым классом, она должна работать и с любым наследником **без изменений**. Наследник не должен ломать то, что обещал родитель.

---

## ❌ Проблема: Что происходит, когда нарушаешь LSP?

### Пример 1: Пингвин не может летать

```python
# Базовый класс (T)
class Bird:
    def fly(self):
        print("Я лечу!")

# Наследник (S)
class Penguin(Bird):
    def fly(self):
        raise RuntimeError("Я не летаю!")  # ❌ Ломает обещание родителя!

# Функция, которая работает с любой птицей (Программа)
def make_bird_fly(bird: Bird):
    bird.fly()  # Ожидает, что любая птица умеет летать

# Использование
sparrow = Bird() #(Обьект типа Т)
make_bird_fly(sparrow)  # ✅ Работает

penguin = Penguin() #(Обьект типа S)
make_bird_fly(penguin)  # 💥 ОШИБКА! Функция сломалась
```

**Проблема:** функция `make_bird_fly()` написана для работы с `Bird`, но она не работает с `Penguin`. Это нарушение LSP.

---

## ✅ Решение: Разделяй интерфейсы правильно

### Хороший код (соблюдение LSP):

```python
# Базовый класс (все птицы)
class Bird:
    def eat(self):
        print("Ням-ням")

# Интерфейс для летающих птиц
class FlyableBird(Bird):
    def fly(self):
        raise NotImplementedError

# Интерфейс для плавающих птиц
class SwimmableBird(Bird):
    def swim(self):
        raise NotImplementedError

# Конкретные реализации
class Sparrow(FlyableBird):
    def fly(self):
        print("Вжжж! Лечу!")

class Penguin(SwimmableBird):
    def swim(self):
        print("Плюх-плюх! Плыву!")

# Функции работают с правильными интерфейсами
def make_bird_fly(bird: FlyableBird):
    bird.fly()  # Работает только с летающими

def make_bird_swim(bird: SwimmableBird):
    bird.swim()  # Работает только с плавающими

# Использование
sparrow = Sparrow()
make_bird_fly(sparrow)  # ✅ Работает

penguin = Penguin()
make_bird_swim(penguin)  # ✅ Работает
# make_bird_fly(penguin)  # ❌ Ошибка компиляции - пингвин не FlyableBird
```

**Преимущество:** теперь нельзя случайно передать пингвина в функцию, которая ожидает летающую птицу. Ошибка видна сразу.

Этот принцип был создан, чтобы спасти программистов от неожиданных поломок в коде, который использует наследование.
Главная проблема, которую он решает:
Когда ты видишь функцию, которая принимает Bird, ты (как разработчик) уверен: "Ага, это птица, значит, она умеет летать. Я могу вызвать .fly() и не париться". Ты доверяешь типу.
Если кто-то создаст Penguin (наследника Bird), который при вызове .fly() взрывается ошибкой, он обманывает твои ожидания.
В итоге:
Ты пишешь код, который работает с Bird.
Коллега добавляет Penguin.
Твой код падает в продакшене, хотя ты его не трогал.
Тебе приходится лезть в свой код и писать костыли: if not isinstance(bird, Penguin): bird.fly().
**Принцип Лисков говорит:**
**"Ребята, если вы наследуетесь, будьте добры вести себя так, как обещал родитель. Не заставляйте других писать проверки if на каждый чих".**
По сути, это правило доверия: если назвался груздем (Bird), полезай в кузов (умей .fly()).

---

## 🔍 Пример 2: Платежи (реальный кейс)

### ❌ Плохо (нарушение LSP):

```python
class PaymentMethod:
    def pay(self, amount):
        raise NotImplementedError
    # Неявный контракт: pay() должен возвращать bool (True = успех, False = неудача)

class PayPal(PaymentMethod):
    def pay(self, amount):
        print(f"PayPal: списано {amount}")
        return True  # ✅ Соблюдает контракт: возвращает bool

class CryptoPayment(PaymentMethod):
    def pay(self, amount):
        # Крипта может вернуть None, если транзакция в процессе
        print(f"Crypto: транзакция создана, ждём подтверждения...")
        return None  # ❌ НАРУШЕНИЕ LSP! Возвращает None вместо bool

# Функция написана для работы с PaymentMethod
# Она ожидает, что pay() вернёт bool
def process_order(payment: PaymentMethod, amount):
    result = payment.pay(amount)
    
    # Функция ожидает True или False
    if result is True:
        print("✅ Оплата успешна!")
        return "success"
    elif result is False:
        print("❌ Оплата не прошла")
        return "failed"
    else:
        # Этого не должно происходить, если все наследники соблюдают контракт!
        print(f"⚠️ Неожиданный результат: {result}")
        return "unknown"

# Использование
paypal = PayPal()
status = process_order(paypal, 100)
# Вывод: ✅ Оплата успешна!
# status = "success" ✅ Работает как ожидалось

crypto = CryptoPayment()
status = process_order(crypto, 100)
# Вывод: ⚠️ Неожиданный результат: None
# status = "unknown" ❌ Функция не может правильно обработать CryptoPayment!
```

**Где нарушение LSP:**

1. **Контракт родителя:** `PaymentMethod.pay()` должен возвращать `bool` (`True` = успех, `False` = неудача).
2. **PayPal соблюдает контракт:** возвращает `True` ✅
3. **CryptoPayment нарушает контракт:** возвращает `None` вместо `bool` ❌

**Проблема:** Функция `process_order()` написана для работы с `PaymentMethod` и ожидает `bool`. Но `CryptoPayment` возвращает `None`, из-за чего функция не может правильно определить статус оплаты. Это нарушение LSP — наследник не может заменить родителя без изменения логики функции.

---

### ✅ Хорошо (соблюдение LSP):

```python
class PaymentMethod:
    def pay(self, amount):
        raise NotImplementedError

class PayPal(PaymentMethod):
    def pay(self, amount):
        print(f"PayPal: списано {amount}")
        return True

class CryptoPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Crypto: транзакция создана...")
        # Возвращаем True, даже если транзакция в процессе
        # Это соответствует контракту родителя
        return True

# Или разделяем на разные интерфейсы
class InstantPayment(PaymentMethod):
    def pay(self, amount):
        raise NotImplementedError

class AsyncPayment(PaymentMethod):
    def pay(self, amount):
        raise NotImplementedError
    def get_status(self):
        raise NotImplementedError  # Дополнительный метод для проверки статуса
```

**Преимущество:** все наследники соблюдают контракт родителя → функция работает с любым из них.

---

## 🚨 Пример 3: Неожиданные побочные эффекты

### ❌ Плохо (нарушение LSP):

```python
class FileReader:
    def read(self, filename):
        with open(filename, 'r') as f:
            return f.read()

class CachedFileReader(FileReader):
    def __init__(self):
        self.cache = {}
    
    def read(self, filename):
        # Добавляем кэш - это нормально
        if filename in self.cache:
            return self.cache[filename]
        
        # Но также очищаем старые файлы - это НЕ ожидалось!
        self._cleanup_old_files()  # ❌ Побочный эффект!
        
        content = super().read(filename)
        self.cache[filename] = content
        return content
    
    def _cleanup_old_files(self):
        print("Удаляю старые файлы...")  # Клиент этого не ждал!

# Функция, которая работает с любым FileReader
def read_config(reader: FileReader):
    return reader.read("config.txt")

# Использование
normal_reader = FileReader()
read_config(normal_reader)  # ✅ Просто читает файл

cached_reader = CachedFileReader()
read_config(cached_reader)  # 💥 Неожиданно удаляет файлы!
```

**Проблема:** наследник делает что-то, чего не делал родитель. Клиентский код не ожидал этого.

---

### ✅ Хорошо (соблюдение LSP):

```python
class FileReader:
    def read(self, filename):
        with open(filename, 'r') as f:
            return f.read()

class CachedFileReader(FileReader):
    def __init__(self):
        self.cache = {}
    
    def read(self, filename):
        # Только добавляем кэш, не меняем поведение
        if filename in self.cache:
            return self.cache[filename]
        
        content = super().read(filename)  # Вызываем родителя
        self.cache[filename] = content
        return content
        # Никаких побочных эффектов!

# Или выносим очистку в отдельный метод
class ManagedCachedFileReader(CachedFileReader):
    def cleanup(self):  # Отдельный метод для очистки
        self.cache.clear()
        print("Кэш очищен")
```

**Преимущество:** наследник только расширяет функционал (добавляет кэш), но не меняет основное поведение.

---

## 🎯 Как понять, что ты нарушаешь LSP?

**Сигналы проблемы:**

1. **Наследник выбрасывает ошибку там, где родитель работает нормально**
   ```python
   class Parent:
       def method(self): return 42
   
   class Child(Parent):
       def method(self): raise Error("Не работает!")  # ❌
   ```

2. **Наследник возвращает другой тип данных**
   ```python
   class Parent:
       def get_data(self): return {"key": "value"}
   
   class Child(Parent):
       def get_data(self): return "строка"  # ❌ Ожидали словарь
   ```

3. **Наследник делает что-то дополнительное, чего не делал родитель**
   ```python
   class Parent:
       def save(self): print("Сохранено")
   
   class Child(Parent):
       def save(self):
           super().save()
           self.delete_old_data()  # ❌ Неожиданный побочный эффект
   ```

---

## ✅ Правило для Junior:

**Если функция работает с родительским классом, она должна работать и с наследником БЕЗ ИЗМЕНЕНИЙ.**

```python
# Эта функция написана для работы с PaymentMethod
def process_payment(payment: PaymentMethod, amount):
    result = payment.pay(amount)
    return result

# Если ты создаёшь новый класс PayPal(PaymentMethod)
# Функция должна работать с ним БЕЗ ИЗМЕНЕНИЙ
paypal = PayPal()
process_payment(paypal, 100)  # Должно работать!

# Если не работает → ты нарушил LSP
```

---

## 🎓 Главное для Junior:

1. **Наследник должен делать то же самое, что родитель** (может делать больше, но не меньше).
2. **Наследник не должен выбрасывать ошибки там, где родитель работает нормально**.
3. **Наследник не должен менять тип возвращаемых данных**.
4. **Наследник не должен добавлять неожиданные побочные эффекты**.
5. **Если поведение сильно отличается** → раздели на разные интерфейсы (как `FlyableBird` и `SwimmableBird`).

**Помни:** LSP защищает тебя от ситуаций, когда код работает с родителем, но ломается с наследником. Если такое происходит → иерархия классов построена неправильно.

---

## 🔗 Связь с другими принципами:

- **OCP:** LSP помогает соблюдать OCP. Если наследники взаимозаменяемы, ты можешь добавлять новые классы, не меняя старый код.
- **ISP:** Если интерфейс слишком широкий (как `Bird` с методом `fly()`), лучше разделить его на более узкие (`FlyableBird`, `SwimmableBird`).
