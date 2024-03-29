# Глава 07: Хиндли-Милнер и Я

## Каков твой тип?

Когда вы новичок в функциональном мире, вы достаточно быстро оказываетесь по колено в сигнатурах типов. Типы — это метаязык, который позволяет людям с самыми различными фоновыми знаниями общаться ёмко и эффективно. Чаще всего типы записываются с помощью системы Хиндли-Милнера, которую мы рассмотрим в этой главе.

При работе именно с чистыми функциями, сигнатуры типов обладают такой выразительной силой, с которой не сравнится ни один натуральный язык. Сигнатура поведает вам все секреты поведения и назначения функции в одной простой и компактной строке. Мы можем вывести «свободные теоремы» из них. В системе Хиндли-Милнера типы могут быть выведены *(в общем случае в компилируемых языках)*, и тогда можно обойтись без явных аннотаций типов. Сами типы можно описать очень конкретно и точно, а можно оставить обобщенными и абстрактными *(без ущерба для смысла и надёжности)*. Они работают не только на этапе компиляции *(как это происходит, например, в Haskell)*, но также нередко оказываются наилучшей возможной документацией. Таким образом, сигнатуры типов играют важную роль в функциональном программировании - гораздо большую, чем может показаться вначале.

JavaScript — это динамический язык, в нём переменные не имеют типов, но это не значит, что мы можем избегать или игнорировать факт, что значения имеют тип. Мы все еще работаем со строками, числами, булевыми значениями и т.д. Динамическая типизация означает только то, что у нас нет поддержки со стороны языка, и мы должны держать информацию о типах в уме. Но не стоит расстраиваться - мы можем использовать сигнатуры типов для документирования кода, и, таким образом, не потеряем информацию о типах.

