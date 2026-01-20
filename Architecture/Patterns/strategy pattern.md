# Strategy Pattern (Паттерн Стратегия) — для Junior

## 🎯 Что это простыми словами?

Представь навигатор. Ты хочешь добраться из точки А в точку Б.

Навигатор предлагает варианты:
1. 🚗 На машине (быстро, но пробки).
2. 🚶 Пешком (медленно, но через парк).
3. 🚌 На автобусе (дёшево, но долго ждать).

Цель одна — **добраться**. Способы (**стратегии**) — разные.

**Strategy Pattern** — это когда твой код умеет переключаться между разными способами решения одной задачи на лету.

---

## ❌ Проблема: Ад из `if/else`

Представь интернет-магазин. Нужно рассчитать цену доставки.

```python
def calculate_shipping(package, method):
    if method == "standard":
        return 5.00  # Фикс цена
    elif method == "express":
        return package.weight * 2.50  # Зависит от веса
    elif method == "free":
        return 0.00  # Бесплатно
    elif method == "drone":
        return 10.00 + package.weight * 0.5  # Сложная формула
    # ... и так далее
```

**Почему это плохо?**
- Добавишь новый способ (например, "Самовывоз") → придётся лезть в этот код и дописывать `elif`.
- Функция станет огромной и запутанной.
- Сложно тестировать каждый способ отдельно.

---

## ✅ Решение: Strategy Pattern

Выносим каждый способ в отдельную функцию (стратегию).

### Шаг 1: Создаём стратегии (способы)

```python
# Стратегия 1: Стандартная
def standard_shipping(package):
    return 5.00

# Стратегия 2: Экспресс
def express_shipping(package):
    return package.weight * 2.50

# Стратегия 3: Бесплатная
def free_shipping(package):
    return 0.00
```

### Шаг 2: Создаём "Меню" (Словарь)

```python
SHIPPING_STRATEGIES = {
    "standard": standard_shipping,
    "express": express_shipping,
    "free": free_shipping,
}
```

### Шаг 3: Главная функция (Контекст)

```python
def calculate_shipping(package, method):
    # 1. Достаём нужную стратегию из словаря
    strategy = SHIPPING_STRATEGIES.get(method)
    
    # 2. Если стратегии нет — ошибка
    if not strategy:
        raise ValueError("Неизвестный способ доставки")
    
    # 3. Выполняем стратегию
    return strategy(package)
```

**Теперь:**
- Хочешь добавить "Дрон"? Просто напиши функцию `drone_shipping` и добавь её в словарь.
- Главную функцию `calculate_shipping` менять НЕ НУЖНО!

---

## 🔍 Живой пример: Сортировка товаров

У тебя есть список товаров. Пользователь хочет сортировать их по-разному.

### ❌ Без Стратегии (Плохо):

```python
def sort_products(products, sort_type):
    if sort_type == "price_asc":
        return sorted(products, key=lambda p: p.price)
    elif sort_type == "price_desc":
        return sorted(products, key=lambda p: p.price, reverse=True)
    elif sort_type == "name":
        return sorted(products, key=lambda p: p.name)
    # И так далее...
```

### ✅ Со Стратегией (Хорошо):

```python
# 1. Стратегии
def by_price_asc(products):
    return sorted(products, key=lambda p: p.price)

def by_price_desc(products):
    return sorted(products, key=lambda p: p.price, reverse=True)

def by_name(products):
    return sorted(products, key=lambda p: p.name)

# 2. Словарь
SORT_STRATEGIES = {
    "price_asc": by_price_asc,
    "price_desc": by_price_desc,
    "name": by_name,
}

# 3. Использование
def sort_products(products, sort_type):
    strategy = SORT_STRATEGIES.get(sort_type)
    return strategy(products)
```

---

## 🎯 Когда использовать?

**ДА (Используй):**
- У тебя есть 3+ способа сделать одно и то же (доставка, оплата, сортировка).
- Ты выбираешь способ во время работы программы (пользователь нажал кнопку).
- Ты не хочешь писать километровые `if/elif`.

**НЕТ (Не используй):**
- У тебя всего 1-2 способа.
- Способы никогда не меняются.
- Код используется только в одном месте.

---

## 🎓 Главное для Junior:

1. **Strategy** = это просто способ избавиться от кучи `if/elif`.
2. **Как работает:** Выносишь каждый вариант в отдельную функцию → кладёшь в словарь → выбираешь из словаря по ключу.
3. **Плюс:** Легко добавлять новые варианты, не ломая старый код.

**Запомни:** Видишь кучу `if/elif`, которые делают одно и то же по-разному? Это Стратегия!
