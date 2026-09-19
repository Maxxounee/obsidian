## Request

Что делает request()

Когда вы пишете:

```php
request('email')
```
Это сокращение для:

```php
request()->input('email')
```
То есть:

`request()` без аргументов возвращает объект текущего запроса (Illuminate\Http\Request).

`request('email')` — это сокращённая запись, которая сразу достаёт значение email из этого запроса.

#### Откуда берутся данные

`request('email')` ищет значение email во всех входных данных запроса, в таком порядке:

Параметры маршрута (route parameters):

* Тело запроса (для `POST`, `PUT`, `PATCH` — form data или JSON)
* Query string (для GET, например ?email=test@mail.com)
* Загруженные файлы

Например, при таких запросах:

```
GET  /users?email=test@mail.com
POST /users  (body: email=test@mail.com)
оба вернут "test@mail.com".
```

#### Пример пошагово

Допустим, пришёл POST-запрос с телом:

```
email=ivan@example.com&name=Иван
```

Тогда:

```php
request('email');  // "ivan@example.com"
request('name');   // "Иван"
request('phone');  // null (нет такого поля)
Второй аргумент — значение по умолчанию
```

```php
request('email', 'default@mail.com');
// вернёт 'default@mail.com', если email нет в запросе
```

#### Работа с несколькими полями сразу

```php
request(['email', 'name']);
// вернёт массив: ['email' => '...', 'name' => '...']
```

