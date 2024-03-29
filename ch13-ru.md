# Глава 13: Моноиды объединяют все вместе

## Дикая комбинация

В этой главе мы рассмотрим *моноиды* на примере *семигрупп*. *Моноиды* - это жвачка в волосах математической абстракции. Они фиксируют идею, которая охватывает множество дисциплин, образно и буквально объединяя их все вместе. Они - зловещая сила, связывающая все, что вычисляется. Кислород в нашей кодовой базе, земля, на которой она работает, закодированная квантовая запутанность.

*Моноиды - это сочетание. Но что такое комбинация? Это может означать так много вещей: от накопления до конкатенации, от умножения до выбора, композиции, упорядочивания, даже оценки! Здесь мы увидим множество примеров, но мы лишь на цыпочках пройдемся по подножию горы моноидов. Примеров много, а приложений - огромное количество. Цель этой главы - дать хорошую интуицию, чтобы вы могли создать несколько собственных *моноидов*.

## Абстрактное дополнение

Сложение обладает некоторыми интересными свойствами, которые я хотел бы обсудить. Давайте посмотрим на него через очки абстракции.

Для начала, это бинарная операция, то есть операция, которая принимает два значения и возвращает одно значение, все в пределах одного набора.

```js
// бинарный операции
1 + 1 = 2
```

Видите? Два значения в домене, одно значение в кодомене, все это один и тот же набор - числа, так сказать. Кто-то может сказать, что числа "замкнуты на сложение", то есть их тип никогда не изменится, независимо от того, какие числа будут добавлены. Это означает, что мы можем выполнять цепочку операций, поскольку результатом всегда будет другое число:

```js
// мы можем запустить это на любом количестве чисел
1 + 7 + 5 + 4 + ...
```

В дополнение к этому (какой каламбур...), у нас есть ассоциативность, которая дает нам возможность группировать операции по своему усмотрению. Кстати, ассоциативные, бинарные операции - это рецепт для параллельных вычислений, потому что мы можем разбивать и распределять работу.

```js
// ассоциативность
(1 + 2) + 3 = 6
1 + (2 + 3) = 6
```

Не путайте это с коммутативностью, которая позволяет нам менять порядок. Это свойство справедливо и для сложения, но сейчас оно нас не особенно интересует - слишком специфично для наших абстракций.

Если подумать, какие свойства вообще должны быть в нашем абстрактном суперклассе? Какие свойства специфичны для сложения, а какие могут быть обобщены? Есть ли среди этой иерархии другие абстракции или это все один кусок? Именно такие размышления применяли наши предки-математики, когда придумывали интерфейсы в абстрактной алгебре.

Так получилось, что те абстракционисты старой школы при абстрагировании сложения остановились на концепции *группы*. У *группы* есть все возможности, включая концепцию отрицательных чисел. Здесь нас интересует только ассоциативный бинарный оператор, поэтому мы выберем менее специфичный интерфейс *Semigroup*. *Semigroup* - это тип с методом `concat`, который действует как наш ассоциативный двоичный оператор.

Давайте реализуем его для сложения и назовем его `Sum`:

```js
const Sum = x => ({
  x,
  concat: other => Sum(x + other.x)
})
```

Обратите внимание, что мы `concat` с некоторым другим `Sum` и всегда возвращаем `Sum`.

Я использовал здесь фабрику объектов вместо нашей типичной церемонии прототипов, прежде всего потому, что `Sum` не *ориентирована* и мы не хотим вводить `new`. В любом случае, вот он в действии:

```js
Sum(1).concat(Sum(3)) // Sum(4)
Sum(4).concat(Sum(37)) // Sum(41)
```

Вот так мы можем программировать на интерфейс, а не на реализацию. Поскольку этот интерфейс пришел из теории групп, он имеет многовековую литературу, поддерживающую его. Бесплатные документы!

Теперь, как уже упоминалось, `Sum` не является ни *ориентированным*, ни *функтором*. В качестве упражнения, вернитесь назад и проверьте законы, чтобы понять, почему. Хорошо, я просто скажу вам: она может хранить только число, поэтому `map` здесь не имеет смысла, так как мы не можем преобразовать базовое значение к другому типу. Это была бы очень ограниченная `map`!

Так почему же это полезно? Как и в случае с любым интерфейсом, мы можем менять местами наши экземпляры для достижения различных результатов:

Переведено с помощью www.DeepL.com/Translator (бесплатная версия)

```js
const Product = x => ({ x, concat: other => Product(x * other.x) })

const Min = x => ({ x, concat: other => Min(x < other.x ? x : other.x) })

const Max = x => ({ x, concat: other => Max(x > other.x ? x : other.x) })
```

