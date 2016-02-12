# Хиндли-Милнер и Я

## Какого ты типа?

Если вы новичок в функциональном мире, то пройдет совсем немного времени прежде, чем вы погрузитесь в сигнатуры типов достаточно глубоко. Типы - это метаязык, который позволяет людям с самым различным бэкграундом общаться емко и эффективно. Большинство из них написанны в системе Хиндли-Милнера, которую мы вместе рассмотрим в этой главе.

If you're new to the functional world, it won't be long before you find yourself knee deep in type signatures. Types are the meta language that enables people from all different backgrounds to communicate succinctly and effectively. For the most part, they're written with a system called "Hindley-Milner", which we'll be examining together in this chapter.

Когда работаешь с чистыми функциями, сигнатуры типов имеют выразительную мощь, с которой не сравнится никакой естественный язык. Эти сигнатуры шепчут вам о сокровенных секретах функций. В одну простую, компактную строку они вмещают описание поведения и цели функции. Из них мы можем вывести "бесплатные теоремы". Типы могут быть описаны таким образом, что не будет необходимости в явных анотациях. Они могут быть настроены довольно точно или описывать все в общем и абстрактном свете. Они пригодны не только для проверок на этапе компиляции, но также могут служить отличной документацией. Таким образом, сигнатуры типов играют важную роль в функциональном программировании - намного большую, чем вы могли ожидать вначале.

When working with pure functions, type signatures have an expressive power to which the English language cannot hold a candle. These signatures whisper in your ear the intimate secrets of a function. In a single, compact line, they expose behaviour and intention. We can derive "free theorems" from them. Types can be inferred so there's no need for explicit type annotations. They can be tuned to fine point precision or left general and abstract. They are not only useful for compile time checks, but also turn out to be the best possible documentation available. Type signatures thus play an important part in functional programming - much more than you might first expect.

JavaScript - динамический язык, но это не означает, что мы все избегаем типов данных. Мы все еще работаем со строками, числами, булевским типом и так далее. Просто нет каких-либо средств на уровне языка, так что мы должны держать это все в голове. Не беспокойтесь, если мы используем сигнатуры для документирования, мы можем заставить комментарии послужить нашим целям.

JavaScript is a dynamic language, but that does not mean we avoid types all together. We're still working with strings, numbers, booleans, and so on. It's just that there isn't any language level integration so we hold this information in our heads. Not to worry, since we're using signatures for documentation, we can use comments to serve our purpose.

