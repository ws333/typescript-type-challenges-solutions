---
id: 1097
title: IsUnion
lang: uk
level: medium
tags: union
---

## Завдання

Реалізувати тип `IsUnion`, який приймає тип-параметр `T` й повертає `true`, якщо `T` це об'єднання типів.
Наприклад:

```typescript
type case1 = IsUnion<string> // false
type case2 = IsUnion<string | number> // true
type case3 = IsUnion<[string | number]> // false
```

## Розв'язок

Коли я побачив цю проблему, то не знав навіть з чого почати.
Тому що немає рішення в TypeScript, яке б можна було використати для реалізації такого типу.
Немає ніяких вбудованих типів, які б, хоч якось допомогли.

Тому саме час вмикати креативне мислення й використовувати підручні засоби.
Почнемо з того, що подумаємо про об'єднання типів і як вони представлені.

Коли ви вказуєте плоский тип, наприклад `string`, то значеннями цього типу не буде нічого іншого, крім `string`.
Але, якщо ви вказуєте об'єднання типів, наприклад `string | number`, то отримуєте можливі значення як `string`, так і `number`.

Плоскі типи представляють тільки один набір можливих значень, а об'єднання представляють набір із наборів можливих значень.
І немає ніякого сенсу в дистрибутивному обході для плоских типів, але є — для об'єднань типів.

І цю ключову точку ми й візьмемо за фактор, як можна визначити об'єднання типів.
Коли ми дистрибутивно перебираємо тип `T`, який не є об'єднанням, це нічого не змінює.
Але, це змінює багато для об'єднань типів.

У TypeScript є одна можливість мови — дистрибутивні умовні типи.
Коли ви пишете конструкцію `T extends string ? true : false`, де `T` - це об'єднання, TypeScript застосує умовний тип, до кожного элементу з об'єднання, дистрибутивно.
Грубо кажучи, це буде виглядати так.

```typescript
type IsString<T> = T extends string ? true : false

// Наприклад, передаємо параметр T = string | number
// Дистрибутивне застосування умовного типу буде виглядати якось так
type IsStringDistributive =
    string extends string ? true : false
  | number extends string ? true : false
```

Бачите до чого я веду?
Якщо тип `T` це об'єднання, то, використовуючи дистрибутивні умовні типи, можна розбити його й порівняти з вхідним тип-параметром `T`.
У випадку, якщо вони однакові — це не було об'єднання.
Але, якщо вони неоднакові, то це — об'єднання.
Через те, що `string` не дорівнюватиме `string | number`, й так само, `number` не дорівнюватиме `string | number`.

Почнімо з реалізації!
Спочатку, зробимо копію вхідного параметра `T`, щоб порівняти з початковим об'єднанням без змін.

```typescript
type IsUnion<T, C = T> = never
```

Застосовуючи умовні типи, отримуємо дистрибутивну семантику.
Всередині правдивої гілки умовного типу, отримаємо кожен елемент з об'єднання окремо.

```typescript
type IsUnion<T, C = T> = T extends C ? never : never
```

Тепер, важлива частина — порівняти елемент з об'єднання з початковим вхідним тип-параметром `T`.
У випадку, якщо ці типи будуть однакові, то це означає, що в обох випадках було по одному елементу — не об'єднання.
Інакше, дистрибутивні умовні типи зробили свою справу й ми порівнюємо один елемент з об'єднання з початковим об'єднанням — тобто це об'єднання.

```typescript
type IsUnion<T, C = T> = T extends C ? [C] extends [T] ? false : true : never
```

Готово!
Щоб пояснити момент порівняння, подивимося, які типи містять в собі `[C]` і `[T]` всередині дистрибутивного умовного типу.
Коли ми передаємо тип, який не є об'єднанням, наприклад `string`, вони містять однакові типи.
Відповідно, це не об'єднання — повертаємо `false`.

```typescript
[T] = [string]
[C] = [string]
```

Але, якщо передамо об'єднання, наприклад `string | number`, вони містять різні типи.
Наша копія `C` містить в собі кортеж з нашим початковим об'єднанням, а `T` - об'єднання з кортежів.
Відповідно, типи різні, вважаємо це об'єднанням й повертаємо `true`.

```typescript
[T] = [string] | [number]
[C] = [string | number]
```

## Посилання

- [Об'єднання типів](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types)
- [Дженерики](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [Умовні типи](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
- [Дистрибутивні умовні типи](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types)
- [Кортежі](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-3.html#tuple-types)