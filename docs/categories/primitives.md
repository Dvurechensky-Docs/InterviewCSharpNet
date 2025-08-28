---
layout: default
title: '⏳ Какие бывают примитивы синхронизации в .NET?'
description: ''
author: 'Dvurechensky'
date: 2025-08-28
published: true
tags:
  - primitives
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
  - [2️⃣ `lock` (Monitor)](#2️⃣-lock-monitor)
  - [3️⃣ `Mutex` (System.Threading.Mutex)](#3️⃣-mutex-systemthreadingmutex)
  - [4️⃣ `Semaphore` и `SemaphoreSlim`](#4️⃣-semaphore-и-semaphoreslim)
    - [Semaphore](#semaphore)
    - [SemaphoreSlim](#semaphoreslim)
  - [5️⃣ `AutoResetEvent` и `ManualResetEvent`](#5️⃣-autoresetevent-и-manualresetevent)
  - [6️⃣ `Monitor.Wait()` / `Monitor.Pulse()` / `PulseAll()`](#6️⃣-monitorwait--monitorpulse--pulseall)
  - [7️⃣ ReaderWriterLockSlim](#7️⃣-readerwriterlockslim)
  - [8️⃣ Interlocked](#8️⃣-interlocked)
  - [9️⃣ Каверзные моменты и подводные камни](#9️⃣-каверзные-моменты-и-подводные-камни)
  - [🔟 Таблица примитивов](#-таблица-примитивов)

**[⬆ Вернуться к главной](../index.md)**

---

## 1️⃣ Основная идея

- **Примитивы синхронизации** нужны для:

  - **Предотвращения гонок данных (race conditions)**.
  - **Согласования доступа к общим ресурсам**.
  - **Организации взаимодействия между потоками** (thread coordination).

- В .NET есть **несколько уровней примитивов**: от простых до более сложных, включая асинхронные варианты.

---

## 2️⃣ `lock` (Monitor)

- Самый базовый способ синхронизации.
- Синтаксис:

```csharp
private readonly object _lock = new object();

lock(_lock)
{
    // критическая секция
}
```

- Под капотом использует **`Monitor.Enter` / `Monitor.Exit`**.
- Особенности:

  - Позволяет **только одному потоку** войти в критическую секцию.
  - **Блокировка рекурсивная**: один поток может войти несколько раз.
  - В сочетании с `Monitor.Wait()` и `Monitor.Pulse()` можно делать условные ожидания.

---

## 3️⃣ `Mutex` (System.Threading.Mutex)

- **Межпроцессный примитив**: может использоваться для синхронизации потоков **между процессами**.
- Пример:

```csharp
Mutex mutex = new Mutex();
mutex.WaitOne(); // захват
try
{
    // критическая секция
}
finally
{
    mutex.ReleaseMutex();
}
```

- Особенности:

  - Более дорогой, чем `lock`.
  - Используется редко для потоков в одном процессе; чаще для межпроцессной синхронизации.
  - Может быть **named** для нескольких процессов.

---

## 4️⃣ `Semaphore` и `SemaphoreSlim`

### Semaphore

- Ограничивает **количество потоков**, одновременно заходящих в секцию.
- Пример:

```csharp
Semaphore sem = new Semaphore(2, 5); // максимум 2 потока одновременно
sem.WaitOne();
// критическая секция
sem.Release();
```

### SemaphoreSlim

- Более лёгкий, только для одного процесса.
- Поддерживает **асинхронные методы**:

  - `WaitAsync()`

- Отличие от обычного Semaphore:

  - Быстрее, меньше накладных расходов.
  - Не межпроцессный.

---

## 5️⃣ `AutoResetEvent` и `ManualResetEvent`

- **События (events)** для сигнализации между потоками.
- **AutoResetEvent**

  - Сбрасывается автоматически после одного потока.
  - Используется для **одиночного уведомления**.

- **ManualResetEvent**

  - Сохраняет сигнал до явного сброса (`Reset()`).
  - Можно пропускать несколько потоков после сигнала.

- Пример:

```csharp
AutoResetEvent are = new AutoResetEvent(false);

Thread t = new Thread(() =>
{
    Console.WriteLine("Waiting...");
    are.WaitOne(); // ждёт сигнала
    Console.WriteLine("Signaled!");
});

t.Start();
Thread.Sleep(1000);
are.Set(); // отправка сигнала
```

---

## 6️⃣ `Monitor.Wait()` / `Monitor.Pulse()` / `PulseAll()`

- Используются **в сочетании с lock/Monitor**.
- `Wait()` — поток ждёт сигнала, отпуская lock.
- `Pulse()` — сигнал одному ожидающему потоку.
- `PulseAll()` — сигнал всем ожидающим потокам.
- Пример:

```csharp
lock(_lock)
{
    while(!condition) Monitor.Wait(_lock);
    // обработка
    Monitor.Pulse(_lock);
}
```

- Часто используется для **producer/consumer pattern**.

---

## 7️⃣ ReaderWriterLockSlim

- Позволяет:

  - **Много потоков для чтения**.
  - **Один поток для записи**.

- Пример:

```csharp
ReaderWriterLockSlim rwLock = new ReaderWriterLockSlim();

// Чтение
rwLock.EnterReadLock();
try { /* чтение */ } finally { rwLock.ExitReadLock(); }

// Запись
rwLock.EnterWriteLock();
try { /* запись */ } finally { rwLock.ExitWriteLock(); }
```

- Эффективно при частых чтениях и редких записях.

---

## 8️⃣ Interlocked

- Простые **атомарные операции** с числами.
- Методы:

  - `Interlocked.Increment(ref x)`
  - `Interlocked.Decrement(ref x)`
  - `Interlocked.Add(ref x, value)`
  - `Interlocked.Exchange(ref x, newValue)`
  - `Interlocked.CompareExchange(ref x, newValue, comparand)`

- Используется для **микросинхронизации без блокировок**.
- Быстро, но ограничено на простые типы (`int`, `long`, `IntPtr`).

---

## 9️⃣ Каверзные моменты и подводные камни

1. **lock vs Monitor**

   - `lock` — синтаксический сахар для `Monitor.Enter/Exit`.
   - Нужно **никогда не блокировать на публичных объектах** (возможны deadlock/внешние вмешательства).

2. **Mutex vs Semaphore**

   - Mutex → дорого, межпроцессный.
   - SemaphoreSlim → лёгкий, для одного процесса.

3. **Deadlock**

   - Два потока ждут друг друга → взаимная блокировка.
   - Часто при nested lock / комбинировании нескольких примитивов.

4. **Async / await**

   - Для асинхронных методов:

     - `SemaphoreSlim.WaitAsync()`
     - `AsyncLock` (от сторонних библиотек)

   - `lock` не совместим с `await` внутри, нужен `AsyncLock`.

5. **Производительность**

   - `Interlocked` быстрее всех.
   - `lock` медленнее, но проще и безопаснее.
   - `Mutex` самый дорогой.

---

## 🔟 Таблица примитивов

| Примитив             | Scope              | Особенности                    | Поддержка async |
| -------------------- | ------------------ | ------------------------------ | --------------- |
| lock / Monitor       | Один процесс       | Рекурсивный, блокирует поток   | Нет             |
| Mutex                | Межпроцессный      | Можно именовать, дорогой       | Нет             |
| Semaphore            | Один/Межпроцессный | Ограничение N потоков          | Нет             |
| SemaphoreSlim        | Один процесс       | Лёгкий, поддержка async        | Да              |
| AutoResetEvent       | Один процесс       | Сигнал одному потоку           | Нет             |
| ManualResetEvent     | Один процесс       | Сигнал всем потокам            | Нет             |
| ReaderWriterLockSlim | Один процесс       | Много читателей, один писатель | Нет             |
| Interlocked          | Один процесс       | Атомарные операции с числами   | Да (частично)   |

---

**[⬆ Вернуться к главной](../index.md)**

<p align="center">✨Dvurechensky✨</p>
