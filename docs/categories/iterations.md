---
layout: default
title: '🔄 Что такое итераторы и как они реализуются?'
description: ''
author: 'Dvurechensky'
date: 2025-08-28
published: true
tags:
  - iterations
  - C#
---

<h1 align="center">🔄 Что такое итераторы и как они реализуются?</h1>
<p align="center">
    <a href="https://git.io/typing-svg"><img src="https://readme-typing-svg.demolab.com?font=Fira+Code&pause=1000&center=true&vCenter=true&width=435&lines=%D0%9F%D0%BE%D0%B2%D1%82%D0%BE%D1%80%D0%B5%D0%BD%D0%B8%D0%B5+-+%D0%BC%D0%B0%D1%82%D1%8C+%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D1%8F" alt="Typing SVG" /></a>
</p>
<p align="center">
    <a href="https://sites.google.com/view/dvurechensky" target="_blank"><img alt="Static Badge" src="https://shields.dvurechensky.pro/badge/Dvurechensky-Nikolay-blue"></a>
</p>

# ✨ Оглавление

- [✨ Оглавление](#-оглавление)
  - [1️⃣ Основная идея итераторов](#1️⃣-основная-идея-итераторов)
  - [2️⃣ Интерфейсы](#2️⃣-интерфейсы)
    - [`IEnumerable<T>`](#ienumerablet)
    - [`IEnumerator<T>`](#ienumeratort)
  - [3️⃣ Реализация через `yield return`](#3️⃣-реализация-через-yield-return)
  - [4️⃣ Deferred execution](#4️⃣-deferred-execution)
  - [5️⃣ `yield break`](#5️⃣-yield-break)
  - [6️⃣ Сложные моменты и каверзные вопросы](#6️⃣-сложные-моменты-и-каверзные-вопросы)
  - [7️⃣ Пример сложного итератора](#7️⃣-пример-сложного-итератора)
  - [8️⃣ Итераторы и LINQ](#8️⃣-итераторы-и-linq)
  - [9️⃣ Подводные моменты для собеса](#9️⃣-подводные-моменты-для-собеса)
  - [🔟 Таблица сравнения](#-таблица-сравнения)

**[⬆ Вернуться к главной](../index.md)**

---

## 1️⃣ Основная идея итераторов

- **Итератор** — объект, позволяющий **последовательно перебирать элементы коллекции** без раскрытия её внутренней структуры.
- Позволяет писать **foreach-подобный код** на своих коллекциях.
- Реализуется через интерфейсы:

  - `IEnumerable<T>` — предоставляет `GetEnumerator()`
  - `IEnumerator<T>` — предоставляет `Current` и `MoveNext()`

---

## 2️⃣ Интерфейсы

### `IEnumerable<T>`

```csharp
public interface IEnumerable<out T>
{
    IEnumerator<T> GetEnumerator();
}
```

- Используется для **отложенного перебора элементов**.
- Может быть **несколько вызовов GetEnumerator**, каждый создаёт независимый перебор.

### `IEnumerator<T>`

```csharp
public interface IEnumerator<out T> : IDisposable
{
    T Current { get; }
    bool MoveNext();
    void Reset(); // редко используется
}
```

- **MoveNext()** — продвигает курсор.
- **Current** — возвращает текущий элемент.
- **Reset()** — сброс перебора (необязателен, редко используется).

---

## 3️⃣ Реализация через `yield return`

- Ключевое слово **`yield return`** позволяет создавать **итератор без ручной реализации IEnumerator**.
- Пример:

```csharp
IEnumerable<int> GetNumbers()
{
    for (int i = 0; i < 5; i++)
        yield return i;
}
```

- Особенности:

  - Компилятор создаёт **анонимный класс**, который реализует `IEnumerator<T>`.
  - Поддерживает **отложенное выполнение (deferred execution)**.
  - Поддерживает **состояние между вызовами MoveNext()**.

---

## 4️⃣ Deferred execution

- Итератор не выполняется сразу при вызове метода.
- Выполняется **по мере перебора через foreach или LINQ**.
- Пример:

```csharp
var numbers = GetNumbers(); // метод ещё не выполняется
foreach(var n in numbers)    // тут начинается генерация
    Console.WriteLine(n);
```

- Преимущества:

  - Экономия памяти.
  - Возможность работать с бесконечными потоками данных.

- Минусы:

  - Изменения исходной коллекции между вызовами могут влиять на результат.

---

## 5️⃣ `yield break`

- Прерывает итерацию досрочно.
- Пример:

```csharp
IEnumerable<int> NumbersUpTo(int max)
{
    for (int i = 0; i < max; i++)
    {
        if (i == 3)
            yield break; // завершение итератора
        yield return i;
    }
}
```

---

## 6️⃣ Сложные моменты и каверзные вопросы

1. **Deferred vs Immediate**

   - `yield return` → deferred.
   - `ToList()` → immediate.

2. **Многоэтапные итераторы**

   - Можно использовать несколько `yield return` в условных блоках, каждый поддерживает своё состояние.

3. **Изменение коллекции во время итерации**

   - Может вызвать `InvalidOperationException` (для большинства коллекций, кроме Concurrent).

4. **Lazy evaluation**

   - Итераторы помогают реализовать ленивое вычисление (lazy).

5. **IEnumerator.Dispose()**

   - Автоматически вызывается после окончания перебора (`foreach` использует try-finally).

6. **Структурные итераторы**

   - Можно возвращать `struct` для уменьшения накладных расходов на GC.

7. **Несколько foreach на одном IEnumerable**

   - Каждый вызов GetEnumerator создаёт **новый независимый перебор**.

---

## 7️⃣ Пример сложного итератора

```csharp
IEnumerable<int> FilterAndSquare(IEnumerable<int> numbers)
{
    foreach (var n in numbers)
    {
        if (n % 2 == 0)
            yield return n * n; // только чётные возводим в квадрат
    }
}
```

- Вызов `FilterAndSquare(list)` не выполнит ничего, пока не начнётся foreach.
- Deferred execution сохраняет состояние между вызовами MoveNext.

---

## 8️⃣ Итераторы и LINQ

- LINQ методы (`Select`, `Where`) возвращают **IEnumerable<T>**, используют deferred execution.
- Пример:

```csharp
var evens = numbers.Where(n => n % 2 == 0); // ещё не выполняется
foreach(var n in evens)
    Console.WriteLine(n); // здесь выполнение
```

- Вопросы:

  - Когда LINQ выполняется? (deferred vs immediate)
  - Какие операции immediate? (`ToList()`, `Count()`, `Sum()`)

---

## 9️⃣ Подводные моменты для собеса

1. `yield return` vs обычный List

   - Меньше памяти, lazy, но нельзя использовать с асинхронными методами напрямую.

2. `IEnumerator` vs `IEnumerable`

   - `IEnumerable` предоставляет enumerator, который делает фактический перебор.

3. Можно ли использовать `yield return` в асинхронных методах?

   - Нет, для этого есть `IAsyncEnumerable<T>` и `await foreach`.

4. Как устроен внутренний state machine компилятора для yield return?

   - Каждый yield → точка сохранения состояния.

5. Почему `foreach` вызывает Dispose?

   - Для правильного освобождения ресурсов в IEnumerator.

---

## 🔟 Таблица сравнения

| Характеристика       | IEnumerable<T> | IEnumerator<T> | yield return                              |
| -------------------- | -------------- | -------------- | ----------------------------------------- |
| Кто предоставляет    | Коллекция      | Перебор        | Компилятор создаёт enumerator             |
| Deferred execution   | Да             | Нет            | Да                                        |
| Dispose              | Нет            | Да             | Да (через enumerator)                     |
| Много переборов      | Да             | Один           | Да, каждый GetEnumerator новый            |
| Сложность реализации | Нужно вручную  | Нужно вручную  | Очень просто, компилятор генерирует класс |

---

**[⬆ Вернуться к главной](../index.md)**

<p align="center">✨Dvurechensky✨</p>
