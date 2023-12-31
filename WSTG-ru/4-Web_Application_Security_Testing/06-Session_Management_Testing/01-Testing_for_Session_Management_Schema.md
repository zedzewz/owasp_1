---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование схемы управления сессиями

|ID          |
|------------|
|WSTG-SESS-01|

## Обзор

Одним из основных компонентов любого web-приложения является механизм, с помощью которого оно контролирует и поддерживает состояние пользователя, взаимодействующего с ним. Чтобы избежать постоянной аутентификации для каждой страницы web-сайта или сервиса, web-приложения реализуют различные механизмы для хранения и проверки учётных данных в течение заранее определённого промежутка времени. Эти механизмы называются управлением сессиями.

В этом сценарии тестировщик должен проверить, что файлы cookie и другие сессионные токены создаются безопасным и непредсказуемым способом. Злоумышленник, который способен предсказать и подделать нестойкие cookie, может легко перехватить сессии законных пользователей.

Для реализации управления сессией используются файлы cookie, которые подробно описаны в [RFC 2965](https://www.rfc-editor.org/rfc/rfc2965). Вкратце: когда пользователь обращается к приложению, которое должно отслеживать действия и идентичность этого пользователя по всем его запросам, сервер генерирует файл cookie (или несколько) и отправляет клиенту. Затем клиент передаёт cookie обратно на сервер во всех последующих подключениях до тех пор, пока его срок действия не истечёт или он не будет уничтожен. Данные, хранящиеся в cookie, могут предоставить серверу широкий спектр информации о том, кем является пользователь, какие действия он выполнял до сих пор, каковы его предпочтения и т.д. Таким образом, предоставляя состояние протоколу без сохранения состояния, такому как HTTP.

Типичным примером является корзина для онлайн-покупок. На протяжении всей сессии пользователя приложение должно отслеживать его идентичность, его профиль, продукты, которые он выбрал для покупки, количество, индивидуальные цены, скидки и т.д. Cookies — эффективный способ хранения и передачи информации (другими способами являются параметры URL и скрытые поля).

Вследствие ценности данных, которые они хранят, cookie имеют жизненно важное значение для общей безопасности приложения. Возможность несанкционированного доступа к cookie может привести к перехвату сессий законных пользователей, получению более высоких привилегий в активной сессии и в целом к нелегитимному влиянию на работу приложения.

В этом сценарии тестировщик должен проверить, могут ли cookie, выдаваемые клиентам, противостоять широкому спектру атак, направленных на вмешательство в сессии законных пользователей и в само приложение. Общая цель состоит в том, чтобы иметь возможность подделать cookie, который приложение будет считать действительным, и который обеспечит возможность несанкционированного доступа (через захват сессии, повышение привилегий, и пр.).

Обычно основными этапами схемы атаки являются:

- **сбор cookie**: сбор достаточного количества образцов cookie;
- **реверс-инжиниринг cookie**: анализ алгоритма формирования cookie;
- **манипулирование cookie**: подделка действующего cookie для проведения атаки. Этот последний шаг может потребовать большого количества попыток, в зависимости от того, как формируется cookie (атака методом перебора).

Другой вариант атаки заключается в переполнении cookie. Строго говоря, эта атака имеет иную природу, поскольку здесь тестировщики не пытаются воссоздать абсолютно корректный cookie. Вместо этого целью является переполнение области памяти, тем самым мешая правильному поведению приложения; иногда даже с инъекцией вредоносного кода (RCE).

## Задачи тестирования

- Собрать сессионные токены для одного пользователя, и по возможности для разных пользователей.
- Проанализировать их, чтобы убедиться, что обеспечивается их достаточная случайность, чтобы предотвратить атаки подмены сессии.
- Модифицировать cookie, которые не подписаны и содержат информацию, которой можно манипулировать.

## Как тестировать

### Тестирование методом чёрного ящика и примеры

Всё взаимодействие между клиентом и приложением должно быть проверено, по крайней мере, на соответствие следующим критериям:

- Все ли директивы `Set-Cookie` помечены как `Secure`?
- Проводятся ли операции с cookie через незашифрованные соединения?
- Можно ли принудительно передать cookie через незашифрованное соединение?
- Если да, то как приложение обеспечивает безопасность?
- Являются ли какие-либо cookie постоянными?
- Какое время указано в `Expires` для постоянных файлов cookie и адекватно ли его значение?
- Указаны ли cookie, которые, как ожидается, будут временными, как таковые?
- Какие настройки HTTP/1.1 `Cache-Control` применяются для защиты cookie?
- Какие настройки HTTP/1.0 `Cache-Control` применяются для защиты cookie?

#### Сбор Cookie

Первым шагом, необходимым для манипулирования cookie, является понимание того, как приложение создаёт файлы cookie и управляет ими. Для выполнения этой задачи тестировщики должны попытаться ответить на следующие вопросы:

- Сколько файлов cookie используется в приложении?

  Побродите по приложению. Обратите внимание, когда создаются cookie. Составьте список полученных cookie, страницу, которая их устанавливает (с помощью директивы set-cookie), домен, для которого они действительны, их значения и характеристики.

- Какие части приложения генерируют или изменяют cookie?

  Просматривая приложение, определите, какие cookie остаются постоянными, а какие изменяются. Какие события изменяют cookie?

- Каким частям приложения требуется cookie для доступа и использования?

  Узнайте, какие части приложения нуждаются в cookie. Зайдите на страницу, затем повторите попытку без cookie или с его изменённым значением. Попробуйте сопоставить, какие cookie где используются.

Ценным результатом этого этапа может стать таблица, сопоставляющая каждый файл cookie с соответствующими частями приложения и связанной информацией.

#### Анализ сессии

Сами сессионные токены (Cookie, SessionID или Hidden Field) должны быть проверены, чтобы убедиться в их качестве с точки зрения безопасности. Их следует проверять по таким критериям, как случайность, уникальность, устойчивость к статистическому и криптографическому анализу и к утечке информации.

- Структура токена и утечка информации

Первый этап заключается в изучении структуры и содержимого идентификатора сессии (англ.: Session ID), предоставляемого приложением. Распространённой ошибкой является включение конкретных данных в токен вместо обобщённых, и ссылки на реальные данные на стороне сервера.

Если Session ID передаётся открытым текстом, структура и соответствующие данные могут быть сразу видны, например: `192.168.100.1:owaspuser:password:15:58`.

Если часть или весь токен, по-видимому, закодирован или хэширован, его следует сравнить с различными представлениями, чтобы проверить наличие очевидной обфускации. Например, вот как строка `192.168.100.1:owaspuser:password:15:58` выглядит в Hex, Base64 и в виде хэша MD5:

- Hex: `3139322E3136382E3130302E313A6F77617370757365723A70617373776F72643A31353A3538`
- Base64: `MTkyLjE2OC4xMDAuMTpvd2FzcHVzZXI6cGFzc3dvcmQ6MTU6NTg=`
- MD5: `01c2fc4f0a817afd8366689bd29dd40a`

Определив тип обфускации, возможно, удастся декодировать исходные данные. Однако в большинстве случаев это маловероятно. Тем не менее, может оказаться полезным попробовать разные кодировки для сообщения. Кроме того, если можно определить как формат, так и метод обфускации, можно было бы разработать автоматизированные атаки методом перебора.

Гибридные токены могут включать в себя такую информацию как IP-адрес или идентификатор пользователя, вместе с закодированной частью, например, `owaspuser:192.168.100.1:a7656fafe94dae72b1e1487670148412`.

Проанализировав один сессионный токен, следует изучить репрезентативную выборку. Простой анализ токенов должен сразу выявить очевидные закономерности. Например, 32-битный токен может включать в себя 16 бит статичных и 16 бит переменных данных. Это может указывать на то, что первые 16 бит представляют фиксированный атрибут пользователя, например, его имя или IP-адрес. Если второй 16-битный фрагмент монотонно увеличивается, это может указывать на последовательный или даже временной элемент для генерации токена.

Если идентифицированы статичные элементы токенов, следует собрать дополнительные образцы, изменяя по одному потенциальному входному элементу за раз. Например, попытки входа в систему через другую учётную запись или с другого IP-адреса могут привести к изменениям в ранее статичной части сессионного токена.

При тестировании структуры Session ID следует рассмотреть следующие вопросы:

- Какие части Session ID являются статичными?
- Какая конфиденциальная информация из Session ID хранится в открытом виде? Например, имена пользователей/UID, IP-адреса.
- Какая конфиденциальная информация легко декодируется?
- Какую информацию можно извлечь из структуры Session ID?
- Какие части Session ID являются статичными для одних и тех же условий входа в систему?
- Какие очевидные закономерности просматриваются в структуре Session ID в целом или в отдельных его частях?

#### Предсказуемость и случайность Session ID

Следует провести анализ переменных областей в структуре Session ID (если таковые имеются), чтобы установить наличие распознаваемых или предсказуемых закономерностей. Такой анализ можно провести вручную или с помощью разработанных или приобретённых статистических или криптоаналитических инструментов для выявления паттернов в содержимом Session ID. Ручные проверки должны включать сравнение Session ID, выданных для одних и тех же условий входа в систему, например, для одного и того же имени пользователя, пароля и IP-адреса.

Время — важный фактор, который приходится также учитывать. Необходимо обеспечить большое количество одновременных подключений, чтобы собирать выборки в одном и том же временном окне и постоянно поддерживать это количество. Дискретизация даже в 50 мс или менее может оказаться слишком грубой, и выборка, полученная таким образом, может выявить элементы, зависящие от времени, которые в противном случае были бы упущены.

Переменные элементы следует анализировать, чтобы определить, монотонно ли они изменяются с течением времени. Если это так, стоит исследовать закономерности, относящиеся к абсолютному или относительному времени. Многие системы используют время в качестве начального значения для своих псевдослучайных элементов. Там, где паттерны кажутся случайными, следует рассмотреть возможность хэширования времени или других изменений окружающей среды. Как правило, результатом криптографического хэша является десятичное или шестнадцатеричное число, поэтому его и следует искать.

При анализе последовательностей, закономерностей или циклов в Session ID, статичные элементы и зависимости клиента следует рассматривать как потенциально вносящие вклад в структуру и функции приложения.

- Являются ли Session ID доказуемо случайными по своей природе? Можно ли воспроизвести результирующие значения?
- Дают ли одни и те же входные условия один и тот же Session ID при последующем запуске?
- Являются ли Session ID доказуемо устойчивыми к статистическому или криптоанализу?
- Какие элементы Session ID зависят от времени?
- Какие части Session ID являются предсказуемыми?
- Можно ли определить следующий Session ID, зная алгоритм его формирования и предыдущие Session ID?

#### Реверс-инжиниринг Cookie

Теперь, когда тестировщик собрал cookie и имеет общее представление об их использовании, пришло время рассмотреть поподробнее те из них, которые кажутся интересными. Какие cookie нас интересуют? Для безопасного управления сессией cookie должны сочетать в себе несколько характеристик, каждая из которых направлена на защиту от различных видов угроз.

Вот эти характеристики:

1. **Непредсказуемость**: cookie должен содержать некоторое количество данных, которые трудно угадать. Чем сложнее подделать настоящий cookie, тем сложнее взломать сессию законного пользователя. Если злоумышленник сможет угадать cookie, используемый в активной сессии законного пользователя, он сможет полностью выдать себя за этого пользователя (захват сессии). Чтобы сделать cookie непредсказуемым, можно использовать случайные значения или криптографию.
2. **Устойчивость к несанкционированному доступу**: файл cookie должен противостоять злонамеренным попыткам модификации. Если тестировщик находит cookie типа `isAdmin=No`, его легко изменить, чтобы получить права администратора, если только приложение не выполняет двойную проверку (например, добавляя к cookie зашифрованный хэш его значения).
3. **Срок действия**: чувствительные cookie должны действовать в течение минимально необходимого периода времени и сразу удаляться с диска или из памяти, чтобы избежать риска их повторного воспроизведения. Это не относится к cookie, в которых хранятся некритичные данные, которые необходимо запоминать между сессиями (например, внешний вид сайта).
4. **Флаг** `Secure`: cookie, значения которых критичны для целостности сессии, должны иметь этот флаг, чтобы разрешить его передачу только по зашифрованному каналу для предотвращения перехвата.

Подход здесь заключается в том, чтобы собрать достаточную выборку из cookie и начать искать закономерности в их значениях. Точное определение «достаточности» выборки может варьироваться от нескольких экземпляров, если алгоритм формирования cookie легко взломать, до нескольких тысяч, если необходимо провести некоторый математический анализ (например, хи-квадрат, странные аттракторы — см. раздел Ссылки).

Важно обращать внимание на этапы бизнес-процесса в приложения, т.к. состояние сессии может сильно повлиять на cookie. Файл cookie, полученный до аутентификации, может сильно отличаться от такового, полученного после неё.

Ещё один аспект, который следует учитывать, — это время. Всегда записывайте точное время получения cookie, если существует вероятность того, что время влияет на значение cookie (сервер может использовать метку времени в составе значения cookie). Записанное время может быть местным временем или отметкой времени сервера, включённой в HTTP-ответ (или и то, и другое).

При анализе собранных значений необходимо попытаться выяснить все переменные, которые могли повлиять на значение cookie, и попробовать изменять их по одной за раз. Передача на сервер изменённых версий одного и того же cookie может быть очень полезной для понимания того, как приложение считывает и обрабатывает cookie.

Примеры проверок, которые должны быть выполнены на этом этапе, включают:

- Какой набор символов используется в cookie? Cookie имеет числовое значение? буквенно-цифровое? шестнадцатеричное? Что произойдёт, если вставить в cookie символы, не принадлежащие к ожидаемой кодировке?
- Состоят ли cookie из фрагментов, несущих разную информацию? Как части отделяются друг от друга? Какими разделителями? Некоторые элементы cookie могут иметь более высокую дисперсию, другие могут быть постоянными, третьи могут принимать ограниченный набор значений. Разбивка cookie на составные части — это первый и основной этап.

Пример легко определяемой структуры cookie:

```text
ID=5a0acfc7ffeb919:CR=1:TM=1120514521:LM=1120514521:S=j3am5KzC4v01ba3q
```

В данном примере 5 различных полей, содержащих разные типы данных:

- ID – шестнадцатеричный
- CR – малое целое число
- TM и LM – большое целое (Интересно, что они имеют одинаковые значения. Стоит посмотреть, что произойдёт, если изменить одно из них)
- S – буквенно-цифровой.

Даже если разделители не используются, наличие достаточного количества примеров может помочь понять структуру.

#### Атаки методом перебора

Атаки методом перебора неизбежно приводят к вопросам, связанным с предсказуемостью и случайностью. Дисперсия в Session ID должна учитываться наряду с продолжительностью сессии и тайм-аутами. Если дисперсия в Session ID относительно невелика, а срок действия – длительный, вероятность успешной атаки методом перебора намного выше.

Длинный Session ID (или, скорее, Session ID с большой дисперсией) и более короткий срок действия значительно усложнили бы успех атаки методом перебора.

- Сколько времени займет атака методом перебора на все возможные Session ID?
- Достаточно ли велико пространство Session ID, чтобы предотвратить их перебор? Например, достаточна ли длина ключа по сравнению с допустимым сроком действия?
- Снижают ли задержки между попытками подключения с разными Session ID риск такой атаки?

### Тестирование методом серого ящика и примеры

Если есть доступ к реализации схемы управления сессиями, можно проверить следующее:

- Случайный сессионный токен

  Session ID или cookie, выданный клиенту, не должны быть легко предсказуемыми (не используйте линейные алгоритмы, основанные на предсказуемых переменных, таких как IP-адрес клиента). Приветствуется использование криптографических алгоритмов с длиной ключа 256 бит и более (например, AES).

- Длина токена

  Session ID должен содержать не менее 50 символов.

- Тайм-аут сессии

  Сессионный токен должен иметь определённый тайм-аут (в зависимости от чувствительности управляемых приложением данных)

- Флаги Cookie:
    - непостоянные: хранятся только в оперативной памяти (Expires и Max-Age не заполняются)
    - secure (устанавливаются только через HTTPS): `Set-Cookie: cookie=data; path=/; domain=.aaa.it; secure`
    - [HTTPOnly](https://owasp.org/www-community/HttpOnly) (недоступны для JavaScript): `Set-Cookie: cookie=data; path=/; domain=.aaa.it; HttpOnly`

Более подробная информация в разделе [Тестирование атрибутов Cookie](02-Testing_for_Cookies_Attributes.md).

## Инструменты

- [OWASP Zed Attack Proxy Project (ZAP)](https://www.zaproxy.org) - есть механизм анализа сессионных токенов.
- [Burp Sequencer](https://portswigger.net/burp/documentation/desktop/tools/sequencer)
- [YEHG's JHijack](https://github.com/yehgdotnet/JHijack)

## Ссылки

### Технические руководства

- [RFC 2965 "HTTP State Management Mechanism"](https://tools.ietf.org/html/rfc2965)
- [RFC 1750 "Randomness Recommendations for Security"](https://www.ietf.org/rfc/rfc1750.txt)
- [Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis" (2001)](http://lcamtuf.coredump.cx/oldtcp/tcpseq.html)
- [Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis - One Year Later" (2002)](http://lcamtuf.coredump.cx/newtcp/)
- [Correlation Coefficient](http://mathworld.wolfram.com/CorrelationCoefficient.html)
- [ENT](https://fourmilab.ch/random/)
- [DMA[2005-0614a] - 'Global Hauri ViRobot Server cookie overflow'](https://seclists.org/lists/fulldisclosure/2005/Jun/0188.html)
- [Gunter Ollmann: "Web Based Session Management"](http://www.technicalinfo.net/papers/WebBasedSessionManagement.html)
- [OWASP Code Review Guide](https://wiki.owasp.org/index.php/Category:OWASP_Code_Review_Project)
