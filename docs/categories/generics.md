---
layout: default
title: '🧩 Как работают дженерики (generics) в .NET?'
description: ''
author: 'Dvurechensky'
date: 2025-08-28
published: true
tags:
  - generics
  - C#
---

<h1 align="center">🧩 Как работают дженерики (generics) в .NET?</h1>
<p align="center">
    <a href="https://git.io/typing-svg"><img src="https://readme-typing-svg.demolab.com?font=Fira+Code&pause=1000&center=true&vCenter=true&width=435&lines=%D0%9F%D0%BE%D0%B2%D1%82%D0%BE%D1%80%D0%B5%D0%BD%D0%B8%D0%B5+-+%D0%BC%D0%B0%D1%82%D1%8C+%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D1%8F" alt="Typing SVG" /></a>
</p>
<p align="center">
    <a href="https://sites.google.com/view/dvurechensky" target="_blank"><img alt="Static Badge" src="https://shields.dvurechensky.pro/badge/Dvurechensky-Nikolay-blue"></a>
</p>

# ✨ Оглавление

- [✨ Оглавление](#-оглавление)
  - [🔹 Что такое generics (коротко)](#-что-такое-generics-коротко)
  - [🔹 Синтаксис — базовые формы](#-синтаксис--базовые-формы)
  - [🔹 Reified generics (не как в Java) — что это даёт](#-reified-generics-не-как-в-java--что-это-даёт)
  - [🔹 Как CLR/JIT реализует generics (важно для производительности)](#-как-clrjit-реализует-generics-важно-для-производительности)
  - [🔹 Статические поля в generic-классе](#-статические-поля-в-generic-классе)
  - [🔹 Ограничения (constraints)](#-ограничения-constraints)
  - [🔹 default(T) и поведение для разных T](#-defaultt-и-поведение-для-разных-t)
  - [🔹 Variance — ковариантность и контрвариантность](#-variance--ковариантность-и-контрвариантность)
  - [🔹 Generic methods vs generic types](#-generic-methods-vs-generic-types)
  - [🔹 Reflection и generics](#-reflection-и-generics)
  - [🔹 IL и вызовы на value-types (коротко, углубление)](#-il-и-вызовы-на-value-types-коротко-углубление)
  - [🔹 Практические советы и производительность](#-практические-советы-и-производительность)
  - [🔹 Частые подводные камни (gotchas)](#-частые-подводные-камни-gotchas)
  - [🔹 Короткие «правильные» ответы на часто задаваемые собес-вопросы](#-короткие-правильные-ответы-на-часто-задаваемые-собес-вопросы)
  - [🔹 Примеры + шаблоны (полезно на собесе)](#-примеры--шаблоны-полезно-на-собесе)
  - [🔹 «Каверзные» вопросы, которые могут задать — и краткий ответ](#-каверзные-вопросы-которые-могут-задать--и-краткий-ответ)
  - [🔹 Ещё пара продвинутых тем (если интервьюер уперся)](#-ещё-пара-продвинутых-тем-если-интервьюер-уперся)
  - [🔹 Резюме (что нужно знать джуну/мидлу/сеньору)](#-резюме-что-нужно-знать-джунумидлусеньору)

**[⬆ Вернуться к главной](../index.md)**

---

## 🔹 Что такое generics (коротко)

`Generics` — это параметризованные типы и методы. Вместо конкретного типа вы пишете параметр типа `T` (или несколько), и этот код работает для **любого** типа, соответствующего ограничениям. Примеры: `List<T>`, `Dictionary<TKey, TValue>`, `void Swap<T>(ref T a, ref T b)`.

**Зачем:** типобезопасность + отказ от бокcинга для value-types + переиспользуемость кода.

---

## 🔹 Синтаксис — базовые формы

```csharp
// generic класс
public class Box<T>
{
    public T Value { get; set; }
}

// generic метод
public static void Swap<T>(ref T a, ref T b) { var t = a; a = b; b = t; }

// generic интерфейс
public interface IRepository<T> { void Add(T item); T Get(int id); }
```

При вызове компилятор обычно выводит `T` автоматически:

```csharp
var b = new Box<int> { Value = 5 }; // или просто new Box<int>()
Swap<int>(ref x, ref y); // тип можно явно указать
```

---

## 🔹 Reified generics (не как в Java) — что это даёт

- В .NET generics **не стираются** (they are _reified_). Это значит, что информация о типах-параметрах доступна в runtime: `typeof(List<int>)` и `typeof(List<string>)` — разные типы в отражении.
- Это отличается от Java, где generic type arguments стираются (type erasure).

**Практическое следствие:**

- Можно создавать `typeof(List<>)` → `MakeGenericType(typeof(int))`.
- Можно делать `default(T)`, `typeof(T)`, `T[]` и т.д. на уровне IL/runtime.

---

## 🔹 Как CLR/JIT реализует generics (важно для производительности)

- CLR хранит _generic type definition_ и _constructed types_ (определение + конкретные аргументы).
- **Для value-types (struct)** JIT обычно **генерирует отдельный нативный код** для каждой конкретной инстанциации (например, `List<int>` и `List<double>` — разные скомпилированные версии). Это даёт **отсутствие бокcинга** и высокую производительность.
- **Для reference-types** CLR может **шарить (share) JIT-код** между разными reference-type аргументами (чтобы не дублировать код), но при этом _метаданные и identity типов всё равно различаются_.
  → Важно: реализация может различаться по деталям в разных рантаймах, но концепция — отдельный код для value-types, шаринг для ссылочных — общая.

**Следствие:**

- Generics помогают **избежать бокcинга** value-types (например `List<int>` хранит int без boxing).
- Но множество instantiations для value-types может увеличить размер нативного кода (code bloat).

---

## 🔹 Статические поля в generic-классе

- **Статические поля привязаны к конкретной constructed-инстанции**. То есть `StaticHolder<int>.X` и `StaticHolder<string>.X` — разные поля.
- Даже если JIT шарит код для reference-types, статические поля всё равно хранятся отдельно для каждого closed type.

---

## 🔹 Ограничения (constraints)

`where T : ...` — позволяет накладывать ограничения на типы-параметры.

Частые варианты:

- `where T : class` — ссылка (reference type).
- `where T : struct` — **non-nullable value type** (заметь: `int?` не допускается).
- `where T : new()` — должен иметь публичный конструктор без параметров.
- `where T : SomeBaseClass` — наследник указанного класса.
- `where T : ISomeInterface` — реализует интерфейс.
- `where T : unmanaged` — blittable unmanaged type (нет управляемых ссылок).
- `where T : notnull` — запрещает `null` (включая nullable reference types).
- `where T : System.Enum` / `where T : System.Delegate` — специальные ограничения для enum/delegate.

Пример:

```csharp
public T CreateInstance<T>() where T : class, new()
{
    return new T();
}
```

**Частые вопросы:**

- Почему нельзя `new T()` без `new()`? — Компилятор не гарантирует, что `T` имеет публичный параметр `less ctor`.
- `where T : struct` — запрещает Nullable<T> (т.е. `int?` не подойдёт).

---

## 🔹 default(T) и поведение для разных T

- `default(T)` даёт `null` для ссылочных типов, нулевое значение (`0`, `false`, `'\0'`) для value-types.
- С C# 7.1/8 можно писать `default` сокращённо в коде (контекстно).

---

## 🔹 Variance — ковариантность и контрвариантность

- Объявления `out` и `in`:

  - `out T` — ковариантность (можно присваивать `IEnumerable<string>` в `IEnumerable<object>`).
  - `in T` — контрвариантность (для делегатов/интерфейсов-«потребителей», например `IComparer<in T>`).

- **Работает только для интерфейсов и делегатов**, только для **reference types**.
- **Нельзя** применять `out/in` к классам.

Пример:

```csharp
IEnumerable<string> ss = new List<string>();
IEnumerable<object> oo = ss; // OK — IEnumerable<out T> ковариантен
```

Проверки безопасности:

- Для `out T` нельзя принимать `T` в качестве параметра метода — только возвращать/выдавать.
- Для `in T` нельзя возвращать `T` — только принимать.

---

## 🔹 Generic methods vs generic types

- Метод может иметь свои generic параметр(ы), независимо от типа:

```csharp
public void Do<T>(T item) { ... }
```

- Параметры метода выводятся компилятором по аргументам. Если вывод невозможен — можно явно указать `Do<int>(5)`.

---

## 🔹 Reflection и generics

- `typeof(List<>)` — открытый generic type definition.
- `typeof(List<int>)` — constructed (closed) generic type.
- API: `Type.IsGenericType`, `Type.IsGenericTypeDefinition`, `GetGenericArguments()`, `GetGenericTypeDefinition()`, `MakeGenericType(...)`.
- Для методов: `MethodInfo.IsGenericMethodDefinition`, `MakeGenericMethod(...)`.

Пример:

```csharp
var open = typeof(Dictionary<,>);
var closed = open.MakeGenericType(typeof(string), typeof(int));
```

---

## 🔹 IL и вызовы на value-types (коротко, углубление)

- IL имеет инструкцию `constrained.` — помогает вызывать интерфейсные/виртуальные методы на value-types без бокcинга (runtime знает, как это делать).
- Это часть «как CLR избегает бокcинга при вызовах интерфейсов у value-types».

---

## 🔹 Практические советы и производительность

- **Используйте generics**, когда нужен типобезопасный контейнер/алгоритм — особенно для value-types (избегаете boxing).
- **Будьте осторожны с числом разных value-type instantiations** — может быть много сгенерированного нативного кода.
- `List<T>` vs `Array` — array of T удобен и быстрый; generic коллекции лучше для абстракций.
- `new T()` — требует `new()` constraint; если нужен non-public ctor — нельзя.

---

## 🔹 Частые подводные камни (gotchas)

- **Covariance/contravariance только для интерфейсов и делегатов** — многие путают с возможностью присваивать `List<string>` в `List<object>` (это **нельзя**).
- **Массивы ковариантны**, но небезопасны: `string[]` → `object[]` разрешено, но при попытке записать `object` в `string[]` будет `ArrayTypeMismatchException`.
- **`where T : struct` ≠ `where T : new()`** — разные семантики.
- **Generic и virtual**: поведение виртуальных методов в generic-типах обычное, но реализация JIT-а может повлиять на инлайнинг.
- **Boxing при приведении к non-generic интерфейсу**: если `T` — value type и вы приводите `List<T>` к `IList`/`IEnumerable` без generic, могут быть бокcинги/обёртки.
- **`Nullable<T>` и `where T : struct`** — `Nullable<T>` НЕ проходит `where T : struct` (как правило), потому интервьюер может спросить это как ловушку.
- **Статические поля в generic** — люди ожидают один экземпляр поля на тип, но забывают, что каждый closed-generic имеет своё статическое поле.

---

## 🔹 Короткие «правильные» ответы на часто задаваемые собес-вопросы

(коротко — для использования в интервью)

- **Что дает generics?** — Типобезопасность и производительность (избегаем boxing для value-types).
- **Generics стираются в runtime?** — Нет. .NET generics — reified: информация о типах доступна в runtime.
- **JIT генерирует код для каждого типа?** — Для value-types — как правило да; для ссылочных типов — код может шариться.
- **Можно ли `new T()`?** — Только с `where T : new()` constraint.
- **Что такое covariance/contravariance?** — `out`/`in` — позволяет безопасно преобразовать generic интерфейсы/делегаты между совместимыми reference-типами.
- **Можно ли использовать `==` для `T`?** — Нет гарантии: `==` действует для тех типов, где он определён; лучше использовать `EqualityComparer<T>.Default.Equals(a,b)`.
- **Static field generic class — один или много?** — Статическое поле у generic класса хранится отдельно для каждой closed constructed type.

---

## 🔹 Примеры + шаблоны (полезно на собесе)

**Generic repository**

```csharp
public interface IRepository<T> where T : IEntity
{
    void Add(T entity);
    T? Get(int id);
}
```

**Generic singleton (статик per-T)**

```csharp
public static class SingletonHolder<T> where T : new()
{
    public static readonly T Instance = new T();
}
```

**Covariant interface**

```csharp
public interface IReadOnlyList<out T>
{
    T Get(int index); // только возвращаем T — разрешено
}
```

**Generic method with interface constraint**

```csharp
public T Max<T>(T a, T b) where T : IComparable<T>
    => a.CompareTo(b) >= 0 ? a : b;
```

---

## 🔹 «Каверзные» вопросы, которые могут задать — и краткий ответ

(сформулируй на собесе коротко — и можешь раскрыть, если попросят)

1. **«Generics в .NET — стираются или реальные?»**
   ⇒ Реальные (reified). Метаданные сохраняют параметры типов.

2. **«Почему List<int> быстрее List<object> при хранении int?»**
   ⇒ Потому что нет бокcинга: `List<int>` хранит реальные int, JIT сгенерировал код для value-type.

3. **«Можно ли присвоить List<string> переменной List<object>?»**
   ⇒ Нельзя. Generic классы не ковариантны. Используйте `IEnumerable<out T>` для ковариантности чтения.

4. **«Сколько статических полей у GenericClass<T>?»**
   ⇒ По одному на _каждую_ constructed-инстанцию (GenericClass<int> и GenericClass<string> — свои поля).

5. **«Можно ли использовать nullable типы с where T : struct?»**
   ⇒ Нет — `where T : struct` запрещает nullable value types (ожидается non-nullable value type).

6. **«Что такое constrained. в IL?»**
   ⇒ Это механизм CLR, позволяющий вызывать виртуальные/интерфейсные методы на value-type без бокcинга.

7. **«Почему иногда JIT шарит код для reference types?»**
   ⇒ Для экономии нативного кода: одна версия кода способна обслуживать разные ссылочные типы.

8. **«Можно ли new T\[size] внутри generic метода?»**
   ⇒ Да, `new T[n]` разрешён (создаётся массив элементов типа T).

9. **«Equality for T — как правильно?»**
   ⇒ Используйте `EqualityComparer<T>.Default` — он корректно обрабатывает value/reference types и пользовательские overrides.

10. **«Почему generic constraints влияют на производительность?»**
    ⇒ Потому что наличие конкретных интерфейсных/методных ограничений позволяет компилятору/CLR генерировать эффективные вызовы (и избежать рефлексии/boxing).

---

## 🔹 Ещё пара продвинутых тем (если интервьюер уперся)

- **Generic covariance/contravariance и делегаты**: `Func<out T>` — ковариантен в возвращаемом типе; `Action<in T>` — контрвариантен в параметре.
- **Generic virtual methods & VTable**: виртуальные вызовы в generic-типах работают как обычно, но вхождение JIT может влиять на inlining.
- **Generic constraints + runtime checks**: CLR выполняет проверки типа при создании constructed type — если ограничение не выполняется, будет `TypeLoadException`.
- **Новые ограничения (unmanaged, notnull, Enum/Delegate)**: дают дополнительную безопасность и оптимизации.

---

## 🔹 Резюме (что нужно знать джуну/мидлу/сеньору)

- **Junior**: зачем generics, базовый синтаксис, `new()`/`class`/`struct` constraints, почему generics убирают boxing, `List<T>`/`Dictionary<TKey,TValue>`.
- **Middle**: reified nature, JIT behavior (value vs reference), static fields per instantiation, variance basics, reflection with generics, common pitfalls (covariance vs arrays).
- **Senior**: IL (`constrained.`), performance tradeoffs (code bloat vs speed), advanced constraints (`unmanaged`, `notnull`), generics + interop/unsafe, memory & JIT internals.

---

**[⬆ Вернуться к главной](../index.md)**

<p align="center">✨Dvurechensky✨</p>
