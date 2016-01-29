# Глава 6: Пример приложения

## Декларативное программирование

Мы приближаемся к изменению образа нашего мышления. С этого момента мы прекращаем говорить компьютеру как делать его работу, вместо того, чтобы указывать ему как вернуть результат. Я уверен, что это будет вызывать у вас меньше стресса, нежели попытки управлять всеми мелочами постоянно.

Декларативный подход, то есть противоположный императивному означает, что мы будем писать выражения, вместо пошаговых инструкций.

Вспомните SQL. Там нет «Сначала сделай то, затем сделай это». Есть выражение, которое определяет, что бы мы хотели сделать с базой данных. Мы не решаем как сделать работу, выражение делает это. Если база данных обновится и движок SQL изменится, то нам не придётся редактировать запросы. Всё происходит так, потому что существует много путей интерпретировать наши условия и получить такой же результат.

Многим, включая меня, может показаться сложным осмыслить концепцию декларативного программирования вот так сразу. Поэтому, давайте обратимся к примерам, чтобы получить более точное представление.

```js
// императивная парадигма
var makes = [];
for (i = 0; i < cars.length; i++) {
  makes.push(cars[i].make);
}

// декларативная парадигма
var makes = cars.map(function(car){ return car.make; });
```

Императивный цикл сначала инициализирует массив. Интерпретатор должен выполнить это выражение перед тем как двигаться дальше. Затем он непосредственно проходит по списку автомобилей, вручную увеличивая счетчик, открыто показывая нам каждую часть итерации.

Вариант с методом `map` – это одно выражение. Оно не требует особого порядка выполнения. В этом столько свободы – вы определяете как проходить по списку и как можно сформировать массив. Выражение определяет *что*, а не *как*. Таким образом, оно использует блестящий декларативный подход.

К тому же, декларативный вариант более краток и ясен. Вы можете оптимизировать функцию `map` по вашему желанию, при этом вам не потребуется менять код самого приложения

