## Что делает `constrained()`

`constrained()` — это метод в миграциях Laravel, который **автоматически создаёт внешний ключ** для колонки, созданной через `foreignId()` и `index`. Он избавляет от ручного написания `foreign()->references()->on()`.

## Базовый пример

Вместо:

```php
$table->unsignedBigInteger('category_id')->nullable();
$table->foreign('category_id')
    ->references('id')
    ->on('categories');
```

Пишете:

```php
$table->foreignId('category_id')->nullable()->constrained();
```

Одна строка вместо трёх — результат **тот же самый**.

### Что именно он делает

`foreignId('category_id')` создаёт колонку `BIGINT UNSIGNED`. Затем `constrained()`:

1. **Угадывает таблицу** по имени колонки: `category_id` → `categories`.
2. **Угадывает колонку** для ссылки: по умолчанию `id`.
3. **Создаёт FK** с именем по конвенции: `posts_category_id_foreign`.
4. **Создаёт индекс** на `category_id` (нужен для FK).

## foreignId отличия foreign

Оба метода участвуют в создании внешних ключей, но работают на **разных уровнях**:

- **`foreignId()`** — создаёт **колонку** (и заодно может создать FK).
- **`foreign()`** — создаёт **ограничение** FK на уже существующей колонке.

### foreignId('category_id')

Это **сокращение** для создания колонки типа `BIGINT UNSIGNED`:

```php
$table->foreignId('category_id');
```

Эквивалент:

```php
$table->unsignedBigInteger('category_id');
```

Сам по себе `foreignId()` **не создаёт внешний ключ** — только колонку. FK появляется, если вы прицепите `constrained()` или `foreign()`:

```php
// FK создаётся автоматически, имя таблицы угадывается
$table->foreignId('category_id')->constrained();
// FK создаётся вручную, полный контроль
$table->foreignId('category_id')->nullable();
$table->foreign('category_id')->references('id')->on('categories');
```

### foreign('category_id')

Это метод, который создаёт **ограничение** FOREIGN KEY на **уже существующей** колонке:

```php
$table->foreign('category_id', 'post_category_fk')
    ->references('id')
    ->on('categories');
```

Он **не создаёт колонку**. Если колонки `category_id` нет — будет ошибка.


## Три способа записать одно и то же

**Способ 1 — самый короткий (современный):**

```php
$table->foreignId('category_id')->nullable()->constrained();
```

**Способ 2 — `foreignId` + ручной `foreign`:**

```php

$table->foreignId('category_id')->nullable();
$table->foreign('category_id')
    ->references('id')
    ->on('categories');
```

**Способ 3 — без `foreignId`, вручную:**

```php
$table->unsignedBigInteger('category_id')->nullable();
$table->foreign('category_id')
    ->references('id')
    ->on('categories');
```



Способы 1 и 2 дают **одинаковый SQL**, но имя FK отличается:

- Способ 1: `posts_category_id_foreign` (авто).
- Способ 2: тоже `posts_category_id_foreign` (авто, если имя не задано).
- Если задать имя явно: `->foreign('category_id', 'post_category_fk')` → `post_category_fk`.
    

## Зачем вообще нужен отдельный `foreign()`

`constrained()` покрывает 95% случаев, но `foreign()` нужен, когда:

1. **Колонка уже есть** (например, добавляете FK в отдельной миграции):
    
```php
Schema::table('posts', function (Blueprint $table) {
        $table->foreign('category_id')->references('id')->on('categories');
    });
```
    
2. **Нужно нестандартное имя FK**:
    ```php
      $table->foreign('category_id', 'post_category_fk')
        ->references('id')->on('categories');
```
    
3. **Ссылка на нестандартную колонку** (не `id`):
    
    ```php
      $table->foreign('category_uuid')
        ->references('uuid')->on('categories');
```
    
4. **Составной FK** (несколько колонок):
    
```php
    $table->foreign(['country_id', 'region_id'])
        ->references(['country_id', 'region_id'])->on('regions');
```
    
5. **Разные типы колонок** (например, `string` вместо `bigint`):
    
    ```php
    $table->string('category_slug');
    $table->foreign('category_slug')->references('slug')->on('categories');
```
    
    Здесь `foreignId` не подойдёт — он всегда создаёт `BIGINT UNSIGNED`.