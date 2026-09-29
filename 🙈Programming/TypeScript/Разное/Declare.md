`declare` — это ключевое слово TypeScript, которое говорит компилятору: **«эта штука существует в runtime, но её определение лежит вне текущего файла (или вообще вне TS). Не генерируй для неё JS, просто поверь мне на слово»**.

Проще говоря, `declare` — это **обещание компилятору**, что что-то есть, без реальной реализации в этом файле. После компиляции от `declare` не остаётся **ничего** — он полностью стирается.

## Зачем он нужен

TypeScript должен знать типы всего, что вы используете. Но не всё можно описать внутри `.ts`:

- глобальные переменные из браузера (`window`, `document`);
    
- модули на JS без типов (`lodash`, старые библиотеки);
    
- переменные, которые подставляет сборщик (`process.env`, `__DEV__`);
    
- глобальные функции из подключаемых скриптов.
    

Для таких случаев и нужен `declare`.

## Основные формы

### 1. `declare const` / `let` / `var` — глобальная переменная

```ts
declare const API_URL: string;
console.log(API_URL);   // ✅ TS знает тип
```


В скомпилированном JS останется только `console.log(API_URL)` — сама переменная **не создаётся**. Если в runtime её не будет, получите `ReferenceError`.

### 2. `declare function` — глобальная функция

```ts
declare function greet(name: string): void;
greet("Alice");   // ✅ типы есть, реализации нет
```

### 3. `declare class`

```ts
declare class Animal {
    name: string;
    constructor(name: string);
    speak(): void;
}
```

Используется, когда класс приходит извне (например, из JS-библиотеки).

### 4. `declare module` — типы для модуля без типов

```ts
declare module "my-lib" {
    export function doSomething(x: number): string;
    export const version: string;
}
```

Теперь можно писать:

```ts
import { doSomething, version } from "my-lib";   // ✅
```

Классический случай — библиотека написана на JS, а типов нет. Пишете `.d.ts` с `declare module`, и всё работает.

### 5. `declare global` — расширение глобальной области

```ts
export {};   // важно: делает файл модулем
declare global {
    interface Window {
        myAnalytics: {
            track(event: string): void;
        };
    }
}
window.myAnalytics.track("click");   // ✅
```

Без `export {}` файл считается скриптом, и `declare global` работать не будет.

### 6. `declare namespace`

```ts
declare namespace MyLib {
    function foo(): void;
    const bar: number;
}
```

Устаревающий, но всё ещё встречается в старых `.d.ts` (например, в DefinitelyTyped).

## Ключевое свойство: `declare` = только типы

```ts
declare const x: number;
console.log(x);   // ✅ компилируется
```

JS-вывод:

```ts
console.log(x);
```

Никакого `const x = ...` не появится. Это **только типовая информация**.

## Где `declare` используется чаще всего

### Файлы `.d.ts`

Файлы деклараций — это практически целиком `declare`:

```ts
// types/global.d.ts
declare const __DEV__: boolean;
declare function require(path: string): any;
interface Window {
    gtag(...args: any[]): void;
}
```

### Аугментация модулей

Добавить поле в существующий тип:

ts
```ts
// types/express.d.ts
import "express";
declare module "express" {
    interface Request {
        user?: { id: string };
    }
}
```

Теперь `req.user` доступен во всём проекте.

### Паттерн `declare module "*.css"`

```ts
declare module "*.css" {
    const styles: { [key: string]: string };
    export default styles;
}
declare module "*.png" {
    const src: string;
    export default src;
}
```

Позволяет импортировать CSS/картинки в TS без ошибок.

## `declare` vs `interface`/`type`

Тут часто путаются. Разница:

```ts
// type/interface — просто описание типа, ничего не "объявляют"
type User = { name: string };
// declare — обещание, что значение существует
declare const user: User;
```

`type` и `interface` **всегда** только описывают форму. `declare` нужен, когда вы **связываете** тип с именем, которое реально будет в JS.

## `declare` vs `export`

```ts
// ✅ Экспортируемая функция с реализацией
export function foo() { return 1; }
// ✅ Объявление без реализации (в .d.ts)
export declare function foo(): number;
```

В `.d.ts` файлах `declare` часто неявный — но если файл не `.d.ts`, без `declare` TS попросит реализацию.

## Практический пример

```ts
// globals.d.ts
declare const BUILD_VERSION: string;
declare function trackEvent(name: string, data?: object): void;
declare module "legacy-lib" {
    export function oldApi(x: number): string;
}

// app.ts
console.log(BUILD_VERSION);        // ✅ типы есть
trackEvent("page_view");           // ✅
import { oldApi } from "legacy-lib";
oldApi(42);                        // ✅
```

Сборщик (Webpack/Vite) подставит реальные значения `BUILD_VERSION`, `trackEvent` и модуль `legacy-lib` в runtime. TypeScript же довольствуется декларациями.

## Итоговая таблица

|Форма|Что объявляет|
|---|---|
|`declare const x: T`|глобальная переменная|
|`declare function f(): T`|глобальная функция|
|`declare class C {}`|класс извне|
|`declare module "m" {}`|типы для модуля|
|`declare global {}`|расширение глобальной области|
|`declare namespace N {}`|пространство имён (устаревает)|
|`declare module "*.css"`|wildcard-импорты|

## Главное, что нужно запомнить

1. **`declare` не генерирует JS** — это чистая типовая информация.
2. **`declare` ничего не создаёт в runtime** — если значения нет, будет `ReferenceError`.
3. **`declare` говорит компилятору «поверь мне»** — проверки существования не будет.
4. **Чаще всего живёт в `.d.ts`** файлах — там он почти всегда неявный.
5. **Основные применения**: глобальные переменные, JS-библиотеки без типов, wildcard-импорты, аугментация модулей.
    

Если хотите, могу показать, как правильно написать `.d.ts` для конкретной библиотеки или как расширить типы `process.env` через `declare global` — это одна из самых частых задач.