[![cover](images/cover.png)](SUMMARY-ru.md)

## Об этой книге

Эта книга рассказывает о парадигме функционального программирования в общем. Мы будем использовать самый популярный в мире функциональный язык программирования: JavaScript. Некоторые из вас могут сказать, что это не самый удачный выбор, потому что в данный момент в JavaScript преобладают императивные тенденции. Однако, я считаю, что JavaScript — это лучший способ знакомства с функциональным программированием по нескольким причинам:

 * **Вы, скорее всего, и так каждый день используете его в своей работе.**

    Это открывает возможность воспользоваться полученным знанием в реальных прикладных программах, в отличие от тех проектов, которые вы пишите в качестве хобби по выходным на каком-нибудь экзотическом языке.

 * **Вам не потребуется учить язык с нуля для того, чтобы начать писать программы.**

    В чисто функциональном языке у вас не получится залогировать значение переменной или получить элемент DOM без использования монад. Пока мы не освоили все приёмы функционального программирования, благодаря смешанной парадигме JS, можно немного сжульничать и воспользоваться приёмами из ООП.

 * **JS отлично подходит для написания первоклассного функционального кода.**

    В JS у нас есть всё что нужно для имитации Scala или Haskell с помощью парочки небольших библиотек. В данный момент ООП доминирует в индустрии, но он очень неудобен в Javascript’е, примерно также, как пойти с палатками на трассу или танцевать чечётку в сапогах. Чтобы случайно не потерять контекст `this`, мы повсеместно используем `bind`. Забыли написать `new`? Будьте готовы к причудливым ошибкам. А приватные поля доступны только через замыкания. Многие из нас считают функциональное программирование более подходящим вариантом для JS.

Учитывая всё вышесказанное, бесспорно, лучше всего для примеров из этой книги подойдут типизированные функциональные языки. JavaScript поможет нам познакомиться с подходом к программированию функционально, какой язык использовать для его применения — решать вам. К счастью, все интерфейсы математические и, посему, универсальные. Вы будете комфортно себя чувствовать, используя Swiftz, Haskell, PureScript и другие математически-ориентированные языки.

## Читать онлайн (на английском)

For a best reading experience, [read it online via Gitbook](https://mostly-adequate.gitbooks.io/mostly-adequate-guide/).

- Quick-access side-bar
- In-browser exercises
- In-depth examples

## Поиграть с кодом

To make the training efficient and not get too bored while I am telling you another story, make sure to play around with the concepts introduced in this book. Some can be tricky to catch at first and are better understood by getting your hand dirty. 
All functions and algebraic data-structures presented in the book are gathered in the appendixes. The corresponding code is also available as an npm module:

```bash
$ npm i @mostly-adequate/support
```

Alternatively, exercises of each chapter are runnable and can be completed in your editor! For example, complete the `exercise_*.js` in `exercises/ch04` and then run:

```bash
$ npm run ch04
```

## Скачать

* [Скачать PDF](https://www.gitbook.com/download/pdf/book/mostly-adequate/mostly-adequate-guide) (на английском)
* [Скачать EPUB](https://www.gitbook.com/download/epub/book/mostly-adequate/mostly-adequate-guide) (на английском)
* [Скачать Mobi (Kindle)](https://www.gitbook.com/download/mobi/book/mostly-adequate/mostly-adequate-guide) (на английском)

## Собрать книгу самостоятельно

```
git clone https://github.com/MostlyAdequate/mostly-adequate-guide-ru.git
cd mostly-adequate-guide-ru/
npm install
npm run setup
npm run generate-pdf
npm run generate-epub
```

> Note! To generate the ebook version you will need to install `ebook-convert`. [Installation instructions](https://toolchain.gitbook.com/ebook.html#installing-ebook-convert).

# Оглавление

[SUMMARY-ru.md](SUMMARY-ru.md)

### Contributing

[CONTRIBUTING-ru.md](CONTRIBUTING-ru.md)

### Переводы

[TRANSLATIONS-ru.md](TRANSLATIONS-ru.md)

### FAQ

[FAQ-ru.md](FAQ-ru.md)

# Plans for the future

* **Part 1** (chapters 1-7) is a guide to the basics. I'm updating as I find errors since this is the initial draft. Feel free to help!
* **Part 2** (chapters 8-13) address type classes like functors and monads all the way through to traversable. I hope to squeeze in transformers and a pure application.
* **Part 3** (chapters 14+) will start to dance the fine line between practical programming and academic absurdity. We'll look at comonads, f-algebras, free monads, yoneda, and other categorical constructs.

---

<p align="center">
  <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
    <img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" />
  </a>
  <br />
  This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
</p>
