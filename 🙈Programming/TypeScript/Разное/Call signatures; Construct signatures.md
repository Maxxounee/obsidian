
### Call signature

```ts
type DescribableFunction = {  
    description: string;  
    (test: number, someArg: number): boolean;  
};  
function doSomething(fn: DescribableFunction) {  
    console.log(fn.description + " returned " + fn(-3, 2));  
}  
  
function myFunc(someArg: number, test: number) {  
    return (someArg + test) > 0;  
}  
myFunc.description = "default description";  
  
doSomething(myFunc);
```

### Construct

```ts
type SomeConstructor = {
	new (s: string): SomeObject;
};

function fn(ctor: SomeConstructor) {
	return new ctor("hello");
}
```