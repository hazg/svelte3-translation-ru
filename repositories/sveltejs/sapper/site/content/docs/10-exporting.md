---
title: Экспортирование
---

Есть немалая часть сайтов, которые по сути своей *статичны*, то есть им для работы не нужен сервер Express. Вместо этого они могут распространяться в виде статических файлов, что позволяет развёртывать их практически на любом хостинге (вроде [Netlify](https://www.netlify.com/) или [GitHub Pages](https://pages.github.com/)). Статические сайты, как правило, дешевле в эксплуатации и имеют непрeвзойдённую производительность.

Sapper позволяет вам *экспортировать* сайт в статические файлы с помощью одной простой команды `sapper export`. Кстати, вы прямо сейчас смотрите на экспортированный сайт!

При этом термин *статический* не означает, что приложение перестанет быть *интерактивным* — ваши компоненты Svelte будут работают точно так же, как и обычно, и такие вещи, как маршрутизация на клиенте и упреждающая загрузка, тоже никуда не денутся.


### sapper export

В директории вашего проекта Sapper выполните следующую команду:

```bash
# npx позволяет использовать локально установленные зависимости
npx sapper export
```

Будет создана директория `__sapper__/export` с готовой сборкой вашего сайта. Вы можете сразу запустить его таким образом:

```bash
npx serve __sapper__/export
```

Перейдите в браузере на адрес [localhost:5000](http://localhost:5000) и убедитесь, что ваш сайт работает корректно.

Вы также можете добавить скрипт в свой файл `package.json`...

```js
{
	"scripts": {
		...
		"export": "sapper export"
	}
}
```

...что позволит экспортировать ваше приложение командой `npm run export`.


### Как это работает

Когда вы запускаете `sapper export`, Sapper сначала создаёт рабочую версию вашего приложения, как происходит при запуске `sapper build`, и копирует содержимое вашей папки `static` в место назначения. Затем он запускает сервер и 'заходит' на главную страницу получившегося сайта. Оттуда он следует по всем найденным локальным ссылкам из элементов `<a>`, `<img>`, `<link>` и `<source>` и сохраняет любые данные, предоставляемые приложением.

По этой причине любые страницы, которые необходимы в экспортированом сайте, должны быть доступны с помощью элементов `<a>`или добавлены при помощи ключа `--entry` для команды `sapper export`. Кроме того, любые серверные или иные нестраничные маршруты должны запрашиваться в `preload`, а *не* в `onMount` или ещё где-либо.

Опция `--entry` ожидает получить строку значений разделенных пробелом. Например:

```bash
sapper export --entry "some-page some-other-page"
```

Установка опции `--entry` перезаписывает любые значения по умолчанию. Если вы хотите добавить точки входа _в дополнение_ к `/` тогда убедитесь, что вы указали `/` среди передаваемых значений опции, иначе `sapper export` не зайдет в корень сайта.


### Когда экспортировать не нужно

Основное правило таково: чтобы приложение могло быть экспортировано, любые два пользователя, попадающие на одну и ту же страницу вашего приложения, должны получать одинаковое содержимое с сервера. Другими словами, любое приложение, которое включает в себя пользовательские сессии или аутентификацию, *не* может быть правильно экспортировано командой `sapper export`.

Обратите внимание, что вы всё ещё можете экспортировать приложения с динамическими маршрутами, как в нашем примере `src/routes/blog/[slug].html`. Команда `sapper export` будет обрабатывать `this.fetch` запросы внутри функций `preload`, поэтому данные, поступающие из `src/routes/blog/[slug].json.js` тоже будут сохранены.


### Конфликты маршрутов

Поскольку `sapper export` создаёт отражение всех маршрутов в виде файлового дерева, то невозможно иметь два *серверных маршрута*, где возникает ситуация, что директория и файл в одном и том же месте будут иметь одинаковое имя. Например, `src/routes/foo/index.js` и `src/routes/foo/bar.js` будут пытаться создать файлы `export/foo` и `export/foo/bar`, что приведёт к ошибке.

Решение состоит в том, чтобы переименовать один из маршрутов и избежать подобного конфликта — например так: `src/routes/foo-bar.js`. Не забудьте, что при этом придётся подправить код приложения в том месте, где он берёт данные `/foo/bar`, указав новый маршрут `/foo-bar`.

Для *маршрутов страниц* этой проблемы не возникает, поскольку мы создаём файл `export/foo/index.html` вместо `export/foo`.
