# 🛴 Автоматизация коллекции Postman `API бэкенд-приложения` с помощью скриптов

#### 👾 Тестирование внутренней интеграции `бэкенд-приложения`, разработанного [Валерием Меньшиковым](https://aqa-engineer.com/about).
***
### Архитектура бэкенд-приложения включает:
- Register-сервис — Отвечает за регистрацию новых пользователей и их активацию;
- Auth-сервис — Позволяет авторизоваться в системе, используя логин и пароль, и получить токен для доступа к другим сервисам
- Mail-сервис — Обрабатывает письма, отправленные пользователю;
- Users-сервис — Получение доступа к информации о пользователях;
- Account API — Предоставляет функции для изменения данных учетной записи, таких как имя, email или пароль;
- gRPC, GraphQL и др.

Все построено на языках C# и Python.

---

- Автотесты спроектированы в `Postman-Scripts` на языке ***JavaScript***
- Работа с запросами ***Postman for Windows Version 11.56.1***
- Работа с удалёнными базами данных `PostgreSQL` и `ClickHouse` ***DBeaver 24.3.3***
- Работа с кэшируемыми данными ***Redis Insight 2.68.0***

## Организация коллекции Postman
- Коллекция разбита по микросевисам: Register; Mail; Auth; Account; Users; Forum.
- Каждый раздел-сервис включает в себя позитивные и негативные проверки API.
- Негативные проверки разбиты по статус-кодам ответов сервера - 400, 401, 403 и т.д.
---
```sh
- Positive tests
- Negative tests
  - 400
  - 404
  - 405
  - 415
  - 422
```

## Содержание скриптов

Для коллекции Postman создано отдельное окружение(Environment) с объявлянными переменными. Значение переменных задаётся через парсинг тела ответа для извлечения данных из ответа от сервера в формате JSON и преобразования их в объект JavaScript.

---
```javascript
// Получение Auth токена в переменную окружения
console.log(pm.response.json())

var token = pm.response.json().metadata.token 
console.log(token)

pm.environment.set("env_token", token);


// Сравнение значения из тела ответа от сервера со значением из тела запроса
pm.test("Test", function () {
    // Получаем тело запроса
  var requestBody = pm.request.body.raw;
    // Парсим его в объект
    var parsedRequestBody = JSON.parse(requestBody);    
    // Получаем тело ответа
  var responseBody = pm.response.json();
// Сравниваем значение resource.login из запроса и ответа c login из тела запроса
  pm.expect(responseBody.resource.login).to.eql(parsedRequestBody.login);

  // Логируем значение из тела запроса
    console.log(parsedRequestBody.login);
   // Логируем значение из тела ответа от сервера
    console.log(responseBody.resource.login );
});
```

```javascript
// Получаем тело ответа в формате JSON
let responseBody = pm.response.json();

// Указываем нужный email, для которого ищем письмо
let targetEmail = "astarion@baldura12.info";

// Объявляем переменную для ID письма
let message_id = null;

// Перебираем все письма в массиве items
responseBody.items.forEach(item => {

    item.To.forEach(recipient => {     // Перебор получателей каждого письма
        let email = recipient.Mailbox + "@" + recipient.Domain;    // Получаем полный email из ключей Mailbox и Domain
        if (email === targetEmail) {  // Если искомый email совпадает с нужным, сохраняем ID письма в message_id
            message_id = item.ID;
        }
    });
});

// С помощью консоли, проверяем, найдено ли письмо
if (message_id) {  
    pm.environment.set("message_id", message_id);  // Сохраняем message_id письма в переменную окружения 
    console.log("Письмо найдено. ID:", message_id);
} else {
    console.log("Письмо для " + targetEmail + " не найдено.");     // Выводим сообщение в консоль, если письмо не найдено
}
```


---

## Работа с базой данных. Создание SQL-запросов для проверки данных

---
```sql
-- user registration
SELECT * FROM public."Users" u WHERE u."Login" = 'Astarion+04';


-- activation check
SELECT u."Activated" from public."Users" u WHERE u."UserId" = '605c0511-cc79-4fb2-96ff-8254b23aeabd';


SELECT u."Activated" from public."Users" u WHERE u."UserId" = '605c0511-cc79-4fb2-96ff-8254b23aeabd';
```