Для тех, кто думает: «Да, но ведь это гораздо быстрее – написать императивный цикл», я предлагаю ознакомиться с тем, как JIT оптимизирует ваш код. Вот [ужасное видео, которое может пролить свет на вас](https://www.youtube.com/watch?v=65-RbBwZQdU)

Давайте рассмотрим другой пример.

```js
// императивная парадигма
var authenticate = function(form) {
  var user = toUser(form);
  return logIn(user);
};

// декларативная парадигма
var authenticate = compose(logIn, toUser);
```

Хотя в императивной версии и нет чего-то явно неправильного, это по-прежнему объединение пошаговых команд. Выражение с `compose` просто утверждает факт: аутентификация — это композиция функций `toUser` и `logIn`. Опять же, это оставляет нам пространство для манёвра в поддержке изменений кода и оставляет результаты работы нашего приложения конкретными.

По причине того, что мы не определяем порядок выполнения, декларативное программирование подходит для параллельных вычислений. Объеденив этот подход с чистыми функциями, мы получаем функциональное программирование как хороший вариант для использования в асинхронных вычислениях, нам не нужно делать что-то особенное, чтобы получить параллельные или асинхронно выполняющие код системы.

## Flickr, написанный в функциональном стиле

Сейчас мы построим пример приложения декларативным, компонуемым способом. Пока что мы схитрим и будем использовать побочные эффекты, но мы будем использовать их по-минимуму и отделять от чистого кода. Мы собираемся написать виджет для браузера, который парсит изображения из flckr и выводит их на экран. Начнем с каркаса приложения. Это HTML нашего приложения:

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.1.11/require.min.js"></script>
    <script src="flickr.js"></script>
  </head>
  <body></body>
</html>
```

А это основа flickr.js:

```js
requirejs.config({
  paths: {
    ramda: 'https://cdnjs.cloudflare.com/ajax/libs/ramda/0.13.0/ramda.min',
    jquery: 'https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min'
  }
});

require([
    'ramda',
    'jquery'
  ],
  function (_, $) {
    var trace = _.curry(function(tag, x) {
      console.log(tag, x);
      return x;
    });
    // код приложения
  });
```

Мы выбираем [ramda](http://ramdajs.com) вместо lodash или других вспомогательных библиотек. Она содержит `compose`, `curry` и другие методы. Я использую requirejs, который может показаться излишним, но мы будем использовать его и в дальнейшем, а постоянство это ключ к успеху. Также я начал с объявления нашей прекрасной функции `trace` для облегчения отладки.

Следующий пункт в нашем пути - спецификация. Приложение будет иметь 4 действия.

1. Формировать url для конкретного поискового запроса
2. Делать вызов flickr api
3. Преобразовывать полученный json в html с изображениями
4. Выводить их на экран

Выше упомянуты 2 нечистых действия. Вы их видите? Получение данных с помощью api вызова и вывод их на экран. Давайте напишем их первыми, чтобы сразу отделить их.

```js
var Impure = {
  getJSON: _.curry(function(callback, url) {
    $.getJSON(url, callback);
  }),

  setHtml: _.curry(function(sel, html) {
    $(sel).html(html);
  })
};
```

Тут мы просто оборачиваем методы jQuery, чтобы они были каррируемыми и переставляем аргументы в более удобном порядке. Я выделил эти функции в отдельное пространство имён `Impure`, чтобы мы сразу замечали, что это опасные функции. В последующих примерах мы сделаем эти функции чистыми.

Далее мы должны собрать url-адрес, который му будем передавать нашей функции `Impure.getJSON`.

```js
var url = function (term) {
  return 'https://api.flickr.com/services/feeds/photos_public.gne?tags=' +
    term + '&format=json&jsoncallback=?';
};
```

Существуют различные причудливые и черезчур сложные методы написания функции `url` в стлие отсутствия ссылок используя моноиды[^Мы изучим их позже] или комбинаторы. Наш выбор – читаемая версия, собирающая эту строку в в обычном ссылочном стиле.

Давайте напишем функцию `app`, которая делает вызов и выводит данные на экран.

```js
var app = _.compose(Impure.getJSON(trace("response")), url);

app("cats");
```

Этот код вызывает функцию `url`, затем передает полученную строку в функцию `getJSON`, которая частично применяется с `trace`. В результате загрузки приложения, в консоли будет выведен ответ на api вызов.

<img src="images/console_ss.png"/>

Мы хотим получить изображения из этого json-объекта. Похоже, что их адреса запрятаны в элементах `items`, а конкретно – в каждом свойсте `m` объекта `media`.

Во всяком случае, для получения этих значений мы можем использовать замечательную универсальную функцию чтения из ramda, с именем `_.prop()`. Ниже представлена оригинальная версия метода:

```js
var prop = _.curry(function(property, object){
  return object[property];
});
```

Она достаточно скучная. Мы просто используем синтаксис квадратных скобок, чтобы получить доступ к свойству какого-либо объекта. Давайте используем её и получим адреса изображений.

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var srcs = _.compose(_.map(mediaUrl), _.prop('items'));
```

Как только мы собрали элементы `items`, мы проходимся по ним функцией `map`, чтобы получить адрес каждого изображения. На выходе – замечательный массив адресов. Давайте зацепим его к нашему приложению, чтобы выводить картинки прямо на экран.

```js
var renderImages = _.compose(Impure.setHtml("body"), srcs);
var app = _.compose(Impure.getJSON(renderImages), url);
```

Всё, что мы сделали – это очередная композиция функций, которая будет вызывать конструктор массива адресов `srcs` и заполнять ими содержимое `body` в нашей разметке. Мы заменили `trace` функцией `renderImages` и теперь мы выводим не только чистый json. Мы просто выводим ссылки на изображения прямо в тело приложения.

Последним шагом необходимо преобразовать эти ссылки в настоящие картинки. Если бы приложение было большим, мы бы использовали шаблонизатор и библиотеку для работы с DOM, например, Handlebars и React. В нашем же приложении, нам нужен только тег img, поэтому давайте обойдемся jQuery.

```js
var img = function (url) {
  return $('<img />', { src: url });
};
```

jQuery-метод `html()` принимает массив тегов. Нам нужно только вставить адреса изображений в теги и пустить их дальше в функцию `setHtml`.

```js
var images = _.compose(_.map(img), srcs);
var renderImages = _.compose(Impure.setHtml("body"), images);
var app = _.compose(Impure.getJSON(renderImages), url);
```

И мы закончили!

<img src="images/cats_ss.png" />

Ниже представлен полный скрипт:
```js
requirejs.config({
  paths: {
    ramda: 'https://cdnjs.cloudflare.com/ajax/libs/ramda/0.13.0/ramda.min',
    jquery: 'https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min'
  }
});

require([
    'ramda',
    'jquery'
  ],
  function (_, $) {
    ////////////////////////////////////////////
    // Utils

    var Impure = {
      getJSON: _.curry(function(callback, url) {
        $.getJSON(url, callback);
      }),

      setHtml: _.curry(function(sel, html) {
        $(sel).html(html);
      })
    };

    var img = function (url) {
      return $('<img />', { src: url });
    };

    var trace = _.curry(function(tag, x) {
      console.log(tag, x);
      return x;
    });

    ////////////////////////////////////////////

    var url = function (t) {
      return 'http://api.flickr.com/services/feeds/photos_public.gne?tags=' +
        t + '&format=json&jsoncallback=?';
    };

    var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

    var srcs = _.compose(_.map(mediaUrl), _.prop('items'));

    var images = _.compose(_.map(img), srcs);

    var renderImages = _.compose(Impure.setHtml("body"), images);

    var app = _.compose(Impure.getJSON(renderImages), url);

    app("cats");
  });
```

А теперь посмотрите на это. Красивая декларативная спецификация, показывающая, чем являются элементы, а не то как они стали такими(?). Сейчас мы воспринимаем каждую строку, как уравнение, с параметрами, которые они используют(?). Мы можем использовать эти параметры, для обсуждения нашего приложения и рефакторинга.

## A Principled Refactor

There is an optimization available - we map over each item to turn it into a media url, then we map again over those srcs to turn them into img tags. There is a law regarding map and composition:


```js
// map's composition law
var law = compose(map(f), map(g)) == map(compose(f, g));
```

We can use this property to optimize our code. Let's have a principled refactor.

```js
// original code
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var srcs = _.compose(_.map(mediaUrl), _.prop('items'));

var images = _.compose(_.map(img), srcs);

```

Let's line up our maps. We can inline the call to `srcs` in `images` thanks to equational reasoning and purity.

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var images = _.compose(_.map(img), _.map(mediaUrl), _.prop('items'));
```

Now that we've lined up our `map`'s we can apply the composition law.

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var images = _.compose(_.map(_.compose(img, mediaUrl)), _.prop('items'));
```

Now the bugger will only loop once while turning each item into an img. Let's just make it a little more readable by extracting the function out.

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var mediaToImg = _.compose(img, mediaUrl);

var images = _.compose(_.map(mediaToImg), _.prop('items'));
```

## In Summary

We have seen how to put our new skills into use with a small, but real world app. We've used our mathematical framework to reason about and refactor our code. But what about error handling and code branching? How can we make the whole application pure instead of merely namespacing destructive functions? How can we make our app safer and more expressive? These are the questions we will tackle in part 2.

[Chapter 7: Hindley-Milner and Me](ch7.md)
