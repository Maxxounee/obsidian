
```ts
interface Func {  
    (arg: number): void;  
    test: number;  
}  
  
const supaFunc = Object.assign(  
    (arg) => (console.log(arg)),  
    {test: 222},  
) as Func;  
  
supaFunc(111); // 111  
supaFunc.test; // 222
```
