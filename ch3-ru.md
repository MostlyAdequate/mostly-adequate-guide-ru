# Chapter 3: Pure Happiness with Pure Functions
# Глава 3: Чистое счастье с Чистыми функциями

## Oh to be pure again
## Как же хорошо снова быть чистым

One thing we need to get straight is the idea of a pure function.

Нам совершенно необходимо чётко понять концепцию чистой функции.

>A pure function is a function that, given the same input, will always return the same output and does not have any observable side effect.

>Чистая функция — это функция, которая при одинаковых аргументах всегда возращает одно и то же значения и не имеет видимых побочных эффектов.

Take `slice` and `splice`. They are two functions that do the exact same thing - in a vastly different way, mind you, but the same thing nonetheless. We say `slice` is *pure* because it returns the same output per input every time, guaranteed. `splice`, however, will chew up its array and spit it back out forever changed which is an observable effect.

Для примера рассмотрим `slice` и `splice`. Эти две функции работают абсолютно одинаково, хотя и делают это совершенно по-разному. Функция `slice` является *чистой*, потому что для одинаковых входных значений она всегда возвращает одни и те же значения на выходе. `splice`, с другой стороны, «съест» кусок массива и вернёт изменённую переменную, что и является побочным эффектом.

```js
var xs = [1,2,3,4,5];

// чистая
xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]


// не чистая
xs.splice(0,3);
//=> [1,2,3]

xs.splice(0,3);
//=> [4,5]

xs.splice(0,3);
//=> []
```

In functional programming, we dislike unwieldy functions like `splice` that *mutate* data. This will never do as we're striving for reliable functions that return the same result every time, not functions that leave a mess in their wake like `splice`.

В функциональном программировании мы не любим неуклюжие функции, такие как `splice`, которые изменяют данные. Такое поведение совершенно неприемлимо, потому как мы стремимся к написанию надёжных функций. Таких, которые для одинаковых входных данных возвращают одни и те же выходные. Нам не интересны функции, оставляющие за собой беспорядок в виде испорченных данных, как `splice`.

Let's look at another example.

Приведу ещё один пример.

```js
// не чистая
var minimum = 21;

var checkAge = function(age) {
  return age >= minimum;
};



// чистая
var checkAge = function(age) {
  var minimum = 21;
  return age >= minimum;
};
```

In the impure portion, `checkAge` depends on the mutable variable `minimum` to determine the result. In other words, it depends on system state which is disappointing because it increases the cognitive load by introducing an external environment.

В не чистом примере, `checkAge` использует изменяемую переменную `minimum`, другими словами, она зависит от состояния системы, что плохо, так как это усложняет понимание функции из-за связанности с внешней средой.

