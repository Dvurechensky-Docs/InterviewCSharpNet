---
layout: default
title: '📦 Чем отличаются разные коллекции в .NET и когда какую лучше использовать?'
description: ''
author: 'Dvurechensky'
date: 2025-08-28
published: true
tags:
  - collections
  - C#
---

<h1 align="center">📦 Чем отличаются разные коллекции в .NET и когда какую лучше использовать?</h1>
<p align="center">
    <a href="https://git.io/typing-svg"><img src="https://readme-typing-svg.demolab.com?font=Fira+Code&pause=1000&center=true&vCenter=true&width=435&lines=%D0%9F%D0%BE%D0%B2%D1%82%D0%BE%D1%80%D0%B5%D0%BD%D0%B8%D0%B5+-+%D0%BC%D0%B0%D1%82%D1%8C+%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D1%8F" alt="Typing SVG" /></a>
</p>
<p align="center">
    <a href="https://sites.google.com/view/dvurechensky" target="_blank"><img alt="Static Badge" src="https://shields.dvurechensky.pro/badge/Dvurechensky-Nikolay-blue"></a>
</p>

# ✨ Оглавление

- [✨ Оглавление](#-оглавление)
  - [1️⃣ Основная классификация коллекций](#1️⃣-основная-классификация-коллекций)
  - [2️⃣ List vs Array](#2️⃣-list-vs-array)
  - [3️⃣ LinkedList](#3️⃣-linkedlist)
  - [4️⃣ Stack / Queue](#4️⃣-stack--queue)
  - [5️⃣ HashSet / Dictionary\<K,V\>](#5️⃣-hashset--dictionarykv)
  - [6️⃣ SortedList / SortedDictionary / SortedSet](#6️⃣-sortedlist--sorteddictionary--sortedset)
  - [7️⃣ Concurrent коллекции](#7️⃣-concurrent-коллекции)
  - [8️⃣ ObservableCollection / ReadOnlyCollection](#8️⃣-observablecollection--readonlycollection)
  - [9️⃣ Подводные моменты и каверзные вопросы](#9️⃣-подводные-моменты-и-каверзные-вопросы)
  - [🔟 Пример выбора коллекции](#-пример-выбора-коллекции)

**[⬆ Вернуться к главной](../index.md)**

---

## 1️⃣ Основная классификация коллекций

| Категория                                     | Примеры                                            | Особенности                                                               | Когда использовать                                         |
| --------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **List / Array**                              | `List<T>`, `T[]`                                   | Индексированный доступ, динамический (`List<T>`), фиксированный (`array`) | Частые обращения по индексу, небольшие вставки/удаления    |
| **LinkedList**                                | `LinkedList<T>`                                    | Двунаправленный список, быстрые вставки/удаления в середине               | Когда часто нужно вставлять/удалять элементы внутри списка |
| **Stack / Queue**                             | `Stack<T>`, `Queue<T>`                             | LIFO / FIFO                                                               | Очередь или стек операций                                  |
| **HashSet / Dictionary**                      | `HashSet<T>`, `Dictionary<K,V>`                    | Уникальные элементы / ключ-значение, быстрый поиск                        | Уникальные элементы, быстрый доступ по ключу               |
| **SortedList / SortedDictionary / SortedSet** | `SortedList<K,V>`, `SortedDictionary<K,V>`         | Отсортированные коллекции                                                 | Когда нужен доступ по ключу с сортировкой                  |
| **Concurrent коллекции**                      | `ConcurrentDictionary`, `ConcurrentQueue`          | Потокобезопасные                                                          | Многопоточные приложения                                   |
| **Observable / ReadOnly**                     | `ObservableCollection<T>`, `ReadOnlyCollection<T>` | Для binding UI / защиты от изменений                                      | WPF/WinForms MVVM, когда нужен контроль доступа            |

---

## 2️⃣ List<T> vs Array

| Особенность        | Array                                    | List<T>                                        |
| ------------------ | ---------------------------------------- | ---------------------------------------------- |
| Размер             | фиксированный                            | динамический, увеличивается по мере добавления |
| Индексирование     | O(1)                                     | O(1)                                           |
| Вставка/удаление   | дорого (копирование)                     | дорого в середине, быстро в конце (`Add`)      |
| Тип                | value/ref                                | value/ref                                      |
| Использовать когда | известен размер заранее, не нужно менять | размер меняется динамически, нужен удобный API |

---

## 3️⃣ LinkedList<T>

- Двунаправленный список (`Prev`/`Next`).
- Вставка/удаление в середине — O(1), если есть Node.
- Доступ по индексу — O(n) (медленно!).
- Подходит для: очередей, списков с частыми вставками/удалениями внутри.

---

## 4️⃣ Stack<T> / Queue<T>

- **Stack<T>**

  - LIFO (Last In First Out)
  - Методы: `Push`, `Pop`, `Peek`

- **Queue<T>**

  - FIFO (First In First Out)
  - Методы: `Enqueue`, `Dequeue`, `Peek`

- Подходит для:

  - Стек вызовов, отмены операций, парсинг
  - Очереди задач, обработка событий

---

## 5️⃣ HashSet<T> / Dictionary\<K,V>

- **HashSet<T>**

  - Уникальные элементы
  - Быстрый поиск: O(1)
  - Методы: `Add`, `Remove`, `Contains`

- **Dictionary\<K,V>**

  - Ключ → Значение
  - Быстрый поиск по ключу: O(1)
  - Методы: `Add`, `TryGetValue`, `ContainsKey`

- Особенности:

  - Хэш-функция `GetHashCode()` должна быть корректной!
  - Конфликты хэшей → цепочки → падает производительность (редко)

---

## 6️⃣ SortedList / SortedDictionary / SortedSet

- **SortedList\<K,V>**

  - Хранит элементы в массиве
  - Поиск по ключу O(log n)
  - Хорошо для небольших коллекций

- **SortedDictionary\<K,V>**

  - На базе сбалансированного дерева (RB-tree)
  - Поиск O(log n), вставка/удаление O(log n)
  - Лучше для больших динамических коллекций

- **SortedSet<T>**

  - Уникальные, отсортированные элементы
  - На базе дерева

---

## 7️⃣ Concurrent коллекции

- Потокобезопасные:

  - `ConcurrentDictionary<K,V>`
  - `ConcurrentQueue<T>`
  - `ConcurrentStack<T>`
  - `ConcurrentBag<T>`

- Позволяют безопасно добавлять и удалять элементы из разных потоков без lock.
- Использовать, когда **многопоточность неизбежна**, обычные коллекции не подходят.

---

## 8️⃣ ObservableCollection<T> / ReadOnlyCollection<T>

- **ObservableCollection<T>**

  - События `CollectionChanged`
  - Используется для **UI binding** (WPF, MVVM)

- **ReadOnlyCollection<T>**

  - Только чтение
  - Обёртка над List<T> или массивом

---

## 9️⃣ Подводные моменты и каверзные вопросы

1. **List<T> vs LinkedList<T>**

   - Когда O(1) вставка в середину реально быстрее?
   - Когда индексирование медленнее?

2. **Array vs List<T>**

   - Почему массив быстрее по памяти и cache-friendly?

3. **HashSet / Dictionary**

   - Что произойдет при плохой `GetHashCode()`?

4. **Concurrent коллекции**

   - Какие накладные расходы, чем отличаются от lock + обычного Dictionary?

5. **Sorted vs Unsorted**

   - SortedList быстрее по памяти, медленнее вставка?
   - SortedDictionary vs SortedList: trade-offs

6. **Boxing / unboxing**

   - `ArrayList` устарел → boxing value types

7. **CopyOnWrite**

   - ReadOnlyCollection или ToArray → копирование, какие подводные моменты?

---

## 🔟 Пример выбора коллекции

| Сценарий                           | Лучшая коллекция                           | Почему                       |
| ---------------------------------- | ------------------------------------------ | ---------------------------- |
| Частый доступ по индексу           | `List<T>` / `T[]`                          | O(1)                         |
| Частые вставки/удаления в середине | `LinkedList<T>`                            | O(1) вставка/удаление        |
| Уникальные элементы                | `HashSet<T>`                               | Быстрый поиск и уникальность |
| Ключ → Значение                    | `Dictionary<K,V>`                          | O(1) поиск по ключу          |
| Отсортированные элементы           | `SortedList` / `SortedDictionary`          | Автоматическая сортировка    |
| Многопоточные добавления/удаления  | `ConcurrentDictionary` / `ConcurrentQueue` | Потокобезопасно              |
| UI привязка                        | `ObservableCollection<T>`                  | События изменений            |

---

**[⬆ Вернуться к главной](../index.md)**

<p align="center">✨Dvurechensky✨</p>
