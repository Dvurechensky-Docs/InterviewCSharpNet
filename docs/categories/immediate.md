---
layout: default
title: '🕰️ Что такое отложенные (deferred) и немедленные (immediate) запросы в LINQ?'
description: ''
author: 'Dvurechensky'
date: 2025-08-28
published: true
tags:
  - deferred
  - immediate
  - LINQ
  - C#
---

<p align="center">
    <a href="https://git.io/typing-svg"><img src="https://readme-typing-svg.demolab.com?font=Fira+Code&pause=1000&center=true&vCenter=true&width=435&lines=%D0%9F%D0%BE%D0%B2%D1%82%D0%BE%D1%80%D0%B5%D0%BD%D0%B8%D0%B5+-+%D0%BC%D0%B0%D1%82%D1%8C+%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D1%8F" alt="Typing SVG" /></a>
</p>
<p align="center">
    <a href="https://sites.google.com/view/dvurechensky" target="_blank"><img alt="Static Badge" src="https://shields.dvurechensky.pro/badge/Dvurechensky-Nikolay-blue"></a>
</p>

# ✨ Оглавление

- [✨ Оглавление](#-оглавление)
  - [1️⃣ Основная идея](#1️⃣-основная-идея)
  - [2️⃣ Deferred execution — отложенные запросы](#2️⃣-deferred-execution--отложенные-запросы)
  - [3️⃣ Immediate execution — немедленные запросы](#3️⃣-immediate-execution--немедленные-запросы)
  - [4️⃣ Отличия Deferred и Immediate](#4️⃣-отличия-deferred-и-immediate)
  - [5️⃣ Подводные моменты и каверзные вопросы](#5️⃣-подводные-моменты-и-каверзные-вопросы)
  - [6️⃣ Примеры](#6️⃣-примеры)
    - [Отложенный запрос](#отложенный-запрос)
    - [Немедленный запрос](#немедленный-запрос)

**[⬆ Вернуться к главной](../index.md)**

---

## 1️⃣ Основная идея

- **Deferred execution (отложенное выполнение)**: запрос **не выполняется сразу**, а откладывается до момента, когда к результату обращаются (`foreach`, `ToList()`, `Count()`, `First()`).
- **Immediate execution (немедленное выполнение)**: запрос **выполняется сразу**, данные материализуются в память.

---

## 2️⃣ Deferred execution — отложенные запросы

- Выполнение запроса **откладывается до момента итерации**.
- Все стандартные LINQ методы для `IEnumerable<T>` (например, `Where`, `Select`, `Take`) **отложенные**.
- Пример:

```csharp
List<int> numbers = new() {1,2,3,4,5};
var query = numbers.Where(n => n > 2); // ещё не выполняется

numbers.Add(6); // это повлияет на результат
foreach(var n in query)
{
    Console.WriteLine(n); // теперь выполняется, выводит 3,4,5,6
}
```

**Особенности:**

1. Отложенное выполнение **реагирует на изменения источника данных**.
2. Экономит память, особенно при больших коллекциях.
3. Позволяет строить цепочки фильтров и трансформаций, которые **не создают новые коллекции**.

**Примеры отложенных методов:**

- `Where`
- `Select`
- `Take`, `Skip`
- `OrderBy`, `ThenBy`
- `SelectMany`

---

## 3️⃣ Immediate execution — немедленные запросы

- Выполнение запроса **происходит сразу**, результат сохраняется в коллекции.
- Пример:

```csharp
var numbersList = numbers.Where(n => n > 2).ToList(); // выполняется сразу
numbers.Add(7);
Console.WriteLine(numbersList.Count); // не учитывает 7
```

**Методы немедленного выполнения:**

- `ToList()`, `ToArray()`
- `Count()`, `LongCount()`
- `Sum()`, `Average()`, `Min()`, `Max()`
- `First()`, `FirstOrDefault()`, `Single()`, `SingleOrDefault()`
- `Any()`, `All()`, `Contains()`

---

## 4️⃣ Отличия Deferred и Immediate

| Характеристика              | Deferred Execution                 | Immediate Execution                      |
| --------------------------- | ---------------------------------- | ---------------------------------------- |
| Когда выполняется           | При итерации                       | Сразу при вызове метода                  |
| Память                      | Минимально                         | Все результаты материализуются           |
| Влияние изменений источника | Да                                 | Нет, результат фиксирован                |
| Методы LINQ                 | Where, Select, Take, Skip, OrderBy | ToList, ToArray, Count, First, Sum, etc. |
| Производительность          | Эффективно при больших данных      | Может быть дороже для больших данных     |

---

## 5️⃣ Подводные моменты и каверзные вопросы

1. **Изменение источника данных**

   - Отложенный запрос учитывает изменения:

     ```csharp
     var query = list.Where(x => x > 0);
     list.Add(10);
     foreach(var n in query) { ... } // включает 10
     ```

   - Немедленный запрос игнорирует изменения после материализации.

2. **Повторная итерация**

   - Deferred: каждый `foreach` выполняет запрос заново.
   - Immediate: данные уже в памяти, повторная итерация не вызывает повторного выполнения.

3. **LINQ to SQL / IQueryable**

   - Deferred — **SQL запрос формируется**, но не выполняется до `ToList()` или `foreach`.
   - Immediate — `ToList()`, `Count()` вызывают **фактический запрос к БД**.

4. **Побочные эффекты**

   - Если источник данных изменяется или имеет методы с побочными эффектами, **Deferred** может вызывать их несколько раз.
   - Immediate фиксирует результат один раз.

5. **Накладные расходы**

   - Deferred экономит память, но при многократной итерации может выполнять один и тот же запрос несколько раз.
   - Immediate: больше памяти, меньше повторных вычислений.

---

## 6️⃣ Примеры

### Отложенный запрос

```csharp
var numbers = new List<int> {1,2,3};
var query = numbers.Where(n => n > 1); // Deferred
numbers.Add(4);
foreach(var n in query) Console.WriteLine(n); // 2,3,4
```

### Немедленный запрос

```csharp
var numbers = new List<int> {1,2,3};
var result = numbers.Where(n => n > 1).ToList(); // Immediate
numbers.Add(4);
foreach(var n in result) Console.WriteLine(n); // 2,3
```

---

**[⬆ Вернуться к главной](../index.md)**

<p align="center">✨Dvurechensky✨</p>