It might not seem like a lot in this example, but this reliance upon state is one of the largest contributors to system complexity[^http://www.curtclifton.net/storage/papers/MoseleyMarks06a.pdf]. This `checkAge` may return different results depending on factors external to input, which not only disqualifies it from being pure, but also puts our minds through the ringer each time we're reasoning about the software.

В данном примере это может показаться мелочью, но зависимость системы от своего состояния — это один из самых важных факторов её сложности[^http://www.curtclifton.net/storage/papers/MoseleyMarks06a.pdf]. Функция `checkAge` может возвращать разные значения, в зависимости от вншнего фактора, что не только исключает её из класса чистых, но ещё и затрудняет её понимание, каждый раз когда мы пытаемся проанализировать код.

Its pure form, on the other hand, is completely self sufficient. We can  also make `minimum` immutable, which preserves the purity as the state will never change. To do this, we must create an object to freeze.

Её чистый вариант, наоборот, полностью самодостаточен. Мы также можем сделать переменную `minimum` неизменяемой, что позволит сохранить чистоту, так как состояние никогда не изменится. Чтобы добиться этого нам потребуется использовать метод `freeze` класса `Object`.

```js
var immutableState = Object.freeze({
  minimum: 21
});
```

## Side effects may include...
## Среди побочных эффектов может наблюдаться...

Let's look more at these "side effects" to improve our intuition. So what is this undoubtedly nefarious *side effect* mentioned in the definition of *pure function*? We'll be referring to *effect* as anything that occurs in our computation besides the calculation of a result.

Давайте более подробно разберёмся с этими «побочными эффектами», дабы развить нашу интуицию. Что же за именно скрывается за этими несомненно гнусными *побочными эффектами*, упомянутыми в определении *чистой функции*? Мы будем считать *эффектом* всё, что вызывает какое-либо вычисление, кроме результата самой функции.

There's nothing intrinsically bad about effects and we'll be using them all over the place in the chapters to come. It's that *side* part that bears the negative connotation. Water alone is not an inherent larvae incubator, it's the *stagnant* part that yields the swarms, and I assure you, *side* effects are a similar breeding ground in your own programs.

В эффектах нет ничего «врождённо» плохого, напротив, в будущих главах мы будем использовать их постоянно. Негативную окраску добавляет именно прилагательное «побочный». В слове «вода» нет негативных оттенков, *застоявшаяся* вода — напротив, вызывает ассоциации бактерий и тухлятины. То же самое и с *побочными* эффектами в ваших программах.

>A *side effect* is a change of system state or *observable interaction* with the outside world that occurs during the calculation of a result.

>*Побочный эффект* — это изменение состояния системы или *заметное взаимодействие* с окружающим миром, которое происходит во время вычисления результата.

Side effects may include, but are not limited to
Побочные эффекты могут включать (но не ограничиваются):

  * changing the file system
  * inserting a record into a database
  * making an http call
  * mutations
  * printing to the screen / logging
  * obtaining user input
  * querying the DOM
  * accessing system state
  
  
  * изменениями в файловой системе
  * вставкой записи в базу данных
  * выполнение http-запроса
  * мутациями
  * выводом на экран / записью в лог
  * получением данных от пользователя
  * выполнением запроса к DOM
  * получением доступа к состоянию системы
  

And the list goes on and on. Any interaction with the world outside of a function is a side effect, which is a fact that may prompt you to suspect the practicality of programming without them. The philosophy of functional programming postulates that side effects are a primary cause of incorrect behavior.

Это далеко не полный список. Любое взаимодействие с миром вне функции уже является побочным эффектом, что может вызвать в вас сомнения в пользе отказа от побочных эффектов в программировании. Философия функционального программирования постулирует, что побочные эффекты являются первопричиной некорректного поведения.

It is not that we're forbidden to use them, rather we want to contain them and run them in a controlled way. We'll learn how to do this when we get to functors and monads in later chapters, but for now, let's try to keep these insidious functions separate from our pure ones.

Мы не собираемся полностью отказываться от них, лучше ... Мы научимся делать это используя функторы и монады в последующих главах, сейчас же, мы просто будем отделять коварные функции вызывающие побочные эффекты от чистых.

Side effects disqualify a function from being *pure* and it makes sense: pure functions, by definition, must always return the same output given the same input, which is not possible to guarantee when dealing with matters outside our local function.

Побочные эффекты исключают функцию из лиги *чистых* и это имеет свой смысл: чистые функции, по определению, обязаны всегда возвращать одно и то же значение, для одинаковых входных данных, что невозможно обеспечить, когда имеешь дело с зависимостями вне самой функции.

Let's take a closer look at why we insist on the same output per input. Pop your collars, we're going to look at some 8th grade math.

Давайте разберёмся, почему я так наставиваю на одном и том же результате при одинаковых входных данных. Садитесь за парту ровно, я покажу вам немного математики из 8го класса.

## Математика 8го класса

From mathisfun.com:

С сайта mathisfun.com

> A function is a special relationship between values:
> Each of its input values gives back exactly one output value.

> Фунцкия — это взаимоотношение между величинами:
> Каждое из входных значений возвращает одно и только одно выходное.

In other words, it's just a relation between two values: the input and the output. Though each input has exactly one output, that output doesn't necessarily have to be unique per input. Below shows a diagram of a perfectly valid function from `x` to `y`;

Другими словами, функция — это всего лишь взаимоотношение между двумя величинами: аргументом и значением. Хотя и каждому аргументу ставится в соответствие единственное значение функции, обратное не верно. Так, функция, вызванная с разными аргументами может возвращать одно и то же значение. На диаграмме ниже изображена вполне законная функция `x` → `y`.

<img src="images/function-sets.gif" />[^http://www.mathsisfun.com/sets/function.html]

To contrast, the following diagram shows a relation that is *not* a function since the input value `5` points to several outputs:

Для сравнения, следующая диаграмма показывает отношение, которое *не* является функцией. Так как для аргумента `5` есть несколько значений:

<img src="images/relation-not-function.gif" />[^http://www.mathsisfun.com/sets/function.html]

Functions can be described as a set of pairs with the position (input, output): `[(1,2), (3,6), (5,10)]`[^It appears this function doubles its input].

Функцию можно описать как набор упорядоченных пар (аргумент, значение): `[(1,2), (3,6), (5,10)]`[^Похоже, что эта функция удваивает аргумент].

Or perhaps a table:

Или в виде таблицы:
<table> <tr> <th>Input</th> <th>Output</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

<table> <tr> <th>Аргумент</th> <th>Значение</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

Or even as a graph with `x` as the input and `y` as the output:

Или в виде графика, где по оси `x` отложен аргумент, а по оси `y` — значение:

<img src="images/fn_graph.png" width="300" height="300" />


There's no need for implementation details if the input dictates the output. Since functions are simply mappings of input to output, one could simply jot down object literals and run them with `[]` instead of `()`.

Нам совершенно не важны детали реализации механизма функции, если мы знаем, что аргумент диктует значение. Так как функции являются всего лишь отображением аргумента на значение, то мы можем просто-напросто записывать литералы объекта с `[]`, вместо `()`.

```js
var toLowerCase = {"A":"a", "B": "b", "C": "c", "D": "d", "E": "e", "D": "d"};

toLowerCase["C"];
//=> "c"

var isPrime = {1:false, 2: true, 3: true, 4: false, 5: true, 6:false};

isPrime[3];
//=> true
```

Of course, you might want to calculate instead of hand writing things out, but this illustrates a different way to think about functions.[^You may be thinking "what about functions with multiple arguments?". Indeed, that presents a bit of an inconvenience when thinking in terms of mathematics. For now, we can bundle them up in an array or just think of the `arguments` object as the input. When we learn about *currying*, we'll see how we can directly model the mathematical definition of a function.]

Согласен, возможно вы захотите вычислять значение функции, а не просто вписывать вручную все возможные пар аргументов-значений.[^Кто-то из вас может заметить: «а как же функции многих переменных?» Действительно, с точки зрения может показаться несколько странным что мы не обговорили этот момент. На данный момент, будем считать несколько аргументов как один аргумент, являющийся массивом. Когда мы изучим *каррирование*, мы сможем легко пользоваться строгим математическим определением функции.]

Here comes the dramatic reveal: Pure functions *are* mathematical functions and they're what functional programming is all about. Programming with these little angels can provide huge benefits. Let's look at some reasons why we're willing to go to great lengths to preserve purity.

Здесь нас поджидает внезапное открытие: чистые функции *являются* математическими функциями, они явяляются фундаментом функционального программирования. Использование этих маленьких ангелочков таит в себе большое количество плюсов. Давайте посмотрим, зачем так стараться ради сохранения чистоты.

## The case for purity
## Место для чистоты

### Cacheable
### Кешируемость

For starters, pure functions can always be cached by input. This is typically done using a technique called memoization:

Начнём с того, что значения чистых функций можно кешировать по аргументу. Обычно это реализуется с помощью техники мемоизации:

```js
var squareNumber  = memoize(function(x){ return x*x; });

squareNumber(4);
//=> 16

squareNumber(4); // возвращает результат из кеша для аргумента 4
//=> 16

squareNumber(5);
//=> 25

squareNumber(5); // возвращает результат из кеша для аргумента 5
//=> 25
```

Here is a simplified implementation, though there are plenty of more robust versions available.

Ниже я написал упрощённую реализацию мемоизации, в интернете вы можете найти более надёжную версию.

```js
var memoize = function(f) {
  var cache = {};

  return function() {
    var arg_str = JSON.stringify(arguments);
    cache[arg_str] = cache[arg_str] || f.apply(f, arguments);
    return cache[arg_str];
  };
};
```

Something to note is that you can transform some impure functions into pure ones by delaying evaluation:

Некоторые функции можно превратить в чистые благодаря отложенному выполнению:

```js
var pureHttpCall = memoize(function(url, params){
  return function() { return $.getJSON(url, params); }
});
```

The interesting thing here is that we don't actually make the http call - we instead return a function that will do so when called. This function is pure because it will always return the same output given the same input: the function that will make the particular http call given the `url` and `params`.

Интересный момент, мы не делаем http-запрос, вместо него мы возвращаем функцию, которая сделает, когда будет вызвана. Эта функция является чистой, так как всегда для одинакового аргумента возвращает одно и то же значение — функцию, которая сделает http-запрос по заданным `url` и `params`.

Our `memoize` function works just fine, though it doesn't cache the results of the http call, rather it caches the generated function.

Функция `memoize` работает нормально, хотя она и не кеширует результат http-запроса, а кеширует сгенерированную функцию.

This is not very useful yet, but we'll soon learn some tricks that will make it so. The takeaway is that we can cache every function no matter how destructive they seem.

Пока что `memoize` не кажется очень полезной, однако скоро мы изучим некоторые хитрости, которые позволят кешировать любую функцию, вне зависимости от её чистоты.

### Portable / Self-Documenting
### Переносимость / Самодокументированность

Pure functions are completely self contained. Everything the function needs is handed to it on a silver platter. Ponder this for a moment... How might this be beneficial? For starters, a function's dependencies are explicit and therefore easier to see and understand - no funny business going on under the hood.

Чистые функцию полностью самодостаточны, всё что им нужно для работы они получают на блюдечке. Вдумайтесь, какие в этом могут быть плюсы? Начнём с зависимостей: они явные и, следовательно, их проще отследить — ничего не скрыто под капотом.

```js
// не чистая
var signUp = function(attrs) {
  var user = saveUser(attrs);
  welcomeUser(user);
};

// чистая
var signUp = function(Db, Email, attrs) {
  return function() {
    var user = saveUser(Db, attrs);
    welcomeUser(Email, user);
  };
};
```

The example here demonstrates that the pure function must be honest about its dependencies and, as such, tell us exactly what it's up to. Just from its signature, we know that it will use a `Db`, `Email`, and `attrs` which should be telling to say the least.

Этот пример показывает, что чистая функция должна быть полностью откровенна по отношению к своим зависимостям и, тем самым, иллюстрирует нам своё предназначение. Просто посмотрев на сигнатуру функции мы по меньшей мере можем понять, что она использует `Db`, `Email` и `attrs`.

We'll learn how to make functions like this pure without merely deferring evaluation, but the point should be clear that the pure form is much more informative than its sneaky impure counterpart which is up to God knows what.

Мы научимся писать такие чистые функции, даже без использования отложенного выполнения. Надеюсь, вам уже стало ясно, что чистая функция гораздо более информативна, чем подозрительная нечистая, намерения которой не ясны.

Something else to notice is that we're forced to "inject" dependencies, or pass them in as arguments, which makes our app much more flexible because we've parameterized our database or mail client or what have you[^Don't worry, we'll see a way to make this less tedious than it sounds]. Should we choose to use a different Db we need only to call our function with it. Should we find ourselves writing a new application in which we'd like to reuse this reliable function, we simply give this function whatever `Db` and `Email` we have at the time.

Хочу также отметить, что мы вынуждены «внедрять» зависимости, или передавать их в качестве аргументов, что делает наше приложение куда более гибким, за счёт параметризации вашего приложения-клиента к базе данных или почтового клиента (или что у вас там за приложение?)[^Не волнуйтесь, мы научимся делать это менее скучно, чем это звучит]. Если вы хотите поменять базу данных, то вам всего-навсего потребуется вызвать функцию, передав новую базу в качестве аргумента. Когда вы будете писать новое приложение и захотите переиспользовать нашу надёжную чистую функцию, вы можете просто передать ей те `Db` и `Email`, которые актуальны для вас.

In a JavaScript setting, portability could mean serializing and sending functions over a socket. It could mean running all our app code in web workers. Portability is a powerful trait.

В среде JavaScript портативность может значить сериализацию и отправку функции через сокет или же запуск всего нашего приложения с помощью Web Worker'ов. Портативность — мощная штука.

Contrary to "typical" methods and procedures in imperative programming rooted deep in their environment via state, dependencies, and available effects, pure functions can be run anywhere our hearts desire.

В отличие от «традиционных» методов и процедур в императивном программировании, которые жёстко связаны с состоянием системы, зависимостями и доступными эффектами, чистые функции могут использовать везде, где нашей душе угодно.

When was the last time you copied a method into a new app? One of my favorite quotes comes from Erlang creator, Joe Armstrong: "The problem with object-oriented languages is they’ve got all this implicit environment that they carry around with them. You wanted a banana but what you got was a gorilla holding the banana... and the entire jungle".

Вспомните последний раз, когда вы копировали метод из одного приложения в другое. Давно это было? Одна из моих самых любимых цитат была произнесена создателем языка Erlang, Джое Армстронгом: «Проблема объектно-ориентированных языков в неявной среде, которая их окружает. Вы хотели получить банан, а получили гориллу с бананом... И все джунгли».

### Testable
### Тестирумость

Next, we come to realize pure functions make testing much easier. We don't have to mock a "real" payment gateway or setup and assert the state of the world after each test. We simply give the function input and assert output.

Дальше — больше, тестировать чистые функции проще. Вам больше не потребуется симулировать «реальный» платёжный шлюз или настраивать и строить предположения по поводу состояния всего мира во время каждого теста.

In fact, we find the functional community pioneering new test tools that can blast our functions with generated input and assert that properties hold on the output. It's beyond the scope of this book, but I strongly encourage you to search for and try *Quickcheck* - a testing tool that is tailored for a purely functional environment.

# FIXME
Функциональное сообщество создаёт новые инструменты для тестирования, которые могут очень серьёзно ускорить работу тестов с помощью генерации аргументов и предположений по значению функции. Это вне рамок этой книги, но я крайне рекомендую поискать в интернете и попробовать *Quickcheck* — инструмент для тестирования, написанный как раз для функциональной среды.

### Reasonable
### Разумность

Many believe the biggest win when working with pure functions is *referential transparency*. A spot of code is referentially transparent when it can be substituted for its evaluated value without changing the behavior of the program.

Многие считают одним из главных преимуществ чистых функций *прозрачность ссылок*. Участок когда можно считать «ссылочно-прозрачным», когда его можно заменить на вычисленный им результат и при этом поведение программы не изменится.

Since pure functions always return the same output given the same input, we can rely on them to always return the same results and thus preserve referential transparency. Let's see an example.

Поскольку чистые функции всегда возвращают один и тот же результат, для одинаковых аргументов — они сохраняют прозрачность ссылок. Давайте обратимся к примеру.

```js

var decrementHP = function(player) {
  return player.set("hp", player.hp-1);
};

var isSameTeam = function(player1, player2) {
  return player1.team === player2.team;
};

var punch = function(player, target) {
  if(isSameTeam(player, target)) {
    return target;
  } else {
    return decrementHP(target);
  }
};

var jobe = Immutable.Map({name:"Jobe", hp:20, team: "red"});
var michael = Immutable.Map({name:"Michael", hp:20, team: "green"});

punch(jobe, michael);
//=> Immutable.Map({name:"Michael", hp:19, team: "green"})
```

`decrementHP`, `isSameTeam` and `punch` are all pure and therefore referentially transparent. We can use a technique called *equational reasoning* wherein one substitutes "equals for equals" to reason about code. It's a bit like manually evaluating the code without taking into account the quirks of programmatic evaluation. Using referential transparency, let's play with this code a bit.

Функции `decrementHP`, `isSameTeam` и `punch` являются чистыми и, следовательно, ссылочно-прозрачными. Мы можем воспользоваться техникой *рассуждения о равном*, в которой мы заменяем равное равным, чтобы анализировать код. Это примерно то же самое, что и в уме просчитывать код, не учитывая тонкостей его программной реализации. Давай поиграем с кодом, используя прозрачность ссылкок.

First we'll inline the function `isSameTeam`.

Начнём с того, что встроим функцию `isSameTeam` в `punch`.

```js
var punch = function(player, target) {
  if(player.team === target.team) {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

Since our data is immutable, we can simply replace the teams with their actual value

Так как наши данные не меняются, то мы просто заменим переменные `team` на их значения.

```js
var punch = function(player, target) {
  if("red" === "green") {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

We see that it is false in this case so we can remove the entire if branch

Заметим, что условие всегда ложно и избавимся от него:

```js
var punch = function(player, target) {
  return decrementHP(target);
};

```

And if we inline `decrementHP`, we see that, in this case, punch becomes a call to decrement the `hp` by 1.

И, если мы встроим `decrementHP`, мы поймём, что вызов функции `punch` становится вызовом функции по уменьшению ХП на 1 единицу.

```js
var punch = function(player, target) {
  return target.set("hp", target.hp-1);
};
```

This ability to reason about code is terrific for refactoring and understanding code in general. In fact, we used this technique to refactor our flock of seagulls program. We used equational reasoning to the properties of addition and multiplication. Indeed, we'll be using these techniques throughout the book.

Способность анализировать код таким образом чрезвычайно удобна для рефакторинга и понимания того, что код на самом деле делает. Мы уже пользовались этой техникой ранее, для рефакторинга нашей программы про стаи чаек (свойства сложения и умножения)

### Parallel Code
### Параллелизм

Finally, and here's the coup de grâce, we can run any pure function in parallel since it does not need access to shared memory and it cannot, by definition, have a race condition due to some side effect.

В конце концов, сейчас будет вишнка на торте, мы можем запустить любую чистую функцию параллельно, так как ей не нужен доступ в общую память и она по определению привести к состоянию гонки из-за какого-либо побочного эффекта.

This is very much possible in a server side js environment with threads as well as in the browser with web workers though current culture seems to avoid it due to complexity when dealing with impure functions.

Это вполне применимо к потоковому JS на сервере, как и к браузерному с использованием web workers, хотя их особенно и не используют из-за сложностей, возникающих при работе с нечистыми функциями.

## In Summary
## В итоге

We've seen what pure functions are and why we, as functional programmers, believe they are the cat's evening wear. From this point on, we'll strive to write all our functions in a pure way. We'll require some extra tools to help us do so, but in the meantime, we'll try to separate the impure functions from the rest of the pure code.

Мы познакомились с чистыми функциями и поняли, почему мы, как функциональные программисты, считаем их манной небесной. С этого момента мы будем стараться писать все функции чистыми. Нам потребуется ещё несколько инструментов для этого, а пока они не изучены, мы будем отделять чистые функции от остального кода.

Writing programs with pure functions is a tad laborious without some extra tools in our belt. We have to juggle data by passing arguments all over the place, we're forbidden to use state, not to mention effects. How does one go about writing these masochistic programs? Let's acquire a new tool called curry.

Писать программы с чистыми функциями нелегко, без использования некоторых инструментов. Нам придётся жонглировать данными, постоянно передавая аргументы, нам запрещено использоваться состояние, не говоря уже об эффектах. Как вы смотрите на такой мазохистский подход? Давайте изучим новый инструмент — каррирование.

[Глава 4: Каррирование](ch4-ru.md)
