---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование перенаправления URL на стороне клиента

|ID          |
|------------|
|WSTG-CLNT-04|

## Обзор

В этом разделе описывается, как проверить перенаправление URL на стороне клиента, также известное как открытое перенаправление. Это недостаток контроля входных данных, который возникает, когда приложение принимает от пользователя ссылку, ведущую на внешний URL, который может быть вредоносным. Такого рода уязвимость может эксплуатироваться при проведении фишинговой атаки или перенаправления жертвы на заражённую страницу.

Эта уязвимость возникает, когда приложение принимает недоверенные входные данные, содержащие значение URL, и не нейтрализует их. Это значение URL может привести к перенаправлению пользователя на другую страницу, например, контролируемую злоумышленником.

Эта уязвимость может позволить злоумышленнику успешно запустить фишинговую атаку и украсть учётные данные пользователя. Поскольку перенаправление инициируется самим приложением, попытки фишинга могут выглядеть более достоверно.

Вот пример URL для фишинговой атаки.

```text
http://www.target.site?#redirect=www.fake-target.site
```

Жертва, которая перейдёт по этому URL, будет автоматически перенаправлена ​​на `fake-target.site`, где злоумышленник может разместить поддельную страницу, напоминающую реальный сайт, чтобы украсть учётные данные жертвы.

Открытое перенаправление также может быть использовано для составления URL, который позволит обойти проверки контроля доступа приложения и перенаправить злоумышленника к привилегированным функциям, к которым он доступа не имеет.

## Задачи тестирования

- Найти точки инъекции, которые обрабатывают URL или пути.
- Подумать, куда система могла бы отсюда перенаправить.

## Как тестировать

Когда тестировщики проверяют этот тип уязвимости вручную, они сначала определяют, есть ли перенаправления, реализованные в коде на стороне клиента. Эти перенаправления могут быть реализованы с помощью объекта `window.location`. Его можно использовать для направления браузера на другую страницу, просто присвоив её URL строке. Это иллюстрируется в следующем фрагменте кода на JavaScript:

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location='http://'+decodeURIComponent(redir);
}
```

В этом примере скрипт никак не проверяет переменную `redir`, которая содержит строку запроса, введённую пользователем. Поскольку никакая форма кодирования не применяется, этот недоверенный ввод передается объекту `windows.location`, создавая уязвимость перенаправления URL.

Это означает, что злоумышленник может перенаправить жертву на вредоносный сайт, просто отправив следующую строку запроса:

```text
http://www.victim.site/?#www.malicious.site
```

С небольшой модификацией приведённый выше фрагмент кода может быть уязвим для инъекции JavaScript:

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location=decodeURIComponent(redir);
}
```

Его можно проэксплуатировать, в следующей строке запроса:

```text
http://www.victim.site/?#javascript:alert(document.cookie)
```

При тестировании на наличие этой уязвимости учитывайте, что некоторые символы обрабатываются разными браузерами по-разному. Для справки см. [XSS на основе DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS).
