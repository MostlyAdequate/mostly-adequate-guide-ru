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

Here we've simply wrapped jQuery's methods to be curried and we've swapped the arguments to a more favorable position. I've namespaced them with `Impure` so we know these are dangerous functions. In a future example, we will make these two functions pure.

Next we must construct a url to pass to our `Impure.getJSON` function.

```js
var url = function (term) {
  return 'https://api.flickr.com/services/feeds/photos_public.gne?tags=' +
    term + '&format=json&jsoncallback=?';
};
```

There are fancy and overly complex ways of writing `url` pointfree using monoids[^we'll learn about these later] or combinators. We've chosen to stick with a readable version and assemble this string in the normal pointful fashion.

Let's write an app function that makes the call and places the contents on the screen.

```js
var app = _.compose(Impure.getJSON(trace("response")), url);

app("cats");
```

This calls our `url` function, then passes the string to our `getJSON` function, which has been partially applied with `trace`. Loading the app will show the response from the api call in the console.

<img src="images/console_ss.png"/>

We'd like to construct images out of this json. It looks like the srcs are buried in `items` then each `media`'s `m` property.

Anyhow, to get at these nested properties we can use a nice universal getter function from ramda called `_.prop()`. Here's a homegrown version so you can see what's happening:

```js
var prop = _.curry(function(property, object){
  return object[property];
});
```

It's quite dull actually. We just use `[]` syntax to access a property on whatever object. Let's use this to get at our srcs.

```js
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var srcs = _.compose(_.map(mediaUrl), _.prop('items'));
```

Once we gather the `items`, we must `map` over them to extract each media url. This results in a nice array of srcs. Let's hook this up to our app and print them on the screen.

```js
var renderImages = _.compose(Impure.setHtml("body"), srcs);
var app = _.compose(Impure.getJSON(renderImages), url);
```

All we've done is make a new composition that will call our `srcs` and set the body html with them. We've replaced the `trace` call with `renderImages` now that we have something to render besides raw json. This will crudely display our srcs directly in the body.

Our final step is to turn these srcs into bonafide images. In a bigger application, we'd use a template/dom library like Handlebars or React. For this application though, we only need an img tag so let's stick with jQuery.

```js
var img = function (url) {
  return $('<img />', { src: url });
};
```

jQuery's `html()` method will accept an array of tags. We only have to transform our srcs into images and send them along to `setHtml`.

```js
var images = _.compose(_.map(img), srcs);
var renderImages = _.compose(Impure.setHtml("body"), images);
var app = _.compose(Impure.getJSON(renderImages), url);
```

And we're done!

<img src="images/cats_ss.png" />

Here is the finished script:
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

Now look at that. A beautifully declarative specification of what things are, not how they come to be. We now view each line as an equation with properties that hold. We can use these properties to reason about our application and refactor.

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
