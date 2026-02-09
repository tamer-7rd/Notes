# Mock-собес по DSA (Week 14)

## Цель
Подготовка к секции алгоритмов и структур данных (Coding Interview). В этой секции проверяется умение решать алгоритмические задачи, писать чистый код и оценивать сложность решения.

---

## Структура решения задачи (FRAMEWORK)

### 1. Уточнение условия (Clarification) - 2-5 мин
Не бросаться сразу писать код!
*   **Входные данные:** Типы данных, ограничения (размер массива, диапазон чисел), формат.
*   **Крайние случаи (Edge Cases):** Пустой массив? Отрицательные числа? Дубликаты? Большие числа (overflow)?
*   **Пример:** "Если на вход `[1, 2, 3]`, то ожидаем `6`?"

### 2. Обсуждение подхода (Approach) - 5-10 мин
Сначала обсуди решение голосом или псевдокодом.
*   **Brute Force:** Сначала предложи наивное решение. Оцени его сложность (например, O(N^2)).
*   **Оптимизация:** "Можем ли мы сделать лучше?". Предложи более эффективный алгоритм (например, O(N) или O(N log N)).
    *   Использование дополнительной памяти (Hash Map, Set).
    *   Сортировка.
    *   Два указателя (Two Pointers).
*   **Согласование:** "Вам нравится этот подход? Могу приступать к коду?"

### 3. Написание кода (Coding) - 15-20 мин
Пиши чистый и понятный код.
*   Используй понятные имена переменных.
*   Следи за стилем кода (отступы, скобки).
*   Разбивай на функции, если логика сложная.
*   Комментируй неочевидные моменты, но не каждую строку.

### 4. Тестирование и отладка (Dry Run) - 5 мин
Пройдись по коду вручную с примером.
*   **Happy Path:** Обычный пример.
*   **Edge Cases:** Пустой ввод, один элемент, граничные значения.
*   Найди и исправь ошибки (баги) *до* того, как это сделает интервьюер.

### 5. Анализ сложности (Complexity Analysis)
*   **Time Complexity (Big O):** O(1), O(log N), O(N), O(N log N), O(N^2). Объясни почему.
*   **Space Complexity:** Сколько дополнительной памяти используем?

---

## Основные паттерны и темы (Must Know)

### 1. Массивы и Строки (Arrays & Strings)
*   **Two Pointers:** Palindrome, Two Sum (sorted), Container With Most Water.
*   **Sliding Window:** Longest Substring Without Repeating Characters.
*   **Prefix Sum:** Range Sum Query.

### 2. Хэш-таблицы (Hash Maps / Sets)
*   **Frequency Counter:** Valid Anagram.
*   **Lookups:** Two Sum (unsorted), Group Anagrams.

### 3. Связные списки (Linked Lists)
*   **Slow & Fast Pointers:** Cycle Detection, Middle of List.
*   **Reversal:** Reverse Linked List.

### 4. Деревья и Графы (Trees & Graphs)
*   **BFS (Breadth-First Search):** Shortest Path, Level Order Traversal.
*   **DFS (Depth-First Search):** Path Finding, Valid BST, Number of Islands.

### 5. Бинарный поиск (Binary Search)
*   Поиск в отсортированном массиве (O(log N)).

### 6. Стек и Очередь (Stack & Queue)
*   **Stack:** Valid Parentheses.
*   **Queue:** BFS implementation.

---

## Чек-лист для самопроверки
- [ ] Я задал вопросы по ограничениям (Constraints)?
- [ ] Я проговорил решение вслух *до* написания кода?
- [ ] Я оценил сложность (Time & Space)?
- [ ] Я проверил код на крайних случаях (null, empty, 0, 1)?
- [ ] Код читаемый и аккуратный?

## Полезные ресурсы
*   *LeetCode* (Top Interview 150)
*   *NeetCode.io* (Roadmap)
*   *Cracking the Coding Interview* (Book)
