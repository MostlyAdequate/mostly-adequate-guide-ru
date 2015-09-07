# Chapter 2: First Class Functions
# Глава 2: Функции первого класса

## A quick review
## Краткий обзор

When we say functions are "first class", we mean they are just like everyone else... so normal class[^coach?]. We can treat functions like any other data type and there is nothing particularly special about them - store them in arrays, pass them around, assign them to variables, what have you.

Когда мы говорим о функциях «первого класса» мы имеем в виду то, что они такие же как и все остальные...[^учитель, поясните!]. С такими функциями можно обращаться так же, как и с любым другим типом данных, в функциях первого класса нет ничего особенного: их можно хранить в массивах, передавать в функции в качестве аргумента, присваивать переменным — всё, что душе угодно.

That is JavaScript 101, but worth a mention as a quick code search on github will show the collective evasion, or perhaps widespread ignorance of the concept. Shall we go for a feigned example? We shall.

Пример ниже — это азы JavaScript. Тем не менее, поискав по коду на github, легко удостовериться, что большинство старательно игнорирует подобный подход. Соскучились по надуманным примерам? Пожалуйста:

```js
var hi = function(name){
  return "Hi " + name;
};

var greeting = function(name) {
  return hi(name);
};
```

Here, the function wrapper around `hi` in `greeting` is completely redundant. Why? Because functions are *callable* in JavaScript. When `hi` has the `()` at the end it will run and return a value. When it does not, it simply returns the function stored in the variable. Just to be sure, have a look-see:

Здесь совершенно не нужно оборачивать `hi` в функцию `greeting`. Почему? Потому что в JavaScript функции *вызываемые*. Если написать `hi` и добавить `()` на конце, то функция будет вызвана и вернёт какое-то значение. Если не дописывать скобки на конце, то будет возвращена сама функция, сохранённая в переменную. Убедимся в этом:

```js
hi;
// function(name){
//  return "Hi " + name
// }

hi("jonas");
// "Hi jonas"
```

Since `greeting` is merely turning around and calling `hi` with the very same argument, we could simply write:

Поскольку `greeting` не делает ничего, кроме вызова `hi` с тем же самым аргументом, то можно написать проще:

```js
var greeting = hi;


greeting("times");
// "Hi times"
```

In other words, `hi` is already a function that expects one argument, why place another function around it that simply calls `hi` with the same bloody argument? It doesn't make any damn sense. It's like donning your heaviest parka in the dead of July just to blast the air and demand an ice lolly.

Другими словами, `hi` — уже функция с одним аргументом, зачем же оборачивать её в ещё одну функцию, которая будет вызывать ту же `hi` с тем же аргументом? Бессмыслица какая-то. Это всё равно, что надеть самое тёплое пальто в середине июля, чтобы ловить каждый порыв ветерка и просить мороженное.

