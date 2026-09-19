
### Coalesce. Заменить NULL на 0

Еще есть метод ISNULL(a, 0)

```sql
SELECT
    author.name_author,
    book.title,
    COALESCE(SUM(buy_book.amount), 0) AS Количество
FROM
    buy_book
    RIGHT JOIN book USING(book_id)
    JOIN author USING(author_id)
GROUP BY
    book.title, author.name_author


```


### Проверка на истинность

```sql
SELECT buy_id, date_step_end
FROM step JOIN buy_step USING(step_id)
WHERE step_id = 1 AND date_step_end; /* Тут может быть NULL */
```
