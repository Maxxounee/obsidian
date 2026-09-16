### Через ALL/ANY

```sql
WHERE genre_id IN (
	SELECT genre_id
	FROM book
	GROUP BY genre_id
	HAVING SUM(amount) >= ALL(SELECT SUM(amount) FROM book GROUP BY genre_id)
)
```

### Через ORDER BY и LIMIT

```sql
...
WHERE SUM(amount) = (
	SELECT SUM(amount) AS sum_amount
	FROM book
	GROUP BY genre_id
	ORDER BY sum_amount DESC
	LIMIT 1
)
```