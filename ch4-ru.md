# Chapter 4: Currying
# Глава 4: Каррирование

## Can't live if livin' is without you
## Без тебя мне жизнь не мила

My Dad once explained how there are certain things one can live without until one acquires them. A microwave is one such thing. Smart phones, another. The older folks among us will remember a fulfilling life sans internet. For me, currying is on this list.

Мой отец однажды рассказал мне про то, что есть вещи, без которых легко живёшь, пока у тебя их нет. Например, микроволновка или смартфон. Люди постарше вполне счастливо жили и без интернета. Для меня каррирование — одна из таких вещей.

The concept is simple: You can call a function with fewer arguments than it expects. It returns a function that takes the remaining arguments.

Идея очень простая: вы можете вызвать функцию с меньшим количеством аргументов, чем она принимает, в ответ вы получите функцию, которая принмает оставшиеся аргументы.

You can choose to call it all at once or simply feed in each argument piecemeal.

Вы можете как вызвать функцию со всем аргументами сразу, так и с любым меньшим количеством.

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

Here we've made a function `add` that takes one argument and return a function. By calling it, the returned function remembers the first argument from then on via the closure. Calling it with both arguments all at once is a bit of a pain, however, so we can use a special helper function called `curry` to make defining and calling functions like this easier.

В этом примере мы определили функцию `add`, которая принимает один аргумент и возвращает функцию. Если мы вызовем функцию `add`, то она сохранит первый аргумент с помощью замыкания. Для того, чтобы определять и вызывать подобные функции было проще, мы воспользуемся функцией `curry`.

Let's setup a few curried functions for our enjoyment.

Давайте же определим несколько каррированных функций в своё удовольствие.

```js
var curry = require('lodash.curry');

var match = curry(function(what, str) {
  return str.match(what);
});

var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement);
});

var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var map = curry(function(f, ary) {
  return ary.map(f);
});
```

The pattern I've followed is a simple, but important one. I've strategically positioned the data we're operating on (String, Array) as the last argument. It will become clear as to why upon use.

В определении всех этих функций я придерживался простого, но очень важного правила: я записывал последним аргументом переменную, содержащую данные, которыми мы собираемся оперировать. Пойзже вы поймёте, зачем я это сделал.

```js
match(/\s+/g, "hello world");
// [ ' ' ]

match(/\s+/g)("hello world");
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces("hello world");
// [ ' ' ]

hasSpaces("spaceless");
// null

filter(hasSpaces, ["tori_spelling", "tori amos"]);
// ["tori amos"]

var findSpaces = filter(hasSpaces);
// function(xs) { return xs.filter(function(x) { return x.match(/\s+/g) }) }

findSpaces(["tori_spelling", "tori amos"]);
// ["tori amos"]

var noVowels = replace(/[aeiou]/ig);
// function(replacement, x) { return x.replace(/[aeiou]/ig, replacement) }

var censored = noVowels("*");
// function(x) { return x.replace(/[aeiou]/ig, "*") }

censored("Chocolate Rain");
// 'Ch*c*l*t* R**n'
```

What's demonstrated here is the ability to "pre-load" a function with an argument or two in order to receive a new function that remembers those arguments.

Здесь я продемонстрировал способность «предзагрузки» функции с аргументом или парой для того, чтобы получить новую функцию, которая запомнит эти аргументы.

I encourage you to `npm install lodash`, copy the code above and have a go at it in the repl. You can also do this in a browser where lodash or ramda is available.

Советую вам запустить `npm install lodash` в консоли, скопировать код выше и...

## More than a pun / special sauce
## Больше, чем просто приправа

Currying is useful for many things. We can make new functions just by giving our base functions some arguments as seen in `hasSpaces`, `findSpaces`, and `censored`.

Каррирование полезно во многих случаях, с его помощью мы можем создавать новые функции на лету, просто передавая меньше аргументов в уже существующие, как мы могли убедиться в случаях с `hasSpaces`, `findSpaces`, и `censored`.

We also have the ability to transform any function that works on single elements into a function that works on arrays simply by wrapping it with `map`:

Мы также можем преобразовать любую функцию, которая принимает один аргумент в функцию, принимающую массив. Для этого нам нужно просто обернуть её в `map`:

```js
var getChildren = function(x) {
  return x.childNodes;
};

var allTheChildren = map(getChildren);
```

Giving a function fewer arguments than it expects is typically called *partial application*. Partially applying a function can remove a lot of boiler plate code. Consider what the above `allTheChildren` function would be with the uncurried `map` from lodash[^note the arguments are in a different order]:

Вызов функции с меньшим количеством аргументов, чем она принимает, называется *частичным применением*. Используя частичное применение мы можем избавиться от большого количества ненужного кода. Давайте посмотрим как могла бы выглядеть функция `allTheChildren` с некаррированной версией `map` из библиотеки lodash[^обратите внимание, аргументы передаются в другом порядке]:

```js
var allTheChildren = function(elements) {
  return _.map(elements, getChildren);
};
```

