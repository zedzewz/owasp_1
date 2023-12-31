---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование способности подделывать запросы

|ID          |
|------------|
|WSTG-BUSL-02|

## Обзор

Подделка запросов — это метод, который злоумышленники используют, чтобы обойти ограничения графического интерфейса клиентской части приложения и напрямую отправить информацию для её обработки сервером. Цель злоумышленника — передать HTTP-запросы POST/GET через перехватывающий прокси со значениями данных, которые не поддерживаются, от которых не предусмотрена защита или они не ожидаются бизнес-логикой приложения. Некоторые примеры подделки запросов включают в себя использование предполагаемых или предсказуемых параметров или раскрытие «скрытых» функций, и, например, включение отладки или представление специальных экранов или окон, которые полезны во время разработки, но могут привести к утечке информации или обходу бизнес-логики.

Уязвимости, связанные с возможностью подделки запросов, индивидуальны для каждого приложения и отличаются от форматно-логического контроля данных тем, что они направлены на нарушение логики бизнес-процесса.

Приложения должны контролировать соблюдение логики, чтобы система не принимала поддельные запросы, которые могут дать злоумышленникам возможность эксплуатировать уязвимости бизнес-логики приложения или бизнес-процесса. Подделка запроса не является чем-то новым; злоумышленники используют перехватывающие прокси для передачи HTTP POST/GET-запросов приложению. С помощью подделки запросов злоумышленники могут обойти бизнес-логику или процесс, находя, прогнозируя и манипулируя параметрами, чтобы заставить приложение думать, что процесс или задача выполнены или не выполнены.

Кроме того, поддельные запросы могут допускать ветвление программы или потока бизнес-логики, вызывая «скрытые» функции, исходно используемые разработчиками и тестировщиками, также иногда называемые [«пасхалками»](https://ru.wikipedia.org/wiki/%D0%9F%D0%B0%D1%81%D1%85%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5_%D1%8F%D0%B9%D1%86%D0%BE_(%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5)). «Пасхалка» — это умышленная внутренняя шутка, скрытое послание или особенность компьютерных программ, фильмов, книг, кроссвордов и пр. По словам игрового дизайнера Уоррена Робинетта, этот термин был придуман в Atari персоналом, который был предупреждён о наличии секретного сообщения, которое было скрыто Робинеттом в его уже широко распространённой игре Adventure. Говорят, что это название отсылает к идее традиционной семейной охоты за пасхальными яйцами, спрятанными на местности».

### Пример 1

Предположим, сайт онлайн-кассы театра позволяет пользователям выбрать билет, применить единовременную скидку 10% для пенсионеров на весь репертуар, просмотреть промежуточный итог и оплатить покупку. Если злоумышленник может через прокси увидеть, что в приложении есть скрытое поле (со значением 1 или 0), используемое бизнес-логикой для определения того, была получена скидка или нет, то он может передать значение 1, т.е. «скидка не применялась» несколько раз, чтобы несколько раз воспользоваться одной и той же скидкой.

### Пример 2

Предположим, что онлайн-видеоигра выплачивает жетоны за очки, набранные за поиск пиратских сокровищ и пиратов, а также за каждый пройденный уровень. Эти жетоны впоследствии могут быть обменены на призы. Кроме того, очки каждого уровня имеют значение множителя, равное этому уровню. Если злоумышленник через прокси увидел, что в приложении есть скрытое поле, используемое при разработке и тестировании, чтобы быстро добраться до самых высоких уровней игры, он смог бы быстро добраться до самых высоких уровней и быстро накопить незаработанные очки.

Кроме того, если злоумышленник смог увидеть через прокси, что в приложении есть скрытое поле, используемое во время разработки и тестирования для включения журнала, в котором указывалось, где находятся другие онлайн-игроки или спрятанные сокровища по отношению к злоумышленнику, тогда они смогут быстро перейти к этим местоположениям и набрать очки.

## Задачи тестирования

- Проанализировать проектную документацию в поисках угадываемой, предсказуемой или скрытой функциональности полей.
- Вставить логически допустимые данные, чтобы обойти стандартную логику бизнес-процесса.

## Как тестировать

### Через угадывание предполагаемых значений

- Используя перехватывающий прокси, понаблюдайте за POST/GET HTTP-запросами в поисках какого-либо признака того, что значения увеличиваются с регулярным интервалом или их легко угадать.
- Если обнаружится, что какое-то значение поддаётся угадыванию, то изменив это значение, можно получить неожиданную выгоду.

### Через выявление скрытых опций

- Используя перехватывающий прокси, наблюдайте за HTTP POST/GET в поисках каких-либо указаний на скрытые функции, такие как debug, которые можно включить или активировать.
- Если таковые будут найдены, попробуйте угадать и изменить эти значения, чтобы получить другой ответ или поведение приложения.

## Связанные сценарии тестирования

- [Тестирование незащищённых параметров сессии](../06-Session_Management_Testing/04-Testing_for_Exposed_Session_Variables.md)
- [Тестирование подделки межсайтовых запросов (CSRF)](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md)
- [Перебор и угадывание учётных записей пользователей](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)

## Меры защиты

Приложение должно быть достаточно интеллектуальным и включать бизнес-логику, которая не позволит злоумышленникам прогнозировать и манипулировать параметрами для нарушения логики приложения или бизнес-процесса или использовать скрытые/недокументированные функции, такие как отладка.

## Инструменты

- [OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Ссылки

- [Cross Site Request Forgery - Legitimizing Forged Requests](http://www.stan.gr/2012/11/cross-site-request-forgery-legitimazing.html)
- [Пасхальное яйцо](https://ru.wikipedia.org/wiki/%D0%9F%D0%B0%D1%81%D1%85%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5_%D1%8F%D0%B9%D1%86%D0%BE_(%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5))
- [Long Live Software Easter Eggs!](https://queue.acm.org/detail.cfm?id=3534857)