It is obnoxiously verbose and, as it happens, bad practice to surround a function with another function merely to delay evaluation. (We'll see why in a moment, but it has to do with maintenance.)

Оборачивать функцию другой функцией просто для того, чтобы отложить её вызов — это не только слишком многословно, но ещё и считается плохой практикой (чуть ниже вы поймёте, почему, но намекну: речь идёт о поддержке кода).

A solid understanding of this is critical before moving on, so let's see a few more fun examples excavated from npm modules.

Очень важно, чтобы вы поняли почему это так, прежде чем мы продолжим, поэтому позвольте мне привести несколько забавных примеров, которые я нашёл в существующих npm-пакетах:

```js
// невежа
var getServerStuff = function(callback){
  return ajaxCall(function(json){
    return callback(json);
  });
};

// прозрел!
var getServerStuff = ajaxCall;
```

The world is littered with ajax code exactly like this. Here is the reason both are equivalent:

Мир JavaScript засорён подобным кодом. Вот почему оба примера выше — одно и то же:

```js
// это строка
return ajaxCall(function(json){
  return callback(json);
});

// равносильна этой
return ajaxCall(callback);

// перепишем getServerStuff
var getServerStuff = function(callback){
  return ajaxCall(callback);
};

// что эквивалентно следующему
var getServerStuff = ajaxCall; // <-- смотри, мам, нет ()
```

And that, folks, is how it is done. Once more then we'll see why I'm so insistent.

Вот так это делается. Ещё один пример, затем я объясню почему же я так настаиваю.

```js
var BlogController = (function() {
  var index = function(posts) {
    return Views.index(posts);
  };

  var show = function(post) {
    return Views.show(post);
  };

  var create = function(attrs) {
    return Db.create(attrs);
  };

  var update = function(post, attrs) {
    return Db.update(post, attrs);
  };

  var destroy = function(post) {
    return Db.destroy(post);
  };

  return {
    index: index, show: show, create: create, update: update, destroy: destroy
  };
})();
```

This ridiculous controller is 99% fluff. We could either rewrite it as:

Код этого контроллера нелеп на 99%, мы можем легко переписать его:

```js
var BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy
};
```

...or scrap it altogether as it does nothing other than bundle our Views and Db together.

...или просто выкинуть его полностью, ведь он не делает ничего кроме объединения наших `Views` и `Db`.

## Why favor first class?

## Зачем отдавать предподчтение первому классу?

Okay, let's get down to the reasons to favor first class functions. As we saw in the `getServerStuff` and `BlogController` examples, it's easy to add layers of indirection that have no actual value and only increase the amount of code to maintain and search through.

Хорошо, давайте обсудим причины использования именно функций первого класса. Как мы уже видели в примерах с `getServerStuff` и `BlogController`, добавить бесполезный уровень абстракции легко, но зачем? Это только увеличивает количество кода, который необходимо поддерживать и читать.

In addition, if a function we are needlessly wrapping does change, we must also change our wrapper function.

К тому же, если сигнатура внутренней функции поменяется, нам придётся также менять и внешнюю функцию.

```js
httpGet('/post/2', function(json){
  return renderPost(json);
});
```

If `httpGet` were to change to send a possible `err`, we would need to go back and change the "glue".

Если вдруг `httpGet` станет принимать новый аргумент `err`, то необходимо отредактировать и «функцию-склейку»:

```js
// найти каждый вызов httpGet в приложении и добавить err
httpGet('/post/2', function(json, err){
  return renderPost(json, err);
});
```

Had we written it as a first class function, much less would need to change:

Изменений потребовалось куда меньше, если бы мы с самого начала воспользовались функцией первого класса:

```js
// rednerPost вызывается внутри httpGet с любым количеством аргументов
httpGet('/post/2', renderPost);  
```

Besides the removal of unnecessary functions, we must name and reference arguments. Names are a bit of an issue, you see. We have potential misnomers - especially as the codebase ages and requirements change.

Помимо определения лишних функций, нам также приходится придумывать названия аргументам, что само по себе не всегда тривиально, особенно с ростом приложения.

Having multiple names for the same concept is a common source of confusion in projects. There is also the issue of generic code. For instance, these two functions do exactly the same thing, but one feels infinitely more general and reusable:

Одной из частых проблем в проектах является как раз использование разных имён для одних и тех же понятий. Также, стоит упомянуть момент с обобщением имён. Ниже, обе функции делают одно и тоже, но последняя кажется более общей и, от этого, переиспользуемой:

```js
// специфична для нашего конкретного приложения-блога
var validArticles = function(articles) {
  return articles.filter(function(article){
    return article !== null && article !== undefined;
  });
};

// более общая, легко переиспользовать в другом проекте
var compact = function(xs) {
  return xs.filter(function(x) {
    return x !== null && x !== undefined;
  });
};
```

By naming things, we've seemingly tied ourselves to specific data (in this case `articles`). This happens quite a bit and is a source of much reinvention.

Когда мы даём имена функциям, мы привязываем их к данным (в данным случае к `articles`). Это происходит чаще, чем кажется и является источником изобретения многих «велосипедов». 

I must mention that, just like with Object-Oriented code, you must be aware of `this` coming to bite you in the jugular. If an underlying function uses `this` and we call it first class, we are subject to this leaky abstraction's wrath.

Я должен также умопянуть, что как и при объектно-ориентированном подходе, нужно опасаться того, что `this` подкрадётся и укусит вас за пятку. Если внутренняя функция использует `this`, а мы вызовем её как функцию первого класса, то почувствуем на себе весь гнев утечки абстракции.

```js
var fs = require('fs');

// страшновато
fs.readFile('freaky_friday.txt', Db.save);

// не так страшно
fs.readFile('freaky_friday.txt', Db.save.bind(Db));

```

Having been bound to itself, the `Db` is free to access its prototypical garbage code. I avoid using `this` like a dirty nappy. There's really no need when writing functional code. However, when interfacing with other libraries, you'll have to acquiesce to the mad world around us.

Вызвав `bind` мы даём объекту `Db` возможность использовать мусорный код из его прототипа. Я стараюсь избегать `this` как грязных подгузников, да и в этом нет никакой необходимости, когда пишешь функциональный код. Однако, если вы собираетесь использовать внешние библиотеки, то не забывайте про безумный мир вокруг вас.

Some will argue `this` is necessary for speed. If you are the micro-optimization sort, please close this book. If you cannot get your money back, perhaps you can exchange it for something more fiddly.

Некоторые могут поспорить, утверждая что `this` необходим с точки зрения производительности. Если вы из микро-оптимизаторов, пожалуйста, закройте эту книгу. Если вам не удастся вернуть за неё деньги, я надеюсь вы сможете её обменять на что-нибудь по-сложнее.

And with that, we're ready to move on.

Теперь же, мы готовы продолжать.

[Глава 3: Чистое счастье с Чистыми функциями](ch3-ru.md)