We typically don't define functions that work on arrays, because we can just call `map(getChildren)` inline. Same with `sort`, `filter`, and other higher order functions[^Higher order function: A function that takes or returns a function].

Обычно, мы не объявляем функции, которые принимают массив в качестве аргумента, потому что мы можем просто вызвать `map(getChildren)`. То же самое и с функциями `sort`, `filter` и другими функциями высшего порядка[^Функция высшего порядка — это функция, которая принимает в качестве аргумента или возвращает функцию.]

When we spoke about *pure functions*, we said they take 1 input to 1 output. Currying does exactly this: each single argument returns a new function expecting the remaining arguments. That, old sport, is 1 input to 1 output.

Когда мы говорили о *чистых функциях*, мы сказали, что для одного аргумента они возвращают одно значение. Это как раз то, чем занимается каррирование — каждый аргумент возвращает новую функцию, принимающую оставшиеся аргументы.

No matter if the output is another function - it qualifies as pure. We do allow more than one argument at a time, but this is seen as merely removing the extra `()`'s for convenience.

Нам совершенно безразлично, что возвращает функция: какое-либо значение или другую функцию — она всё равно будет считаться чистой. Мы позволяем функции принимать больше одного аргумента за раз, но, как мы могли убедиться, это то же самое, что и убрать лишние `()` для удобства.


## In summary
## Итог

Currying is handy and I very much enjoy working with curried functions on a daily basis. It is a tool for the belt that makes functional programming less verbose and tedious.

Каррирование — очень удобный инструмент и я очень люблю регулярно им пользоваться. С его помощью функциональное программирование становится менее многословным и нудным. 

We can make new, useful functions on the fly simply by passing in a few arguments and as a bonus, we've retained the mathematical function definition despite multiple arguments.

Мы можем создавать новые полезные функции не лету, просто передав пару аргументов, при этом, мы сохраняем математическую строгость в определении функции, несмотря на то, что мы пользуемся функциями многих переменных.

Let's acquire another essential tool called `compose`.

Давайте познакомимся с новым инструментом — `композицией`.

[Глава 5: Пишем код с использованием композиции](ch5-ru.md)

## Exercises
## Упражнения

A quick word before we start. We'll use a library called *ramda* which curries every function by default. Alternatively you may choose to use *lodash-fp* which does the same and is written/maintained by the creator of lodash. Both will work just fine and it is a matter of preference.

Небольшое замечание. Мы будем использовать библиотеку *ramda*, которая каррирует каждую функцию по умолчанию. В качестве альтернативы вы можете использовать *lodash-fp*, которая делает то же самое и написана (и поддерживается) создателем lodash. Обе библиотеки подходят для наших, выбирайте какая вам больше нравится.

[ramda](http://ramdajs.com)
[lodash-fp](https://github.com/lodash/lodash-fp)

There are [unit tests](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises) to run against your exercises as you code them, or you can just copy-paste into a javascript REPL for the early exercises if you wish.

Вы можете [тестировать](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises) упражнения в процессе написания или просто копипастить их в среду интерактивного выполнения и проверять — делайте как вам удобно.

Answers are provided with the code in the [repository for this book](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers)

Ответы на упражнения лежат в [репозитории](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers)

```js
var _ = require('ramda');


// Упражнение 1
//==============
// Refactor to remove all arguments by partially applying the function
// Проведите рефакторинг и избавьтесь от всех аргментов путём частичного применения функции

var words = function(str) {
  return _.split(' ', str);
};

// Упражнение 1a
//==============
// Use map to make a new words fn that works on an array of strings.
// Воспользуйтесь функцией map, чтобы создать новую функцию words, которая будет работать с массивами строк.

var sentences = undefined;


// Упражнение 2
//==============
// Refactor to remove all arguments by partially applying the functions
// Проведите рефакторинг и избавьтесь от всех аргментов путём частичного применения функции

var filterQs = function(xs) {
  return _.filter(function(x){ return match(/q/i, x);  }, xs);
};


// Упражнение 3
//==============
// Use the helper function _keepHighest to refactor max to not reference any
// arguments
// Воспользуйтесь функцией _keepHighest чтобы отрефакторить функцию max
// Функция max не должна принимать аргументов

// Не меняйте:
var _keepHighest = function(x,y){ return x >= y ? x : y; };

// Проведите рефакторинг:
var max = function(xs) {
  return _.reduce(function(acc, x){
    return _keepHighest(acc, x);
  }, -Infinity, xs);
};


// Бонус 1:
// ============
// wrap array's slice to be functional and curried.
// оберните метод slice так, чтобы он стал функциональным и каррируемым
// //[1,2,3].slice(0, 2)
var slice = undefined;


// Бонус 2:
// ============
// use slice to define a function "take" that takes n elements. Make it curried
// используйте метод slice, чтобы объявить функцию "take", которая принимает n аргументов. Сделайте её каррируемой
var take = undefined;
```
