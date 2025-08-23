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

## 1.3.1.4 TC39 process

## 1.3.1.5 Асинхронность