Однако это не ограничивается числами. Давайте посмотрим на другие типы:

```js
const Any = x => ({ x, concat: other => Any(x || other.x) })
const All = x => ({ x, concat: other => All(x && other.x) })

Any(false).concat(Any(true)) // Any(true)
Any(false).concat(Any(false)) // Any(false)

All(false).concat(All(true)) // All(false)
All(true).concat(All(true)) // All(true)

[1,2].concat([3,4]) // [1,2,3,4]

"miracle grow".concat("n") // miracle grown"

Map({day: 'night'}).concat(Map({white: 'nikes'})) // Map({day: 'night', white: 'nikes'})
```

Если долго смотреть на них, узор выскочит на вас, как волшебный плакат. Она повсюду. Мы объединяем структуры данных, комбинируем логику, строим строки... кажется, что почти любую задачу можно впихнуть в этот интерфейс, основанный на комбинациях.

Я уже несколько раз использовал `Map`. Прошу прощения, если вы не были должным образом представлены. `Map` просто оборачивает `Object`, чтобы мы могли приукрасить его некоторыми дополнительными методами, не изменяя ткань вселенной.


## Все мои любимые функторы являются полугруппами

Все типы, которые мы уже видели, реализующие интерфейс функтора, также реализуют интерфейс полугруппы. Давайте рассмотрим `Identity` (исполнитель, ранее известный как Container):

```js
Identity.prototype.concat = function(other) {
  return new Identity(this.__value.concat(other.__value))
}

Identity.of(Sum(4)).concat(Identity.of(Sum(1))) // Identity(Sum(5))
Identity.of(4).concat(Identity.of(1)) // TypeError: this.__value.concat is not a function
```

Она является *полугруппой* тогда и только тогда, когда ее `__значение` является *полугруппой*. Подобно дельтаплану с масляными пальцами, он является таковым, пока удерживает его.

Другие типы имеют аналогичное поведение:

```js
// сочетание с обработкой ошибок
Right(Sum(2)).concat(Right(Sum(3))) // Right(Sum(5))
Right(Sum(2)).concat(Left('some error')) // Left('some error')


// комбинирование с асинхронностью
Task.of([1,2]).concat(Task.of([3,4])) // Task([1,2,3,4])
```

Это становится особенно полезным, когда мы складываем эти полугруппы в каскадную комбинацию:

```js
// formValues :: Selector -> IO (Map String String)
// validate :: Map String String -> Either Error (Map String String)

formValues('#signup').map(validate).concat(formValues('#terms').map(validate)) // IO(Right(Map({username: 'andre3000', accepted: true})))
formValues('#signup').map(validate).concat(formValues('#terms').map(validate)) // IO(Left('one must accept our totalitarian agreement'))

serverA.get('/friends').concat(serverB.get('/friends')) // Task([friend1, friend2])

// loadSetting :: String -> Task Error (Maybe (Map String Boolean))
loadSetting('email').concat(loadSetting('general')) // Task(Maybe(Map({backgroundColor: true, autoSave: false})))
```

В верхнем примере мы объединили `IO` с `Either` и `Map` для проверки и объединения значений формы. Далее мы обратились к нескольким различным серверам и объединили их результаты асинхронным способом, используя `Task` и `Array`. Наконец, мы объединили `Task`, `Maybe` и `Map` для загрузки, разбора и объединения нескольких параметров.

Это могут быть `цепочки` или `ap`, но *полугруппы* передают то, что мы хотели бы сделать гораздо более лаконично.

Это выходит за рамки функторов. На самом деле, оказывается, что все, что полностью состоит из полугрупп, само является полугруппой: если мы можем объединить набор, то мы можем объединить и все остальное.

```js
const Analytics = (clicks, path, idleTime) => ({
  clicks,
  path,
  idleTime,
  concat: other =>
    Analytics(clicks.concat(other.clicks), path.concat(other.path), idleTime.concat(other.idleTime))
})

Analytics(Sum(2), ['/home', '/about'], Right(Max(2000))).concat(Analytics(Sum(1), ['/contact'], Right(Max(1000))))
// Analytics(Sum(3), ['/home', '/about', '/contact'], Right(Max(2000)))
```

Видите, все знает, как красиво объединить себя. Оказывается, мы можем сделать то же самое бесплатно, просто используя тип `Map`:

```js
Map({clicks: Sum(2), path: ['/home', '/about'], idleTime: Right(Max(2000))}).concat(Map({clicks: Sum(1), path: ['/contact'], idleTime: Right(Max(1000))}))
// Map({clicks: Sum(3), path: ['/home', '/about', '/contact'], idleTime: Right(Max(2000))})
```

