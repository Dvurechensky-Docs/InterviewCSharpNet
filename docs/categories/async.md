---
layout: default
title: '⚡ Как устроены асинхронность и многопоточность в .NET?'
description: ''
author: 'Dvurechensky'
date: 2025-08-28
published: true
tags:
  - Thread
  - Task
  - Async
  - await
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
  - [1️⃣ Основные понятия](#1️⃣-основные-понятия)
  - [2️⃣ Потоки (Thread)](#2️⃣-потоки-thread)
  - [3️⃣ ThreadPool](#3️⃣-threadpool)
  - [4️⃣ Tasks (Task Parallel Library, TPL)](#4️⃣-tasks-task-parallel-library-tpl)
  - [5️⃣ Асинхронность (`async` / `await`)](#5️⃣-асинхронность-async--await)
  - [6️⃣ Многопоточность vs Асинхронность](#6️⃣-многопоточность-vs-асинхронность)
  - [7️⃣ Примитивы синхронизации](#7️⃣-примитивы-синхронизации)
  - [8️⃣ Каверзные моменты и вопросы](#8️⃣-каверзные-моменты-и-вопросы)
  - [9️⃣ Пример: CPU-bound vs IO-bound](#9️⃣-пример-cpu-bound-vs-io-bound)
  - [🔟 Таблица: асинхронность vs многопоточность](#-таблица-асинхронность-vs-многопоточность)

**[⬆ Вернуться к главной](../index.md)**

---

## 1️⃣ Основные понятия

| Понятие                    | Описание                                                                    |
| -------------------------- | --------------------------------------------------------------------------- |
| **Thread (Поток)**         | Независимый поток выполнения на CPU.                                        |
| **Task (Задача)**          | Абстракция работы, которая может выполняться асинхронно или на потоке.      |
| **Async/await**            | Синтаксис для асинхронного программирования, использующий `Task`.           |
| **ThreadPool**             | Пул потоков, управляемый CLR, для повторного использования потоков.         |
| **SynchronizationContext** | Контекст, в котором продолжается выполнение кода после await (UI, ASP.NET). |

---

## 2️⃣ Потоки (Thread)

- Класс `System.Threading.Thread`
- Можно создать вручную:

```csharp
Thread t = new Thread(() => Console.WriteLine("Hello"));
t.Start();
```

- Особенности:

  - Каждый поток имеет свой стек (stack), heap общий.
  - Потоки дороже по ресурсам, чем задачи.
  - Требуется синхронизация для разделяемых данных.

---

## 3️⃣ ThreadPool

- `ThreadPool.QueueUserWorkItem` / `Task.Run()`
- Плюсы:

  - Повторное использование потоков
  - Меньше накладных расходов на создание потоков

- Минусы:

  - Ограничение по количеству потоков
  - Нельзя управлять приоритетом отдельных задач напрямую

---

## 4️⃣ Tasks (Task Parallel Library, TPL)

- Класс `Task` — представляет асинхронную или параллельную работу
- Пример:

```csharp
Task t = Task.Run(() => DoWork());
await t; // ожидание завершения
```

- Особенности:

  - `Task` — легковесная абстракция
  - Может возвращать результат (`Task<T>`)
  - Под капотом использует ThreadPool

---

## 5️⃣ Асинхронность (`async` / `await`)

- Позволяет писать асинхронный код **как синхронный**
- Пример:

```csharp
async Task<int> GetDataAsync()
{
    var result = await HttpClient.GetStringAsync("https://example.com");
    return result.Length;
}
```

- Механизм:

  1. Метод с `async` компилируется в **state machine**.
  2. При `await` метод **приостанавливается**, управление возвращается вызывающему.
  3. Когда задача завершена, выполнение продолжается на **SynchronizationContext**.

---

## 6️⃣ Многопоточность vs Асинхронность

| Понятие         | Механизм                   | Применение                              |
| --------------- | -------------------------- | --------------------------------------- |
| Многопоточность | Несколько потоков CPU      | CPU-bound задачи (вычисления)           |
| Асинхронность   | Не блокирующее ожидание IO | IO-bound задачи (запросы к сети/файлам) |

- Важно: **await не создаёт новый поток!**
- CPU-bound → лучше Task + ThreadPool / Parallel.
- IO-bound → async/await, освобождает поток для других задач.

---

## 7️⃣ Примитивы синхронизации

- Для многопоточности, чтобы избежать гонок:

  - `lock` — простой монитор
  - `Mutex` — межпроцессный
  - `Semaphore` / `SemaphoreSlim` — ограничение числа потоков
  - `Monitor.Enter/Exit` — вручную
  - `ReaderWriterLockSlim` — чтение/запись
  - `Interlocked` — атомарные операции

- Async-friendly:

  - `SemaphoreSlim.WaitAsync()`

---

## 8️⃣ Каверзные моменты и вопросы

1. **await vs Thread**

   - await не создаёт поток, поток освобождается.
   - Task.Run создаёт работу в пуле потоков.

2. **SynchronizationContext**

   - UI-thread или ASP.NET context влияет на продолжение после await.

3. **Deadlocks**

   - Часто при `.Result` или `.Wait()` на UI или ASP.NET вызывается deadlock.

4. **Thread safety**

   - Любые shared data → lock или concurrent collections.

5. **Task vs Thread**

   - Task — легковесный, может использовать поток из пула.
   - Thread — тяжелый, отдельный стек.

6. **ConfigureAwait(false)**

   - Продолжение после await не обязательно на исходном контексте → предотвращает deadlock, повышает производительность.

7. **Async all the way**

   - Если метод async вызывает синхронный блокирующий код, эффективность падает.

8. **CancellationToken**

   - Позволяет отменять задачи асинхронно.

---

## 9️⃣ Пример: CPU-bound vs IO-bound

**CPU-bound:**

```csharp
var tasks = new Task[4];
for(int i=0;i<4;i++)
    tasks[i] = Task.Run(() => ComputeHeavy());
Task.WaitAll(tasks);
```

**IO-bound:**

```csharp
async Task FetchDataAsync()
{
    var data = await httpClient.GetStringAsync("https://example.com");
    Console.WriteLine(data);
}
await FetchDataAsync(); // не блокирует поток
```

---

## 🔟 Таблица: асинхронность vs многопоточность

| Характеристика          | Асинхронность                   | Многопоточность                   |
| ----------------------- | ------------------------------- | --------------------------------- |
| Цель                    | Не блокировать поток при IO     | Использовать CPU параллельно      |
| Поток                   | Один поток, может освобождаться | Несколько потоков одновременно    |
| Использование           | `async/await`, `Task`           | `Thread`, `ThreadPool`, `Task`    |
| Примитивы синхронизации | SemaphoreSlim, async locks      | lock, Mutex, Monitor, Interlocked |
| Deadlock                | Возможен при sync-over-async    | Возможен при неправильном lock    |
| Производительность      | Эффективнее для IO              | Эффективнее для CPU-heavy         |

---

**[⬆ Вернуться к главной](../index.md)**

<p align="center">✨Dvurechensky✨</p>