Для JavaScript давно существуют как инструменты проверки типов, к которым относится [Flow](https://flow.org/), так и типизированные диалекты, как [TypeScript](https://www.typescriptlang.org/). Однако целью этой книги является предоставление инструментов для написания функционального кода, поэтому мы будем придерживаться системы типов, характерной для функциональных языков *(в то время как Flow и TypeScript явно подходят для ООП — прим. пер.)*.

## Байки из cкрипта

С пыльных страниц математических томов, через бескрайнее море технических документов, среди случайных блоговых постов по субботним утрам, вплоть до самого исходного кода мы находим сигнатуры типов Хиндли-Милнера. Систему, которая довольно проста, но всё же требует краткого объяснения и некоторой практики, чтобы полностью принять в работу этот небольшой язык.

```js
// capitalize :: String -> String
const capitalize = s => toUpperCase(head(s)) + toLowerCase(tail(s));

capitalize('smurf'); // 'Smurf'
```

Здесь `capitalize` принимает `String` и возвращает `String`. Не думайте о реализации - нас интересует сигнатура типов. 

В системе Хиндли-Милнера функции записываются как `a -> b`, где `a` и `b` - это обозначения каких-либо типов. Таким образом, сигнатура `capitalize` может быть прочтена как «функция из `String` в `String`». Другими словами, она принимает `String` в качестве аргумента и возвращает `String` в качестве результата.

Давайте рассмотрим сигнатуры ещё нескольких функций:

```js
// strLength :: String -> Number
const strLength = s => s.length;

// join :: String -> [String] -> String
const join = curry((what, xs) => xs.join(what));

// match :: Regex -> String -> [String]
const match = curry((reg, s) => s.match(reg));

// replace :: Regex -> String -> String -> String
const replace = curry((reg, sub, s) => s.replace(reg, sub));
```

`strLength` читается подобным образом: принимает `String` и возвращает `Number`.

Остальные примеры поначалу озадачат вас. Но, даже когда вы не можете понять все особенности сигнатуры, вы всегда можете посмотреть на тип, который указан последним — это и есть тип возвращаемого значения. Так что сигнатура `match` может быть прочитана так: она берёт `Regex` и `String` и возвращает вам `[String]`. Но здесь происходит нечто интересное, на чем я хотел бы задержаться.

Для `match` мы можем сгруппировать сигнатуру следующим образом:

```js
// match :: Regex -> (String -> [String])
const match = curry((reg, s) => s.match(reg));
```

Другое дело! Заключив последнюю часть сигнатуры в скобки, мы проясняем больше подробностей. Теперь мы видим `match` как функцию, которая принимает `Regex` и возвращает функцию из `String` в `[String]`. Именно так и работают каррированные функции. Разумеется, нам не обязательно каждый раз мысленно представлять каждую вновь возвращаемую функцию, но важно понимать, почему именно тип, указанный последним, соответствует итоговому значению.

```js
// match :: Regex -> (String -> [String])
// onHoliday :: String -> [String]
const onHoliday = match(/holiday/ig);
```

Каждое применение такой функции к аргументу откусывает один тип от левой части сигнатуры. `onHoliday` — это `match`, у которого уже есть `Regex`.

```js
// replace :: Regex -> (String -> (String -> String))
const replace = curry((reg, sub, s) => s.replace(reg, sub));
```

Как вы можете заметить, в случае `replace` расстановка скобок окажется загромождающей и избыточной, поэтому мы просто их опустим. Сигнатура будет прочитана так: `replace` принимает  `Regex`, `String`, другой `String`, и в конце возвращает `String`.

И ещё немного:

```js
// id :: a -> a
const id = x => x;

// map :: (a -> b) -> [a] -> [b]
const map = curry((f, xs) => xs.map(f));
```

Функция `id` принимает любой тип `a` и возвращает значение того же типа `a`. Мы можем использовать переменные для обозначения типов так же, как используем их для значений. Имена переменных, такие как `a` и `b`, являются условными, но они произвольны и могут быть заменены любым именем, каким захотите. Если два значения имеют тип, обозначенный одной переменной, то они обязательно должны быть одного типа. Это очень важное требование, поэтому давайте повторим: `a -> b` означает функцию из любого типа `a` в любой тип `b`, но `a -> a` означает, что функция должна принимать и возвращать значения одного и того же типа. Таким образом, `id` может иметь тип `String -> String` или `Number -> Number`, но не `String -> Bool`.

`map` использует переменные типа *(типовые переменные, type variables)* подобным образом, но в этот раз появляется ещё и `b`, который может совпадать, а может и не совпадать с типом `a`. Мы можем прочитать это так: `map` принимает функцию из любого типа `a` в любой тип `b`, затем принимает массив `a` и в результате возвращает массив `b`.

Надеюсь, вы были поражены выразительной красотой этой сигнатуры. Она почти дословно сообщает нам, что функция делает. Ей дана функция от `a` до `b`, массив `a`, и она производит для нас массив `b`. Единственный разумный способ прийти к такому результату — применить эту самую функцию для каждого `a`. Любая другая реализация была бы наглой ложью *(потому что для фильтрации или сортировки массива потребуется предикат или компаратор, которые должны знать про тип `a` или тип `b`. Если же она их не принимает в аргументах, то либо `a` и `b` не являются любыми, либо функция не является чистой. Без возможности оценивать `a` или `b` можно только обратить порядок массива или предопределённым образом изменить количество элементов, что было бы неразумным в функции общего назначения — прим. пер.)*.

Умение рассуждать о типах и о том, что из них следует — это навык, с которым вы далеко пойдёте в функциональном мире. Не только статьи, блоги, документация и т.д. станут более доступными, но и сами сигнатуры будут рассказывать вам о функциональности. Потребуется практика, чтобы читать их бегло, но, если вы уделите этому время, куча информации станет доступной вам без обращения к справочникам.

Вот еще несколько, попробуйте расшифровать их самостоятельно.

```js
// head :: [a] -> a
const head = xs => xs[0];

// filter :: (a -> Bool) -> [a] -> [a]
const filter = curry((f, xs) => xs.filter(f));

// reduce :: ((b, a) -> b) -> b -> [a] -> b
const reduce = curry((f, x, xs) => xs.reduce(f, x));
```

`reduce`, пожалуй, самая выразительная из всех. Однако это не самая простая функция, так что не чувствуйте себя неадекватным, если вам придётся поломать голову над её сигнатурой. Я постараюсь объяснить простым языком, однако самостоятельный разбор не менее полезен.

Глядя на сигнатуру `reduce`, мы видим, что первый аргумент должен быть функцией, которая ожидает от нас `b` и `a`, и производит из них `b`. И где же взять эти `a` и `b`? Ну, следом за первым аргументом принимаются `b` и целый массив `a`, так что мы можем предположить, что этот `b` и все эти `a` будут скормлены функции (первому аргументу). Мы также видим, что в итоге `reduce` должна вернуть `b`, так что, похоже, переданная в первом аргументе функция и произведёт для нас итоговое значение. С учётом наших знаний о том, как работает reduce, мы можем заключить, что наши предположения верны. *(Стоит отметить, что `f` не может быть каррированной, если она используется с `Array.prototype.reduce`, поэтому свои аргументы `b` и `a` она принимает «вместе» — прим. пер.)*

## Сужение возможностей

С того момента, как мы ввели понятие типовой переменной, открывается любопытное свойство, называемое *[параметричностью](https://ru.wikipedia.org/wiki/%D0%9F%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B9_%D0%BF%D0%BE%D0%BB%D0%B8%D0%BC%D0%BE%D1%80%D1%84%D0%B8%D0%B7%D0%BC)*. Это свойство гласит, что функция будет *вести себя со всеми типами единообразно*. Давайте разберёмся с этим:

```js
// head :: [a] -> a
```

Посмотрев на `head`, мы увидим, что она преобразует `[a]` в `a`. Кроме упоминания конкретного типа `[]`, не предоставлено никакой информации, следовательно, функциональность ограничена только работой с массивами. Что вообще можно сделать со значением типа `a`, когда о нём не известно ничего? Другими словами, `a` говорит нам, что аргумент не может иметь какой-то **определённый** тип *(так как нет оснований для указания точного типа)*, что означает, что это может быть **любой** тип, а это значит, что функция должна работать одинаково для **каждого** мыслимого типа `a`. В этом и заключается **параметричность**. Если размышлять о реализации, единственное, что представляется разумным - выбрать первый, последний или случайный элемент из этого массива. Имя `head` должно подсказать нам, какой именно.

Вот ещё одна: 

```js
// reverse :: [a] -> [a]
```

Исходя только из сигнатуры, что может делать `reverse`? Как и в предыдущем примере, она не может сделать ничего определённого с `a`. Она не может изменить `a` на какой-то другой тип. Может, сортирует? Нет, она не располагает достаточной информацией для сравнения значений всех возможных типов. Может, перестраивает порядок каким-то другим образом? Полагаю, она может это делать, но тогда она должна обеспечить повторение этого результата из раза в раз. Ещё одна возможность, которая приходит на ум, — это удаление или дублирование элемента. В любом случае, суть в том, что возможное поведение функции в значительной степени сужается из-за полиморфности её типа.

Такое сужение возможностей позволяет использовать системы поиска по сигнатурам типов вроде [Hoogle](https://hoogle.haskell.org/), чтобы найти нужную функцию. Информация, упакованная в сигнатуры — действительно мощное средство.

## Бесплатно, как в теореме

Помимо определения возможной реализации, рассуждения о типах дают нам так называемые **«свободные теоремы»** *(утверждения вида «любая функция, имеющая данный тип, обладает таким-то свойством»)*. Ниже приводится несколько примеров теорем, взятых непосредственно из [работы Филипа Вадлера](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf) по этой теме.

```js
// head :: [a] -> a
compose(f, head) === compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) === compose(filter(p), map(f));
```

Вам не нужен код для обретения этих теорем, они следуют непосредственно из типов. Первый пример объявляет: если взять первый элемент массива (`head`), а затем применить к нему некоторую функцию `f`, то мы получим (намного быстрее) тот же результат, как если бы сначала применили `f` к каждому элементу (`map (f)`), а затем взяли первый элемент из получившихся.

Вы можете подумать, что это следует из здравого смысла. Но, когда я проверял, здравого смысла у компьютеров не было. Всё верно — для применения таких оптимизаций компьютерам требуется формализация. Математика позволяет формализовать интуитивное, что полезно в жёстких условиях компьютерной логики.

Правило для `filter` очень похожее. Оно объявляет, что, если мы отфильтруем массив предикатом в виде композиции `f` и `p`, а затем к оставшимся элементам применим `f` с помощью `map` (напоминаю, `filter` не изменяет элементы, это следует из его сигнатуры), то получим точно такой же результат, как если бы мы применили ко всем элементам `f` а затем отфильтровали с помощью предиката `p`.

Это только два примера, но вы можете применять такое рассуждение к любой полиморфной сигнатуре типа, и это всегда будет уместно. Для JavaScript есть несколько инструментов для объявления правил оптимизации *(это касается средств транспиляции — прим. пер.)*. Можно также сделать это через саму функцию compose. Плоды висят низко, а возможности безграничны.

## Ограничения для типов

Последнее, что следует отметить, — это то, что мы можем ограничивать типы интерфейсом *(в мире ФП эту роль играют тайпклассы — прим. пер.)*.

```js
// sort :: Ord a => [a] -> [a]
```

То, что мы видим слева от жирной стрелки, это утверждение факта: тип `a` должен быть `Ord`. Или, другими словами, `a` должен реализовывать интерфейс `Ord`. Что такое `Ord`, и откуда он взялся? В типизированных языках это было бы определённым интерфейсом, который говорит, что мы можем определить порядок для таких значений *(установить, что одно меньше, больше или равно другому)*. Это не только даёт нам больше информации об `a` и о том, чем занимается `sort`, но и ограничивает область определения функции *(function domain)*. Мы называем такие объявления интерфейса **ограничениями типа** *(type constraints)*.

```js
// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
```

Здесь у нас есть два ограничения: `Eq` и `Show`. Это гарантирует, что мы можем проверить равенство значений нашего типа `а`, и отобразить разницу, если они не равны.

В следующих главах мы увидим больше примеров ограничений, и эта идея примет более ясную форму.

## Итог

Сигнатуры Хиндли-Милнера вездесущи в функциональном мире. Несмотря на то, что их легко читать и записывать, потребуется время, чтобы отточить технику понимания программ с помощью одних только сигнатур. С этого момента мы будем добавлять сигнатуры типов к каждой строке кода.

[Глава 08: Контейнеры](ch08-ru.md)