Мы можем складывать и комбинировать их столько, сколько захотим. Это просто вопрос добавления еще одного дерева в лес или еще одного пламени в лесной пожар, в зависимости от вашей кодовой базы.

Интуитивно понятное поведение по умолчанию - объединять то, что содержит тип, однако есть случаи, когда мы игнорируем то, что находится внутри, и объединяем сами контейнеры. Рассмотрим такой тип, как `Stream`:

```js
const submitStream = Stream.fromEvent('click', $('#submit'))
const enterStream = filter(x => x.key === 'Enter', Stream.fromEvent('keydown', $('#myForm')))

submitStream.concat(enterStream).map(submitForm) // Stream()
```

Мы можем объединить потоки событий, фиксируя события из обоих потоков как один новый поток. В качестве альтернативы мы могли бы объединить их, настаивая на том, что они содержат полугруппу. На самом деле, существует множество возможных экземпляров для каждого типа. Рассмотрим `Task`, мы можем объединить их, выбрав более ранний или более поздний из двух. Мы всегда можем выбрать первое `Право` вместо замыкания на `Лево`, что имеет эффект игнорирования ошибок. Существует интерфейс под названием *Alternative*, который реализует некоторые из этих, ну, альтернативных экземпляров, обычно ориентированных на выбор, а не на каскадное комбинирование. На него стоит обратить внимание, если вам нужна такая функциональность.

## Моноиды просто так

Мы абстрагировали сложение, но, подобно вавилонянам, нам не хватало понятия нуля (было ноль упоминаний о нем).

Ноль действует как *идентичность*, то есть любой элемент, добавленный к `0`, вернет обратно тот же самый элемент. С точки зрения абстракции, полезно думать о `0` как о некоем нейтральном или *пустом* элементе. Важно, чтобы он действовал одинаково с левой и правой стороны нашей бинарной операции:

```js
// идентичность
1 + 0 = 1
0 + 1 = 1
```

Назовем эту концепцию `empty` и создадим с ее помощью новый интерфейс. Как и многие другие стартапы, мы выберем чудовищно неинформативное, но удобное для гугла название: *Monoid*. Рецепт *Monoid* заключается в том, чтобы взять любую *полугруппу* и добавить специальный элемент *идентичности*. Мы реализуем это с помощью функции `empty` для самого типа:

```js
Array.empty = () => []
String.empty = () => ""
Sum.empty = () => Sum(0)
Product.empty = () => Product(1)
Min.empty = () => Min(Infinity)
Max.empty = () => Max(-Infinity)
All.empty = () => All(true)
Any.empty = () => Any(false)
```

Когда пустая, тождественная величина может оказаться полезной? Это все равно, что спросить, почему ноль полезен. Как и вообще ничего не спрашивать...

Когда у нас больше ничего нет, на кого мы можем рассчитывать? На ноль. Сколько жучков нам нужно? Ноль. Это наша терпимость к небезопасному коду. Свежий старт. Конечная цена. Он может уничтожить все на своем пути или спасти нас в трудную минуту. Золотой спасательный круг и яма отчаяния.

В кодовом отношении они соответствуют разумным значениям по умолчанию:

```js
const settings = (prefix="", overrides=[], total=0) => ...

const settings = (prefix=String.empty(), overrides=Array.empty(), total=Sum.empty()) => ...
```

Или чтобы вернуть полезное значение, когда у нас больше ничего нет:

```js
sum([]) // 0
```

Они также являются идеальным начальным значением для аккумулятора...

## Собираем всё вместе

Так получилось, что `concat` и `empty` идеально вписываются в первые два слота `reduce`. На самом деле мы можем `reduce` массив *semigroup*, игнорируя значение *empty*, но, как вы видите, это приводит к опасной ситуации:

```js
// concat :: Semigroup s => s -> s -> s
const concat = x => y => x.concat(y)

[Sum(1), Sum(2)].reduce(concat) // Sum(3)

[].reduce(concat) // TypeError: Reduce of empty array with no initial value
```

Бум динамита. Как вывихнутая лодыжка в марафоне, мы получили исключение времени выполнения. JavaScript более чем счастлив позволить нам пристегнуть пистолеты к кроссовкам перед бегом - это консервативный язык, я полагаю, но он останавливает нас на месте, когда массив оказывается бесплодным. Что же он может вернуть? `NaN`, `false`, `-1`? Если бы мы продолжили работу в нашей программе, мы бы хотели получить результат нужного типа. Он мог бы вернуть `Maybe`, чтобы указать на возможность неудачи, но мы можем сделать кое-что получше.

Давайте воспользуемся нашей каррированной функцией `reduce` и сделаем безопасную версию, в которой значение `empty` не является необязательным. Отныне она будет называться `fold`:

