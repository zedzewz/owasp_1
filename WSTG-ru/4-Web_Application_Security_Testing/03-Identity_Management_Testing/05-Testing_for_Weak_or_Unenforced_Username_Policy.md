---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование политики в отношении имён пользователей

|ID          |
|------------|
|WSTG-IDNT-05|

## Обзор

Имена учётных записей пользователя часто чётко структурированы (например, имя учётной записи Joe Bloggs — это jbloggs, а учётной записи Fred Nurks — это fnurks), поэтому действительные имена учётных записей легко угадать.

## Задачи тестирования

- Определить, делает ли следования структуре имени учётной записи приложение уязвимым для перебора учётных записей.
- Определить, разрешён ли перебор учётных записей через сообщения об ошибках.

## Как тестировать

- Определите структуру имён учётных записей.
- Оцените реакцию приложения на действительные и недействительные имена учётных записей.
- Используйте разницу в ответах на действительные и недействительные имена учётных записей для перебора допустимых имён учетных записей.
- Используйте словари имён учётных записей для перебора допустимых имён.

## Как исправить

Убедитесь, что приложение в ответ на неверное имя учётной записи, пароль или другие учётные данные пользователя, введённые в процессе входа в систему, выдаёт непротиворечивые, но неконкретные сообщения об ошибках.