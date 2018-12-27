# Глава 4: Каррирование

## Без тебя мне жизнь не мила

Однажды мой отец рассказал мне, что есть вещи, без которых легко обходишься только до тех пор, пока не приобретаешь. Микроволновки смартфоны — как раз из таких, хотя люди постарше помнят полноценную жизнь даже без интернета. Для меня каррирование — относится к таким вещам.

Идея проста: вы можете вызвать функцию с меньшим количеством аргументов, чем она ожидает, в ответ вы получите функцию, которая принимает оставшиеся аргументы. *(Всё неправда - прим.пер.)*

Таким образом, вы можете передавать функции как все аргументы сразу, так и отдельные аргументы в разные моменты времени.

```js
const add = x => y => x + y;
const increment = add(1);
const addTen = add(10);

increment(2); // 3
addTen(2); // 12
```

В этом примере мы определили функцию `add`, которая принимает один аргумент и возвращает новую функцию. Новая функция принимает второй аргумент (y), а также через замыкание для неё будет известен первый аргумент (x). Чтобы определять подобные функции было проще, мы воспользуемся вспомогательной функцией `curry`. *(При необходимости, читайте [подробнее о работе замыканий в JavaScript](https://developer.mozilla.org/ru/docs/Web/JavaScript/Closures) - прим.пер.)*.

Давайте подготовим для себя несколько каррированных функций. С этого момента мы будем подразумевать, что `curry` - это та функция, которая определена для нас в [Приложении A - Вспомогательные функции](appendix_a.md).

```js
const match = curry((what, s) => s.match(what));
const replace = curry((what, replacement, s) => s.replace(what, replacement));
const filter = curry((f, xs) => xs.filter(f));
const map = curry((f, xs) => xs.map(f));
```

При определении этих функций я придерживался простого, но важного принципа: Набор данных, с которым мы работаем (к примеру, строка или массив) я специально расположил последним аргументом. Вскоре станет ясно, почему я так сделал.

(Синтаксис `/r/g` - это регулярное выражение, которое _соответствует каждой букве 'r'_. Прочитайте [подробнее о регулярных выражениях](https://developer.mozilla.org/ru/docs/Web/JavaScript/Guide/Regular_Expressions), если интересно.)

```js
match(/r/g, 'hello world'); // [ 'r' ]

const hasLetterR = match(/r/g); // x => x.match(/r/g)
hasLetterR('hello world'); // [ 'r' ]
hasLetterR('just j and s and t etc'); // null

filter(hasLetterR, ['rock and roll', 'smooth jazz']); // ['rock and roll']

const removeStringsWithoutRs = filter(hasLetterR); // xs => xs.filter(x => x.match(/r/g))
removeStringsWithoutRs(['rock and roll', 'smooth jazz', 'drum circle']); // ['rock and roll', 'drum circle']

const noVowels = replace(/[aeiou]/ig); // (r,x) => x.replace(/[aeiou]/ig, r)
const censored = noVowels('*'); // x => x.replace(/[aeiou]/ig, '*')
censored('Chocolate Rain'); // 'Ch*c*l*t* R**n'
```

Здесь я продемонстрировал способность «предзагрузки» функции с аргументом или несколькими для того, чтобы получить новую функцию, которая запомнит эти аргументы.

Советую вам выполнить `npm install lodash` в консоли, скопировать код выше и поиграться с ним. То же самое можно сделать и в браузере (вам в любом случае потребуется библиотека lodash или ramda).

## Больше, чем просто приправа

Каррирование полезно во многих случаях, с его помощью мы можем создавать новые функции на лету, просто передавая меньше аргументов в уже существующие, как мы могли убедиться в случаях с `hasSpaces`, `findSpaces`, и `censored`.

Мы также можем преобразовать любую функцию, которая принимает один аргумент, в функцию, принимающую массив. Для этого нам нужно просто обернуть её в `map`:

```js
const getChildren = x => x.childNodes;
const allTheChildren = map(getChildren);
```

Вызов функции с меньшим количеством аргументов, чем она принимает, называется *частичным применением*. Используя частичное применение мы можем избавиться от большого количества ненужного кода. Давайте посмотрим как могла бы выглядеть функция `allTheChildren` с некаррированной версией `map` из библиотеки lodash[^обратите внимание, аргументы передаются в другом порядке]:

```js
const allTheChildren = elements => map(elements, getChildren);
```

Обычно, мы не объявляем функции, которые принимают массив в качестве аргумента, потому что мы можем просто написать `map(getChildren)`. То же самое и с функциями `sort`, `filter` и другими функциями высшего порядка[^Функция высшего порядка — это функция, которая принимает в качестве аргумента или возвращает функцию.]

Когда мы говорили о *чистых функциях*, мы договорились о том, что для одного аргумента они возвращают одно значение. Это как раз то, чем занимается каррирование — каждый аргумент возвращает новую функцию, принимающую оставшиеся аргументы.

Нам совершенно безразлично, что возвращает функция: значение или другую функцию — она всё равно будет считаться чистой. Мы позволяем функции принимать больше одного аргумента за раз, но, как мы могли убедиться, это то же самое, что и убрать лишние `()` для удобства.

## Итог

Каррирование — это удобный инструмент и я очень люблю регулярно им пользоваться. С его помощью функциональное программирование становится куда менее многословным и нудным. 

Мы можем создавать новые полезные функции на лету, просто передав пару аргументов, при этом, мы сохраняем математическую строгость в определении функции, даже несмотря на то, что мы пользуемся функциями многих переменных.

Давайте познакомимся с новым инструментом — `композицией`.

[Глава 05: Пишем код с использованием композиции](ch5-ru.md)

## Упражнения

    Небольшое замечание. Мы будем использовать библиотеку *ramda*, которая каррирует каждую функцию по умолчанию. В качестве альтернативы вы можете использовать *lodash-fp*, которая делает то же самое. Она написана (и поддерживается) создателем lodash. Обе библиотеки подходят для наших нужд, выбирайте какая вам больше нравится.

    [ramda](http://ramdajs.com)
    [lodash-fp](https://github.com/lodash/lodash-fp)

    Вы можете [тестировать](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises) упражнения в процессе написания или просто копипастить их в среду интерактивного выполнения и проверять — делайте как вам удобно.

    Ответы на упражнения лежат в [репозитории](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers)

#### Note about Exercises

Throughout the book, you might encounter an 'Exercises' section like this one. Exercises can be
done directly in-browser provided you're reading from [gitbook](https://mostly-adequate.gitbooks.io/mostly-adequate-guide) (recommended).

Note that, for all exercises of the book, you always have a handful of helper functions
available in the global scope. Hence, anything that is defined in [Appendix A](./appendix_a.md),
[Appendix B](./appendix_b.md) and [Appendix C](./appendix_c.md) is available for you! And, as
if it wasn't enough, some exercises will also define functions specific to the problem
they present; as a matter of fact, consider them available as well.

> Hint: you can submit your solution by doing `Ctrl + Enter` in the embedded editor!

#### Running Exercises on Your Machine (optional)

Should you prefer to do exercises directly in files using your own editor:

- clone the repository (`git clone git@github.com:MostlyAdequate/mostly-adequate-guide.git`)
- go in the *exercises* section (`cd mostly-adequate-guide/exercises`)
- install the necessary plumbing using [npm](https://docs.npmjs.com/getting-started/installing-node) (`npm install`)
- complete answers by modifying the files named *exercises\_\** in the corresponding chapter's folder 
- run the correction with npm (e.g. `npm run ch04`)

Unit tests will run against your answers and provide hints in case of mistake. By the by, the
answers to the exercises are available in files named *answers\_\**.

### Давайте попрактикуемся!

#### Упражнение A

Проведите рефакторинг и избавьтесь от всех аргументов путём частичного применения функции. 
  
```js  
const words = str => split(' ', str);  
```  

#### Упражнение B

Проведите рефакторинг и избавьтесь от всех аргументов путём частичного применения функции. 

```js  
const filterQs = xs => filter(x => match(/q/i, x), xs);
```  
  
#### Упражнение C

Воспользуйтесь функцией `keepHighest`:

```js  
const keepHighest = (x, y) => (x >= y ? x : y);  
```  
Проведите рефакторинг функции `max` таким образом, чтобы она не нуждалась упоминании аргументов.
  
```js  
const max = xs => reduce((acc, x) => (x >= acc ? x : acc), -Infinity, xs);  
```  
