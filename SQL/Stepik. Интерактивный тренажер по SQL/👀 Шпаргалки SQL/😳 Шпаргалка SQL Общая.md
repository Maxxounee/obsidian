## Порядок выполнения команд на сервере

```
1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY
```



### BETWEEN

```sql
SELECT title, amount 
FROM book
WHERE amount BETWEEN 5 AND 14; /*WHERE amount >= 5 AND amount <=14;*/
```


### LIKE

```sql
SELECT title FROM book 
WHERE   title LIKE "_% и _%" /*отбирает слово И внутри названия */
    OR title LIKE "и _%" /*отбирает слово И в начале названия */
    OR title LIKE "_% и" /*отбирает слово И в конце названия */
    OR title LIKE "и" /* отбирает название, состоящее из одного слова И */
```


### CONCAT

```sql
SELECT 
    'Донцова Дарья' AS author,
    CONCAT('Евлампия Романова и ',  title) AS title,
    ROUND(price * 1.42, 2) AS price
```

### SUM COUNT

```sql
SELECT
    author "Автор", /* Вместо AS можно использовать синтаксис с кавычками */
    COUNT(author) AS Различных_книг,
    SUM(amount) AS Количество_экземпляров
FROM book
GROUP BY author;
```

### MIN MAX AVG

```sql
SELECT
    author,
    MIN(price) AS Минимальная_цена,
    MAX(price) AS Максимальная_цена,
    AVG(price) AS Средняя_цена
FROM book
GROUP BY author;
```

### IS NULL

```sql
IF(VAR IS NULL, A, B)
```
