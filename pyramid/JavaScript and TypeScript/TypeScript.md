# 1.3.2 TypeScript

## 1.3.2.1 Виды типизации и их отличия

1. **Статическая vs Динамическая типизация**. Когда проверяются типы — на этапе компиляции или в runtime
2. **Сильная vs Слабая типизация**. Насколько строго язык соблюдает правила типов. 
3. **Явная vs Неявная типизация**. Должен ли программист явно указывать типы.
4. **Структурная vs Номинативная типизация**. Как определяется совместимость типов
``` Java
// Java - номинативная типизация
class Person {
    String name;
    int age;
}

class Employee {
    String name;
    int age;
}

// Несмотря на одинаковую структуру, типы несовместимы
Person person = new Employee(); // Ошибка: несовместимые типы
```

``` TypeScript
// TypeScript - структурная типизация
interface Person {
    name: string;
    age: number;
}

interface Employee {
    name: string;
    age: number;
    salary: number;
}

let person: Person = { name: "John", age: 30 };
let employee: Employee = { name: "Alice", age: 25, salary: 50000 };

// OK: Employee имеет все свойства Person
person = employee;

// Ошибка: Person не имеет свойства salary
employee = person; 
```
## 1.3.2.2 Что такое интерфейсы и type-алиасы? Отличия и когда использовать
Интерфейс — это способ описания структуры объекта. Он определяет, какие свойства и методы должен иметь объект.
Используется обычно для объектов и классов, 

Type Alias — это просто псевдоним для любого типа. Он может описывать не только объекты, но и примитивы, объединения, кортежи и т.д.
Используется обычно для объединений и пересечений, кортежей (tuples), сложных абстрактных типов

``` TypeScript
// Псевдоним для примитива
type ID = number | string;

// Псевдоним для объекта
type UserType = {
  id: ID;
  name: string;
  email: string;
};

// Псевдоним для объединения типов
type Status = "active" | "inactive" | "pending";
```
**Совместимость и производительность**

Совместимость: В большинстве случаев интерфейсы и type aliases взаимозаменяемы для объектов.

Производительность: Интерфейсы немного быстрее для больших проектов, так как создают именованный тип.
## 1.3.2.3 Тайпгарды

**Type Guards** — это выражения, которые выполняют проверку типов во время выполнения и позволяют TypeScript сузить тип переменной внутри блока кода.

Базовый пример
``` TypeScript
function printLength(value: string | number) {
  if (typeof value === "string") {
    // TypeScript знает, что здесь value - string
    console.log(value.length); // OK
  } else {
    // TypeScript знает, что здесь value - number
    console.log(value.toFixed(2)); // OK
  }
}
```
Виды type guards: 
* `typeof`
``` JS
typeof value === "string"
```
* `instanceof`
``` JS
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function handleAnimal(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark(); // animal: Dog
  } else {
    animal.meow(); // animal: Cat
  }
}
```
* `in`
``` JS
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

function move(pet: Bird | Fish) {
  if ("fly" in pet) {
    pet.fly(); // pet: Bird
  } else {
    pet.swim(); // pet: Fish
  }
}
```
* custom
``` JS
// Type Guard функция
function isString(value: any): value is string {
  return typeof value === "string";
}

function isNumber(value: any): value is number {
  return typeof value === "number" && !isNaN(value);
}

// Использование
function processInput(input: string | number) {
  if (isString(input)) {
    console.log(input.length); // input: string
  } else if (isNumber(input)) {
    console.log(input.toFixed(2)); // input: number
  }
}
```
## 1.3.2.4 Union and intersections типы
union types
``` TypeScript
// Type Aliases - легко создавать объединения
type ID = number | string;
type Status = "active" | "inactive";
```

intersections
``` TypeScript
// Interfaces - объединения не поддерживаются напрямую
// Но можно использовать intersection
interface Named {
  name: string;
}

interface Aged {
  age: number;
}

type Person = Named & Aged; // Intersection типа
interface PersonInterface extends Named, Aged {} // Эквивалент через интерфейс
```
## 1.3.2.5 Const assertion and Enum
**Const assertion** — это способ сказать TypeScript: "обрабатывай это выражение как литерал, а не как общий тип".

``` TypeScript
// Без const assertion
let status = "success"; // тип: string
status = "error"; // OK
status = "any string"; // OK

// С const assertion
let statusConst = "success" as const; // тип: "success"
statusConst = "error"; // Error: Type '"error"' is not assignable to type '"success"'
```

``` TypeScript
// Без const assertion
const user = {
  name: "John",
  age: 30,
  role: "admin" // тип: string
};

user.role = "user"; // OK - role это string

// С const assertion
const userConst = {
  name: "John",
  age: 30,
  role: "admin"
} as const; // все свойства становятся readonly с литеральными типами

userConst.role = "user"; // Error: Cannot assign to 'role' because it is a read-only property
```

``` TypeScript
// Обычный массив
const numbers = [1, 2, 3]; // тип: number[]
numbers.push(4); // OK
numbers[0] = 10; // OK

// С const assertion
const numbersConst = [1, 2, 3] as const; // тип: readonly [1, 2, 3]
numbersConst.push(4); // Error: Property 'push' does not exist on type 'readonly [1, 2, 3]'
numbersConst[0] = 10; // Error: Cannot assign to '0' because it is a read-only property
```
✅ Когда использовать Const Assertion:
* Конфигурационные объекты
* Литеральные значения которые не должны меняться
* Создание union types из объектных литералов
* Когда нужна полная иммутабельность

✅ Когда использовать Enum:
* Набор связанных констант с общим смыслом
* Когда нужен reverse mapping (для числовых enum)
* Битовые флаги и побитовые операции
* Когда важно пространство имен

## 1.3.2.6 Встроенные утилитарные типы
* `Partial<T>` - делает все свойства необязательными
* `Readonly<T>` - делает все свойства только для чтения
* `Required<T>` - делает все свойства обязательными
* `Record<T, U>` - словарь
* `Pick<T, U>` - скопировать из типа `T` поля `U`
* `Omit<T, K>` - исключает из типа `T` указанные свойства `K`
* `Exclude<T, U>` - исключает из union-типа `T` поля `U`
* `Extract<T, U>` - извлекает из типа `T` совпадающие поля типа `U`
* `NonNullable<T>` - удаляет из типа `T` `null` и `undefined`
* `ReturnType<T>` - получает тип возвращаемого значения `T`
* `Parameters<T>` - возвращает псевдомассив параметров и типов функции `T`
* `InstanceType<T>` - возвращает тип инстанса класса `T`
* `ThisType` - позволяет явно указывать тип this внутри объектов