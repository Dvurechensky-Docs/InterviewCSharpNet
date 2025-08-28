---
layout: default
title: '📚 В чём разница между `IEnumerable<T>` и `IQueryable<T>`?'
description: ''
author: 'Dvurechensky'
date: 2025-08-28
published: true
tags:
  - IEnumerable
  - IQueryable
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
  - [1️⃣ Основные определения](#1️⃣-основные-определения)
    - [`IEnumerable<T>`](#ienumerablet)
    - [`IQueryable<T>`](#iqueryablet)
  - [2️⃣ Deferred Execution (отложенное выполнение)](#2️⃣-deferred-execution-отложенное-выполнение)
  - [3️⃣ Где выполняется обработка](#3️⃣-где-выполняется-обработка)
  - [4️⃣ Примеры различий](#4️⃣-примеры-различий)
    - [`IEnumerable<T>`](#ienumerablet-1)
    - [`IQueryable<T>` (например, Entity Framework)](#iqueryablet-например-entity-framework)
  - [5️⃣ Когда использовать](#5️⃣-когда-использовать)
  - [6️⃣ Производительность](#6️⃣-производительность)
  - [7️⃣ Каверзные моменты на собеседовании](#7️⃣-каверзные-моменты-на-собеседовании)
  - [8️⃣ Пример различий](#8️⃣-пример-различий)
  - [9️⃣ Таблица основных отличий](#9️⃣-таблица-основных-отличий)

**[⬆ Вернуться к главной](../index.md)**

---

## 1️⃣ Основные определения

### `IEnumerable<T>`

- Определён в `System.Collections.Generic`.
- Позволяет **перебирать коллекцию элементов** в памяти (in-memory).
- Поддерживает **только LINQ to Objects**.
- Выполнение **немедленное** только при обращении к коллекции (`foreach`).
- Методы расширения LINQ для `IEnumerable<T>`:

  - `Where`, `Select`, `OrderBy`, `Take`, `Skip`.

- **Отложенное исполнение** (`deferred execution`) — да, но только **в памяти**.

### `IQueryable<T>`

- Определён в `System.Linq`.
- Расширяет `IEnumerable<T>`:

  - Добавляет `Expression` для построения **дерева выражений**.

- Используется для **удалённого источника данных** (например, БД через Entity Framework, LINQ to SQL).
- LINQ-запросы **не выполняются сразу**; создаётся **дерево выражений**.
- Выполнение (`execution`) происходит только при материализации данных (`ToList()`, `ToArray()`, `foreach`).

---

## 2️⃣ Deferred Execution (отложенное выполнение)

- **IEnumerable<T>**

  - Отложенное выполнение выполняется **после того, как объект в памяти доступен**.
  - Пример:

    ```csharp
    var query = list.Where(x => x > 5); // ещё не выполняется
    foreach(var n in query) { ... }     // выполняется здесь
    ```

- **IQueryable<T>**

  - Отложенное выполнение формирует **SQL-запрос (или другой провайдерский запрос)**.
  - Пример EF:

    ```csharp
    var query = db.Users.Where(u => u.Age > 18); // строится Expression tree
    var result = query.ToList();                 // выполняется SQL-запрос
    ```

---

## 3️⃣ Где выполняется обработка

| Характеристика             | IEnumerable<T>           | IQueryable<T>                                                    |
| -------------------------- | ------------------------ | ---------------------------------------------------------------- |
| Источник данных            | В памяти (объекты .NET)  | Внешний источник (БД, веб-сервис)                                |
| SQL/провайдер              | Нет                      | Да, формирует запрос к источнику                                 |
| LINQ методы                | Все выполняются в памяти | Выражения преобразуются в Expression Tree → выполняются удалённо |
| Поддержка сложных запросов | Ограничена (в памяти)    | Полная, если провайдер поддерживает                              |

---

## 4️⃣ Примеры различий

### `IEnumerable<T>`

```csharp
List<int> numbers = new() {1,2,3,4,5};
var query = numbers.Where(n => n > 2).Select(n => n * 2);

foreach(var n in query) Console.WriteLine(n); // выполняется в памяти
```

- Все элементы уже в памяти.
- `Where` и `Select` — **C# делегаты**.

### `IQueryable<T>` (например, Entity Framework)

```csharp
var query = db.Users
    .Where(u => u.Age > 18)
    .Select(u => new { u.Name, u.Age });

var result = query.ToList(); // SQL запрос к БД выполняется здесь
```

- `Where` и `Select` → выражения, переводятся в SQL.
- Эффективно: фильтруются только нужные данные.

---

## 5️⃣ Когда использовать

- **IEnumerable<T>**

  - Когда данные уже загружены в память.
  - Для небольших коллекций и локальных операций.

- **IQueryable<T>**

  - Когда нужно формировать запрос к БД или удалённым источникам.
  - Для больших данных, чтобы фильтровать/проецировать на стороне сервера.

---

## 6️⃣ Производительность

- `IEnumerable<T>`:

  - Перебор всех элементов → **O(n)**.
  - Если фильтруешь много данных → всё уже загружено в память → дорого.

- `IQueryable<T>`:

  - Генерирует **оптимальный запрос** к серверу.
  - Обрабатывает фильтрацию на стороне источника → экономия памяти и сети.

- Подводный камень:

  - Частое комбинирование `IEnumerable<T>` и `IQueryable<T>` может **вынести всю коллекцию в память** раньше времени.

---

## 7️⃣ Каверзные моменты на собеседовании

1. **Что происходит при смешении `IEnumerable<T>` и `IQueryable<T>`?**

   ```csharp
   var result = db.Users.AsEnumerable().Where(u => u.Age > 18);
   ```

   - `AsEnumerable()` переводит `IQueryable` в `IEnumerable` → дальнейшие фильтры выполняются **в памяти**, а не в SQL.

2. **Deferred execution**

   - Часто спрашивают, **когда именно выполняется запрос**.
     Ответ: только при фактической итерации (`foreach`, `ToList()`, `Count()`, `First()`).

3. **Что такое Expression Tree?**

   - В `IQueryable` каждый метод LINQ создаёт `Expression` → можно преобразовать в SQL или другой провайдерский код.

4. **Можно ли использовать сложные функции C# в `IQueryable`?**

   - Нет, только то, что провайдер может транслировать (например, EF Core может не поддерживать все методы).

5. **Память**

   - `IEnumerable` работает в памяти → нагрузка на RAM.
   - `IQueryable` может использовать **отложенную выборку** → меньше памяти.

6. **Вызовы методов**

   - `IEnumerable<T>` — методы LINQ вызываются в **C# коде**.
   - `IQueryable<T>` — методы формируют **дерево выражений** → выполняются в источнике данных.

---

## 8️⃣ Пример различий

```csharp
// IQueryable -> SQL
var query1 = db.Users.Where(u => u.Name.StartsWith("A"));
// SQL генерируется при ToList()

// IEnumerable -> в памяти
var query2 = db.Users.ToList().Where(u => u.Name.StartsWith("A"));
// Все пользователи загружены, фильтрация в памяти
```

- `query1` — эффективнее.
- `query2` — грузим всё в память → неэффективно для больших таблиц.

---

## 9️⃣ Таблица основных отличий

| Характеристика             | IEnumerable<T>                          | IQueryable<T>                             |
| -------------------------- | --------------------------------------- | ----------------------------------------- |
| Namespace                  | System.Collections.Generic              | System.Linq                               |
| Основа                     | Делегаты C#                             | Expression Tree                           |
| Источник данных            | In-memory                               | Remote / DB / External                    |
| Deferred execution         | Да                                      | Да, с генерацией выражения                |
| SQL / Provider translation | Нет                                     | Да                                        |
| Преобразования             | LINQ to Objects                         | LINQ to Entities / LINQ to SQL            |
| Производительность         | Может быть медленнее для больших данных | Оптимизировано для источника              |
| Поддержка методов C#       | Все                                     | Только методы, поддерживаемые провайдером |
| Аллокация памяти           | Вся коллекция                           | Только нужные элементы                    |

---

**[⬆ Вернуться к главной](../index.md)**

<p align="center">✨Dvurechensky✨</p>