```js
// fold :: Monoid m => m -> [m] -> m
const fold = reduce(concat)
```

Начальное `m` - это наше `пустое` значение - наша нейтральная, отправная точка, затем мы берем массив `m` и дробим их до одного красивого значения, похожего на бриллиант.

```js
fold(Sum.empty(), [Sum(1), Sum(2)]) // Sum(3)
fold(Sum.empty(), []) // Sum(0)

fold(Any.empty(), [Any(false), Any(true)]) // Any(true)
fold(Any.empty(), []) // Any(false)


fold(Either.of(Max.empty()), [Right(Max(3)), Right(Max(21)), Right(Max(11))]) // Right(Max(21))
fold(Either.of(Max.empty()), [Right(Max(3)), Left('error retrieving value'), Right(Max(11))]) // Left('error retrieving value')

fold(IO.of([]), ['.link', 'a'].map($)) // IO([<a>, <button class="link"/>, <a>])
```

Мы предоставили ручное значение `empty` для последних двух, поскольку мы не можем определить его в самом типе. Это совершенно нормально. Типизированные языки могут определить это сами, но мы должны передать его здесь.

## Не совсем моноид

Есть некоторые *полугруппы*, которые не могут стать *моноидами*, то есть дать начальное значение. Посмотрите на `First`:

```js
const First = x => ({ x, concat: other => First(x) })

Map({id: First(123), isPaid: Any(true), points: Sum(13)}).concat(Map({id: First(2242), isPaid: Any(false), points: Sum(1)}))
// Map({id: First(123), isPaid: Any(true), points: Sum(14)})
```

Мы объединим несколько счетов и сохраним идентификатор `First`. Не существует способа определить для него `пустое` значение. Но это не значит, что оно бесполезно.


## Великая объединяющая теория

## Теория групп или теория категорий?

Понятие бинарной операции встречается повсюду в абстрактной алгебре. Фактически, это первичная операция для *категории*. Однако мы не можем моделировать нашу операцию в теории категорий без *идентичности*. По этой причине мы начинаем с полугруппы из теории групп, а затем переходим к моноиду в теории категорий, когда у нас появляется *пустота*.

Моноиды образуют однообъектную категорию, где морфизм - `concat`, `empty` - тождество, и композиция гарантирована.

### Композиция как моноид

Функции типа `a -> a`, где домен находится в том же множестве, что и кодомен, называются *эндоморфизмами*. Мы можем создать *моноид* под названием `Endo`, который отражает эту идею:

```js
const Endo = run => ({
  run,
  concat: other =>
    Endo(compose(run, other.run))
})

Endo.empty = () => Endo(identity)


// в действии

// thingDownFlipAndReverse :: Endo [String] -> [String]
const thingDownFlipAndReverse = fold(Endo(() => []), [Endo(reverse), Endo(sort), Endo(append('thing down')])

thingDownFlipAndReverse.run(['let me work it', 'is it worth it?'])
// ['thing down', 'let me work it', 'is it worth it?']
```

Поскольку все они одного типа, мы можем `concat` через `compose`, и типы всегда совпадают.

### Монада как моноид

Возможно, вы заметили, что `join` - это операция, которая берет две (вложенные) монады и ассоциативно сжимает их до одной. Это также естественное преобразование или "функция-функтор". Как уже говорилось ранее, мы можем создать категорию функторов как объектов с естественными преобразованиями в качестве морфизмов. Теперь, если мы специализируем ее до *Endofunctors*, то есть функторов одного типа, то `join` дает нам моноид в категории Endofunctors, также известный как монада. Чтобы показать точную формулировку в коде, нужно немного повозиться, и я советую вам погуглить, но общая идея такова.

### Аппликатив как моноид

Даже аппликативные функции имеют моноидальную формулировку, известную в теории категорий как *лакс-моноидальный вектор*. Мы можем реализовать интерфейс как моноид и восстановить `ap` из него:

```js
// concat :: f a -> f b -> f [a, b]
// empty :: () -> f ()

// ap :: Functor f => f (a -> b) -> f a -> f b
const ap = compose(map(([f, x]) => f(x)), concat)
```


## Итог

Итак, вы видите, что все связано или может быть связано. Это глубокое осознание делает *моноиды* мощным инструментом моделирования от широких областей архитектуры приложений до мельчайших фрагментов данных. Я призываю вас думать о *моноидах* всякий раз, когда прямое накопление или комбинация являются частью вашего приложения, а затем, когда вы освоите это, начните распространять определение на большее количество приложений (вы будете удивлены, как много можно смоделировать с помощью *моноида*).

## Упражнения

