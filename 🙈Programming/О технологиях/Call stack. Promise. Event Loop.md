https://www.youtube.com/watch?v=DteQDxx5iB8

```js
Promise.resolve()
	.then(() => console.log(1))
	.then(() => console.log(2))
	.catch(() => console.log(3))
	.then(() => console.log(4))

Promise.reject()
	.then(() => console.log(5))
	.then(() => console.log(6))
	.catch(() => console.log(7))
	.then(() => console.log(8))
```

Результат выполнения: `1 2 7 4 8`.

___ 

Then отрабатывает синхронно после резолва (потому что резолв выполнен синхронно через точку). Но callback then'a вызывается асинхронно и помещается в стек microtasks queue.

Сначала JS проходит синхронно по коду и выполняется в следующем порядке:

```js
Promise.resolve()
	.then(() => console.log(1))

Promise.reject()
	.then(() => console.log(5))
```

Все это помещается в microtasks queue. После завершения чтения кода и основного call stack, JS вытаскивает JOB из microtasks queue и выполняет `1` (`5` не выполняется, потому что у нас reject). После чего промис меняет свое состояние с pending на fullfilled/rejected и начинается вызов следующих then:

```js
Promise.resolve()
	.then(() => console.log(1)) // Fullfilled
	.then(() => console.log(2)) // Pending

Promise.reject()
	.then(() => console.log(5)) // Rejected. Callback не вызывается
	.then(() => console.log(6)) // Pending
```

JS выполняет все как в прошлый раз. Вытаскивает из очереди и получается: `1 2` (6 опять игнорируется как в прошлом случае)

Продолжаем чтение кода:

```js
Promise.resolve()
	.then(() => console.log(1)) // Fullfilled
	.then(() => console.log(2)) // Fullfilled
	.catch(() => console.log(3)) // Pending. 

Promise.reject()
	.then(() => console.log(5)) // Rejected. Callback не вызывается
	.then(() => console.log(6)) // Rejected. Callback не вызывается
	.catch(() => console.log(7)) // Pending
```

Выполняется `1 2 7` (3 игнорируется, потому что resolve)

Далее:

```js
Promise.resolve()
	.then(() => console.log(1)) // Fullfilled
	.then(() => console.log(2)) // Fullfilled
	.catch(() => console.log(3)) // Fullfilled. Callback Не вызывается
	.then(() => console.log(4)) // Fullfilled

Promise.reject()
	.then(() => console.log(5)) // Rejected. Callback не вызывается
	.then(() => console.log(6)) // Rejected. Callback не вызывается
	.catch(() => console.log(7)) // Fullfilled.
	.then(() => console.log(8)) // Fullfilled
```

Если в catch не выброшена никакая ошибка, то JOB считается fullfilled и следующие then отработают.

Затем добавляются 4 и 8. Получаем `1 2 7 4 8`



Чуть точнее:

Вызов then происходит синхронно. Callback из then сразу попадает в очередь микротасок и выполнится только выполнения синхронной части кода

## Порядок выполнения eventloop

1. Синхронный код (весь)
2. Все микротаски (Promise, await, Observer'ы)
3. Одна макротаска (setTimeout, setInterval, UI handlers')
4. Снова все микротаски
5. Рендеринг (браузер)
6. Следующая макротаска
