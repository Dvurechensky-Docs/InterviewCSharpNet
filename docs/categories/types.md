---
layout: default
title: '🧱 Какие существуют базовые типы данных в .NET и как они устроены?'
description: ''
author: 'Dvurechensky'
date: 2025-08-28
published: true
tags:
  - types
  - net
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
  - [1️⃣ Основная классификация](#1️⃣-основная-классификация)
  - [2️⃣ Value Types — подробности](#2️⃣-value-types--подробности)
    - [Примеры](#примеры)
    - [Особенности](#особенности)
  - [3️⃣ Reference Types — подробности](#3️⃣-reference-types--подробности)
    - [Примеры](#примеры-1)
    - [Особенности](#особенности-1)
  - [4️⃣ Тип `string` — особый reference type](#4️⃣-тип-string--особый-reference-type)
  - [5️⃣ Nullable типы (`T?`)](#5️⃣-nullable-типы-t)
  - [6️⃣ Структура памяти и упаковка](#6️⃣-структура-памяти-и-упаковка)
  - [7️⃣ Типы в .NET по системной классификации](#7️⃣-типы-в-net-по-системной-классификации)
  - [8️⃣ Каверзные моменты и подводные вопросы](#8️⃣-каверзные-моменты-и-подводные-вопросы)
  - [9️⃣ Примеры](#9️⃣-примеры)

**[⬆ Вернуться к главной](../index.md)**

---

## 1️⃣ Основная классификация

В .NET все типы делятся на две большие категории:

| Категория                   | Примеры                                           | Где хранятся             | Особенности                                                                                  |
| --------------------------- | ------------------------------------------------- | ------------------------ | -------------------------------------------------------------------------------------------- |
| Value types (значимые)      | `int`, `double`, `bool`, `struct`, `enum`         | Stack / Inline в объекте | Хранят значение прямо, копируются при присваивании, не требуют GC для памяти самого значения |
| Reference types (ссылочные) | `string`, `class`, `object`, `interface`, массивы | Heap                     | Хранят **ссылку на объект**, GC управляет памятью, присваивание копирует ссылку              |

---

## 2️⃣ Value Types — подробности

### Примеры

- `int` (`System.Int32`) — 32-битное целое.
- `long` (`System.Int64`) — 64-битное целое.
- `float` (`System.Single`) — 32-битное с плавающей точкой.
- `double` (`System.Double`) — 64-битное с плавающей точкой.
- `decimal` (`System.Decimal`) — 128-битная точность, для финансов.
- `bool` — 1 байт, `true` / `false`.
- `char` — 16-битный символ Unicode.
- `struct` — пользовательские value type.
- `enum` — набор констант на основе целочисленного типа.

### Особенности

1. **Хранятся на стеке**, если локальные переменные.
2. **Копируются при присваивании**:

   ```csharp
   int a = 5;
   int b = a; // создаётся копия
   b = 10;    // a остаётся 5
   ```

3. **Нет null**, кроме `Nullable<T>` (`int?`).
4. **Упаковка (Boxing)** — преобразование value type в object:

   ```csharp
   int x = 42;
   object obj = x; // boxing
   int y = (int)obj; // unboxing
   ```

   - Упаковка — дорогостоящая операция, создает объект на heap.

---

## 3️⃣ Reference Types — подробности

### Примеры

- `string`
- `class`
- `object`
- `interface`
- массивы (`T[]`)

### Особенности

1. Хранят **ссылку на объект**, сам объект на **heap**.
2. Присваивание копирует ссылку, а не значение:

   ```csharp
   class Person { public string Name; }
   Person p1 = new Person { Name = "Alice" };
   Person p2 = p1;
   p2.Name = "Bob";
   Console.WriteLine(p1.Name); // "Bob"
   ```

3. GC управляет освобождением памяти.
4. Может быть `null`.

---

## 4️⃣ Тип `string` — особый reference type

- **Immutable** (неизменяемый) объект.
- Любая модификация строки создает **новый объект в памяти**.
- Методы вроде `Replace`, `Concat` создают новые строки.
- Оптимизация: **interning** — одинаковые литералы могут ссылаться на один объект.

---

## 5️⃣ Nullable типы (`T?`)

- Для value type можно разрешить `null` через `Nullable<T>`:

  ```csharp
  int? x = null;
  ```

- Внутри хранится **value + flag** (`HasValue`).

---

## 6️⃣ Структура памяти и упаковка

- **Stack**:

  - Локальные переменные value type.
  - Быстрое выделение и освобождение (по выходу из метода).

- **Heap**:

  - Ссылочные типы и упакованные value types.
  - GC освобождает память.

- **Boxing / Unboxing**

  - Boxing: value type → object (на heap).
  - Unboxing: object → value type.
  - Частые операции → накладные расходы.

---

## 7️⃣ Типы в .NET по системной классификации

| CLR тип          | C# тип    | Размер    | Примечание   |
| ---------------- | --------- | --------- | ------------ |
| `System.Boolean` | `bool`    | 1 байт    |              |
| `System.Byte`    | `byte`    | 1 байт    | беззнаковый  |
| `System.SByte`   | `sbyte`   | 1 байт    | знаковый     |
| `System.Int16`   | `short`   | 2 байта   |              |
| `System.UInt16`  | `ushort`  | 2 байта   | беззнаковый  |
| `System.Int32`   | `int`     | 4 байта   |              |
| `System.UInt32`  | `uint`    | 4 байта   | беззнаковый  |
| `System.Int64`   | `long`    | 8 байт    |              |
| `System.UInt64`  | `ulong`   | 8 байт    | беззнаковый  |
| `System.Single`  | `float`   | 4 байта   | IEEE 754     |
| `System.Double`  | `double`  | 8 байт    | IEEE 754     |
| `System.Decimal` | `decimal` | 16 байт   | для финансов |
| `System.Char`    | `char`    | 2 байта   | UTF-16       |
| `System.String`  | `string`  | ссылочный | immutable    |
| `System.Object`  | `object`  | ссылочный | базовый тип  |

---

## 8️⃣ Каверзные моменты и подводные вопросы

1. **Boxing / unboxing**

   - Вопросы на производительность: когда оно происходит, чем опасно.

2. **Почему string reference, а int value?**

   - Потому что строки immutable, управляются на heap.

3. **Разница между struct и class**

   - struct — value type, копируется по значению, хранится на стеке или inline.
   - class — reference type, хранится на heap, GC управляет.

4. **Nullable типы**

   - Как работают и что внутри `Nullable<T>` (`HasValue + Value`).

5. **Что происходит при присваивании**

   - Value type → копия.
   - Reference type → копия ссылки.

6. **Размеры типов**

   - Вопрос о `int` = 4 байта, `long` = 8 байт и т.д.

7. **Immutable vs mutable**

   - string immutable, struct по умолчанию mutable (можно делать readonly struct).

---

## 9️⃣ Примеры

```csharp
int a = 10;
int b = a; // копия
b = 20;
Console.WriteLine(a); // 10

string s1 = "hello";
string s2 = s1; // ссылка на тот же объект
s2 = s2.Replace("h","H");
Console.WriteLine(s1); // "hello"
```

```csharp
int? x = null;
if (x.HasValue)
    Console.WriteLine(x.Value);
```

```csharp
object obj = 42;      // boxing
int y = (int)obj;     // unboxing
```

---

**[⬆ Вернуться к главной](../index.md)**

<p align="center">✨Dvurechensky✨</p>
