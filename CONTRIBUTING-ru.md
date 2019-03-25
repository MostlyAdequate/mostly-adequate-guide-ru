
# Contributing to Mostly Adequate Guide to Functional Programming

## Licensing

By opening a pull request to this repository, you agree to provide your work under the [project license](LICENSE). Also, you agree to grant such license of your work as is required for the purposes of future print editions to @DrBoolean. Should your changes appear in a printed edition, you'll be included in the contributors list.

## Small Corrections

Errata and basic clarifications will be accepted if we agree that they improve the content. You can also open an issue so we can figure out how or if it needs to be addressed. If you've never done this before, the [flow guide](https://guides.github.com/introduction/flow/) might be useful.

В русском переводе мы придерживаемся менее строгих правил: мы не ждём принятия PR с исправлением очевидных ошибок и неточностей, а также дополняем текст своими комментариями всюду, где сочтём уместным. Такое отношение ускоряет работу и повышает качество итогового продукта. Однако, во избежание беспорядка, мы следуем своим собственным правилам:

1. Текст и код оригиналов изменять не следует. Они позволяют проследить за изменениями спустя время.
1. Любое исправление должно быть отражено в каком-то PR в основной (английский) репозиторий.
1. Исправленные фрагменты следует сопровождать комментарием со ссылкой на PR в основной репозиторий, например: `<!-- https://github.com/MostlyAdequate/mostly-adequate-guide/pull/508 -->`, что послужит обоснованием для исправлений и позволит легко проследить за их состоянием. Ссылки на принятые PR следует удалять, а оригиналы - обновлять.

## Questions or Clarifications

Please, have a look at the [FAQ](FAQ-ru.md) before you open an issue. Your question may already
have been answered. Should you still need to ask something? Feel free to open an issue and to
explain yourself.

## Translations

Translations to other languages are highly encouraged. Each official translation will be held as a separate repository in the [MostlyAdequate organization](https://github.com/MostlyAdequate) and linked from the English version book.
Since each translation is a different repository, we can also have different maintainers for each project.

### Creating a New Translation Repo

In order to create a new translation, you need to follow these steps:

* Fork the [main repo](https://github.com/MostlyAdequate/mostly-adequate-guide).
* Add yourself to the watch list of the main repo, to keep up with changes.
* When translating chapters, **create NEW files with suffix of your language**.
  * For example, Spanish tranlation for `ch01.md` will be on `ch01-es.md`.
* Open a [new issue](https://github.com/MostlyAdequate/mostly-adequate-guide/issues/new) and ask to be part of the organization.
* Transfer the repo to the organization.
* Merge new content from the main repo.
* keep translating...
* Rinse/repeat last two steps until the book is done.
