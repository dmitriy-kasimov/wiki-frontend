# 1.3.1 JavaScript
## 1.3.1.1 Механизм замыканий

Замыкание — это функция, которая запоминает свои внешние переменные и может получить к ним доступ даже после того, как внешняя функция завершила выполнение.

Проще говоря: функция + её лексическое окружение = замыкание

```` JS
function createCounter() {
  let count = 0; // Локальная переменная внешней функции
  
  return function() {
    count++; // Внутренняя функция имеет доступ к count
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
````

**Как это работает?**
* При вызове createCounter() создается лексическое окружение с переменной count
* Возвращаемая функция "запоминает" это окружение
* После завершения createCounter() её лексическое окружение не уничтожается, потому что на него есть ссылка из возвращаемой функции

Важные особенности замыканий
* Память: Замыкания сохраняют ссылки на переменные, что может приводить к утечкам памяти
* Производительность: Создание множества замыканий может impact на производительность
* Изоляция: Каждое замыкание имеет собственное лексическое окружение

Примеры практического применения
``` JS 
// private переменная через замыкание

function createAccount(initialBalance){
    let balance = initialBalance;
    
    return({
        deposit: function(amount) {
            balance += amount;
            return balance;
        },
        withdraw: function(amount) {
            if(amount <= balance) {
                balance -= amount;
                return amount;
            }
            return null;
        },
        getBalance: function(){
            return balance;
        }
    })
}

const account1 = createAccount(0);
account1.deposit(1000);
account1.getBalance()
account1.withdraw(1500);
account1.withdraw(500);
account1.getBalance()

const account2 = createAccount(10000);
account2.deposit(0);
account2.getBalance()
account2.withdraw(1500);
account2.withdraw(500);
account2.getBalance()
```

``` JS
// каррииорование 

function add(num) {
    return function(num2){return num + num2;}
}

const add5 = add(5)
console.log(add5(3)) // 8
console.log(add(5)(3)) // 8
```

``` JS
// пример с логгером

function logger(prefix){
    return function(message){
        return `[${prefix}] ${message}`
    }
}

const infoLogger = logger("INFO")
infoLogger("system has started"); 
const warnLogger = logger("WARN")
warnLogger("founded some warnings");
```

## 1.3.1.2 Механизм передачи контекста в функции
В JavaScript контекст (значение `this`) — это объект, в контексте которого выполняется функция. Значение `this` определяется в момент вызова функции, а не в момент её объявления.

**Способы передачи контекста**
1. Implicit binding
``` JS
const person = {
  name: 'John',
  greet: function() {
    console.log(`Hello, my name is ${this.name}`);
  }
};

person.greet(); // "Hello, my name is John" - this = person
```
2. Explicit binding (`call`, `apply`)
``` JS
function introduce(age, city){
    console.log(`Hello, my name is ${this.name}, ${age} years old from ${city}`)
}

const person1 = { name: 'Bob' };
const person2 = { name: 'Alice' };

introduce.call(person1, 25, 'Moscow') // Hello, my name is Bob, 25 years old from Moscow
introduce.apply(person1, [30, 'London']) // Hello, my name is Alice, 30 years old from London
```
3. Hard binding(`bind`)
``` JS
function introduce(age, city){
    console.log(`Hello, my name is ${this.name}, ${age} years old from ${city}`)
}

const person1 = { name: 'Bob' };
const boundedIntroduce = introduce.bind(person1, 25, 'Minsk') // Hello, my name is Bob, 25 years old from Minsk
boundedIntroduce.call(person1, 30, 'Kiev') // Hello, my name is Bob, 25 years old from Minsk
```
## 1.3.1.3 Прототипное наследование

**Прототипное наследование** — это механизм, при котором объекты могут наследовать свойства и методы от других объектов-прототипов. В отличие от классического наследования (как в Java/C++), в JavaScript наследование реализуется через цепочку прототипов.

Доступ к прототипу через свойство `__proto__`
## 1.3.1.4 TC39 process

* Stage 1: Proposal - Формальное предложение
* Stage 2: Draft - Черновик спецификации
* Stage 3: Candidate - Кандидат
* Stage 4: Finished - Завершено

### Частота выпусков
Ежегодный цикл: Новые features добавляются в спецификацию ECMAScript каждый год в июне.

* ES2015 (ES6): Классы, промисы, стрелочные функции
* ES2016: `Array.prototype.includes`, оператор `**`
* ES2017: `async`/`await`, `Object.values()`, `Object.entries()`
* ES2018: Rest/Spread properties, `Promise.finally()`
* ES2019: `Array.flat()`, `Object.fromEntries()`
* ES2020: Optional chaining (`?.`), Nullish coalescing (`??`)
* ES2021: `String.replaceAll()`, Logical assignment (`||=`, `&&=`, `??=`)
* ES2022: Top-level `await`, `.at()` method, Class fields
* ES2023: `Array.findLast()`, `Array.findLastIndex()`
* ES2024: Новые типы данных `Tuple` и `Record`, Decorators
* ES2025: импорт JSON как полноценного модуля, без fetch/axios/require; Set 2.0: `.union()`, `.intersection()`, `.difference()`, `.isSubsetOf()`
## 1.3.1.5 Асинхронность
### `requestAnimationFrame` (rAF)

Это инструмент для создания плавных анимаций и любых визуальных обновлений. 

`requestAnimationFrame(fn)` вызывает fn перед каждой перерисовкой кадра.

**Базовый паттерн использования**

Запуск анимации
``` JS
function animate() {
  // ... код, который изменяет стили/свойства ...
  element.style.left = newPosition + 'px';

  // Запрашиваем СЛЕДУЮЩИЙ кадр анимации!
  requestAnimationFrame(animate);
}

// ЗАПУСК анимации
requestAnimationFrame(animate);
```

Остановка анимации
``` JS
let animationId;

function startAnimation() {
  // ...
  animationId = requestAnimationFrame(animate);
}

function stopAnimation() {
  cancelAnimationFrame(animationId);
}

```

**Ключевой параметр:** `timestamp`

Функция-колбэк, которую вы передаете в `rAF`, автоматически получает один аргумент — `timestamp` (метка времени). Это высокоточное значение (в миллисекундах), равное тому, что возвращает `performance.now()`.
Чтобы вычислять дельту времени между кадрами. Это критически важно для создания независимых от частоты кадров анимаций.

``` JS
let lastTime = 0;
const speed = 200; // пикселей в секунду

function animate(timestamp) {
  // Вычисляем, сколько времени прошло с прошлого кадра
  const deltaTime = timestamp - lastTime;
  lastTime = timestamp;

  // Сдвигаем элемент на расстояние, зависящее от времени, а не от кадров
  // (200 px/sec * deltaTime in seconds) = (200 * (deltaTime / 1000))
  element.x += speed * (deltaTime / 1000);

  requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```
Теперь блок будет двигаться со скоростью 200 пикселей в секунду на любом устройстве, независимо от того, работает оно на 30 FPS или на 144 FPS.