Доступны утилиты для проверки типов, такие как [Flow](http://flowtype.org/) или даже типизированные диалекты - [TypeScript](http://www.typescriptlang.org/). Целью этой книги является снабжение вас инструментами для написания функционального кода, так что мы будем придерживаться стандартной системы типов используемой в функциональных языках.

There are type checking tools available for JavaScript such as [Flow](http://flowtype.org/) or the typed dialect, [TypeScript](http://www.typescriptlang.org/). The aim of this book is to equip one with the tools to write functional code so we'll stick with the standard type system used across FP languages.

## Байки из Скрипта

## Tales from the cryptic

С пыльных страниц математических томов, сквозь глубокое моге белых листов, посреди обыкновенных записей в блогах по субботним утрам, приближаясь все ближе к исходному коду, мы находим сигнатуры типов Хиндли-Милнера. Система довольно проста, но гарантирует быстрое объяснение и некоторые практики, которые полностью покрывают небольшой язык.

From the dusty pages of math books, across the vast sea of white papers, amongst casual saturday morning blog posts, down into the source code itself, we find Hindley-Milner type signatures. The system is quite simple, but warrants a quick explanation and some practice to fully absorb the little language.

```js
//  capitalize :: String -> String
var capitalize = function(s){
  return toUpperCase(head(s)) + toLowerCase(tail(s));
}

capitalize("smurf");
//=> "Smurf"
```

Здесь `capitalize` принимет `String` и возвращает тоже `String`. Не думайте о реализации, нас интересует сигнатура типов. 

Here, `capitalize` takes a `String` and returns a `String`. Never mind the implementation, it's the type signature we're interested in.

В системе Хиндли-Милнера функции записываются как `a -> b`, где `a` и `b` - это переменные того или иного типа. Так что сигнатура функции `capitalize` может быть прочтена как "функция из `String` в `String`". Другими словами, она принимает на вход `String` и возвращает `String` в качестве результата.

In HM, functions are written as `a -> b` where `a` and `b` are variables for any type. So the signatures for `capitalize` can be read as "a function from `String` to `String`". In other words, it takes a `String` as its input and returns a `String` as its output.

Давайте посмотрим еще на несколько сигнатур:

Let's look some more function signatures:

```js
//  strLength :: String -> Number
var strLength = function(s){
  return s.length;
}

//  join :: String -> [String] -> String
var join = curry(function(what, xs){
  return xs.join(what);
});

//  match :: Regex -> String -> [String]
var match = curry(function(reg, s){
  return s.match(reg);
});

//  replace :: Regex -> String -> String -> String
var replace = curry(function(reg, sub, s){
  return s.replace(reg, sub);
});
```

В `strLength` та же идея, что и прежде. Мы берем тип `String` и возрвщаем тип `Number`.

`strLength` is the same idea as before: we take a `String` and return you a `Number`.

Другие примеры могут озадачить на первый взгляд. Без полноценного понимания всех деталей, вы всегда можете видеть последний тип, значения которого и возвращаются. Так что функция `match` может быть интерпретирована так: она берет `Regex` и `String` и возвращает вам `[String]`. Но здесь происходит интересная вещь, и я бы хотел объяснить вам это, если можно.

The others might perplex you at first glance. Without fully understanding the details, you could always just view the last type as the return value. So for `match` you can interpret as: It takes a `Regex` and a `String` and returns you `[String]`. But an interesting thing is going on here that I'd like to take a moment to explain if I may.

Для `match` мы можем сгруппировать сигнатуры следуюшим образом:

For `match` we are free to group the signature like so:

```js
//  match :: Regex -> (String -> [String])
var match = curry(function(reg, s){
  return s.match(reg);
});
```

Ах да, группировка последней части в круглые скобки предоставляет нам больше сведений. Теперь мы видим ее как функцию, которая принимает `Regex` и возвращает функцию из `String` в `[String]`. Вот что происходит на самом деле из-за каррирования: передвая функции `Regex` мы получаем другую функцию, которая ожидает в качестве аргумента `String`. Конечно, нам не обязательно думать в таком ключе каждый раз, но полезно понимать откуда и как возвращается последний тип.

Ah yes, grouping the last part in parenthesis reveals more information. Now it is seen as a function that takes a `Regex` and returns us a function from `String` to `[String]`. Because of currying, this is indeed the case: give it a `Regex` and we get a function back waiting for its `String` argument. Of course, we don't have to think of it this way, but it is good to understand why the last type is the one returned.

```js
//  match :: Regex -> (String -> [String])

//  onHoliday :: String -> [String]
var onHoliday = match(/holiday/ig);
```

Каждый аргумент вытасквиается с левого края сигнатуры. `onHoliday` - это `match` у которого уже есть `Regex`.

Each argument pops one type off the front of the signature. `onHoliday` is `match` that already has a `Regex`.

```js
//  replace :: Regex -> (String -> (String -> String))
var replace = curry(function(reg, sub, s){
  return s.replace(reg, sub);
});
```

Как вы можете видеть, все эти круглые скобки в `replace` являются слегка трудными для чтения и кажутся излишними, так что мы просто опустим их. Мы можем взять все аргументы сразу, если если решим, что проще думать о них так: `replace` принимает  `Regex`, `String`, другой `String` и в конце возвращает `String`.

As you can see with the full parenthesis on `replace`, the extra notation can get a little noisy and redundant so we simply omit them. We can give all the arguments at once if we choose so it's easier to just think of it as: `replace` takes a `Regex`, a `String`, another `String` and returns you a `String`.

Еще пару штук, напоследок:

A few last things here:


```js
//  id :: a -> a
var id = function(x){ return x; }

//  map :: (a -> b) -> [a] -> [b]
var map = curry(function(f, xs){
  return xs.map(f);
});
```

Функция `id` принимет любой тип `a` и возвращает что-то того же типа `a`. Мы можем использовать переменные любых типов в коде вроде этого. Имена переменных, такие как `a` и `b` - это соглашение, но они произвольны и могут быть заменены на переменные с любым именем, каким вы захотите. Если это одни и те же переменные, то у них должен быть один и тот же тип. Это важное правило, так что давайте повторим: `a` -> `b` - может быть любого типа `a` и любого типа `b`, но `a -> a` означает, что это это один и тот же тип. Например, `id` может быть `String -> String` или `Number -> Number`, но не `String -> Bool`.

The `id` function takes any old type `a` and returns something of the same type `a`. We're able to use variables in types just like in code. Variable names like `a` and `b` are convention, but they are arbitrary and can be replaced with whatever name you'd like. If they are the same variable, they have to be the same type. That's an important rule so let's reiterate: `a -> b` can be any type `a` to any type `b`, but `a -> a` means it has to be the same type. For example, `id` may be `String -> String` or `Number -> Number`, but not `String -> Bool`.

Таким же образом `map` использует типизированные переменные, но но в этот раз мы представляем `b` который может быть (а может и не быть) того же типа, что и `a`. Мы можем прочитать это так: `map` принимает функцию из любого типа `a` в такой же тип или в другой тип `b`, затем принимает массив переменных типа `a` и в результате передает массив типов `b`.

`map` similarly uses type variables, but this time we introduce `b` which may or may not be the same type as `a`. We can read it as: `map` takes a function from any type `a` to the same or different type `b`, then takes an array of `a`'s and results in an array of `b`'s.

Будем надеятся, что вас захватила выразительная красота этой сигнатуры. Она буквально говорит что делает функция, почти слово в слово. Дана функция из `a` в `b`, массив типа `a` и эта функция даст нам массив типа `b`. Единственно здравая вещь, которую должна делать эта функция - это применить кровавую функцию к каждому `a`. Все остальное - гнусная ложь.

Hopefully, you've been overcome by the expressive beauty in this type signature. It literally tells us what the function does almost word for word. It's given a function from `a` to `b`, an array of `a`, and it delivers us an array of `b`. The only sensible thing for it to do is call the bloody function on each `a`. Anything else would be a bold face lie.

Возможность рассуждать о типах и их смысле - это навык, с которым вы далеко пойдете в функциональном мире. Не только статьи, блоги, документации и прочее станет более доступным, кроме того, сами сигнатуры будут полезными анотациями о функциональности. Требуется практика, чтобы свободно читать их, но если вы уделите этому время, то куча информации станет доступна вам без  чтения подробных мануалов.

Being able to reason about types and their implications is a skill that will take you far in the functional world. Not only will papers, blogs, docs, etc, become more digestible, but the signature itself will practically lecture you on its functionality. It takes practice to become a fluent reader, but if you stick with it, heaps of information will become available to you sans RTFMing.

Вот еще немного, просто чтобы посмотреть, сможете ли вы расшифровать их сами.

Here's a few more just to see if you can decipher them on your own.

```js
//  head :: [a] -> a
var head = function(xs){ return xs[0]; }

//  filter :: (a -> Bool) -> [a] -> [a]
var filter = curry(function(f, xs){
  return xs.filter(f);
});

//  reduce :: (b -> a -> b) -> b -> [a] -> b
var reduce = curry(function(f, x, xs){
  return xs.reduce(f, x);
});
```

Возможно, `reduce` - самый выразительный из них. Он довольно запутан, поэтому не комплексуйте, если вам придётся попотеть.

`reduce` is perhaps, the most expressive of all. It's a tricky one, however, so don't feel inadequate should you struggle with it.

## Сужение возможностей

## Narrowing the possibility

После того, как представлен тип переменной, возникает любопытное свойство называемое *параметричностью*[^http://en.wikipedia.org/wiki/Parametricity]. Это свойство говорит, что функция будет *вести себя со всеми типами одинаково*. Давайте рассмотрим:

Once a type variable is introduced, there emerges a curious property called *parametricity*[^http://en.wikipedia.org/wiki/Parametricity]. This property states that a function will *act on all types in a uniform manner*. Let's investigate:

```js
// head :: [a] -> a
```

Вглядываясь в `head` мы видим как он превращает `[a]` в `a`. Кроме конкретнного типа `array`, не доступно больше никакой информации, следовательно, функциональность ограничена работой только над массивом. Что вообще можно сделать с переменной `a`, если ничего про нее не известно? `a` не говорит о том, что он какого-то *специфичного* типа, напротив, `a` может быть *любого* типа, что оставляет нам функцию, которая должна работать одинаково для *каждого* возможного типа. Это и есть *параметричность*. Задумываясь о реализации, единственно осмысленные допущения - это то, что функция вытаскивает первый, последний, или любой другой элемент из массива. Имя `head` должно подсказать нам, какой именно.

Looking at `head`, we see that it takes `[a]` to `a`. Besides the concrete type `array`, it has no other information available and, therefore, its functionality is limited to working on the array alone. What could it possibly do with the variable `a` if it knows nothing about it? In other words, `a` says it cannot be a *specific* type, which means it can be *any* type, which leaves us with a function that must work uniformly for *every* conceivable type. This is what *parametricity* is all about. Guessing at the implementation, the only reasonable assumptions are that it takes the first, last, or a random element from that array. The name `head` should tip us off.

Вот еще одна: 

Here's another one:

```js
// reverse :: [a] -> [a]
```

Исходя только из сигнатуры, чем может быть `reverse`? Вновь она не может сделать ничего особенного с `a`. Она не может привести `a` к какому-то другому типу. Может быть сортирует? Что же, нет, недостаточно информации, чтобы сортировать все  возможные типы. Возможно перетасовка? Да, я полагаю она может это сделать, но это должно происходить одинаковым и предсказуемым образом. Другая возможность, о которой я могу подумать, - это удаление или дублирование элемента. В любом случае, суть в том, что возможное поведение функции значительно сужается благодаря разнородности типа.

From the type signature alone, what could `reverse` possibly be up to? Again, it cannot do anything specific to `a`. It cannot change `a` to a different type or we'd introduce a `b`. Can it sort? Well, no, it wouldn't have enough information to sort every possible type. Can it re-arrange?  Yes, I suppose it can do that, but it has to do so in exactly the same predictable way. Another possibility is that it may decide to remove or duplicate an element. In any case, the point is, the possible behaviour is massively narrowed by its polymorphic type.

Это сужение возможностей позволяет нам использовать сигнатуры типов в поисковых движках вроде [Hoogle](https://www.haskell.org/hoogle), чтобы найти функцию. Информация упакованная в сигнатуры - действительно довольно мощное средство.

This narrowing of possibility allows us to use type signature search engines like [Hoogle](https://www.haskell.org/hoogle) to find a function we're after. The information packed tightly into a signature is quite powerful indeed.

## Свобода в правилах

## Free as in theorem

Кроме вывода реализации возможностей, этот тип рассужденией дает нам *правила-теоремы, без дополнительных усилий*. Вот несколько случайных примеров прямиком из [работы Уодлера по этой теме](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf).

Besides deducing implementation possibilities, this sort of reasoning gains us *free theorems*. What follows are a few random example theorems lifted directly from [Wadler's paper on the subject](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf).

```js
// head :: [a] -> a
compose(f, head) == compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) == compose(filter(p), map(f));
```

Вам не нужен какой-либо код, чтобы получить эти правила, они следуют прямо из типов. Первое говорит нам, что если мы имеет `head` для массива, затем применяем некую функцию `f` к ней, то это эквивалентно (и между прочим, намного быстрее) тому, как если бы мы сначала исполнили `map(f)` на кажом элементе массива, а затем бы применили `head` к результату.

You don't need any code to get these theorems, they follow directly from the types. The first one says that if we get the `head` of our array, then run some function `f` on it, that is equivalent to, and incidentally, much faster than, if we first `map(f)` over every element then take the `head` of the result.

Возможно вы думаете, ладно, это всего-лишь здравый смысл. Но в последний раз, когда я проверял, у компьютеров не было здравого смысла. Действительно, они должны иметь формальный способ автоматизировать подобные оптимизации программы. У математики есть средство, чтобы формализовать ход своих мыслей, что помогает продираться свозь жесткую местность компьютерной логики.

You might think, well that's just common sense. But last I checked, computers don't have common sense. Indeed, they must have a formal way to automate these kind of code optimizations. Maths has a way of formalizing the intuitive, which is helpful amidst the rigid terrain of computer logic.

Правило для `filter` выглядит знакомым. Оно говорит, что если мы создаем композицию `f` и `p` чтобы вычислить то, что должно быть отфильтрованно, то мы на самом деле применяем `f` с помощью `map` (помните, filter не изменяет элементы, -  его сигнатура принуждает к тому, что `a` не будет тронуто), и это всегда будет эквивалентно тому, что мы преобразуем `f`, а затем отфильтруем результат с помощью предиката `p`.

The `filter` theorem is similar. It says that if we compose `f` and `p` to check which should be filtered, then actually apply the `f` via `map` (remember filter, will not transform the elements - its signature enforces that `a` will not be touched), it will always be equivalent to mapping our `f` then filtering the result with the `p` predicate.

Здесь просто два примера. Но вы можете применить этот тип рассуждений к любой полиморфной сигнатуре и это будет работать. В JavaScript есть несколько доступных инструментов, чтобы объявлять переиспользуемые правила. Можно также сделать это с помощью функции `compose`. Возможности безграничны.

These are just two examples, but you can apply this reasoning to any polymorphic type signature and it will always hold. In JavaScript, there are some tools available to declare rewrite rules. One might also do this via the `compose` function itself. The fruit is low hanging and the possibilities are endless.

## В итоге 

## In Summary

Сигнатуры типов Хиндли-Милнера вездесущи в функциональном мире. Хотя они просты для чтения и записи, нужно время чтобы отточить технику понимания программ с помощью одних только сигнатур. С этого момента, мы будем добавлять сигнатуры типов к каждой строке кода. 

[Глава 8, Контейнер]

Hindley-Milner type signatures are ubiquitous in the functional world. Though they are simple to read and write, it takes time to master the technique of understanding programs through signatures alone. We will add type signatures to each line of code from here on out.

[Chapter 8: Tupperware](ch8.md)
