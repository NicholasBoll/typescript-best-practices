# TypeScript Best Practices

## Code

### Prefer `const` over `let`
This is a general JavaScript preference to not mutate values. Constants are easier to understand. This can be a change in mindset, but it is a good discipline.

TypeScript understands that you can't change the value of a constant and will give a static error if you do. TypeScript knows a `const` cannot change, so it will _narrow_ the type to be more accurate.

```ts
// bad
let foo = 'my string' // string
let bar = true // boolean
let baz = 1 // number

// good
const foo = 'my string' // 'my string'
const bar = false // false
const baz = 1 // 1
```
[const/let Typescript Playground](https://www.typescriptlang.org/play?#code/DYUwLgBAZg9jEF4IHIC2BPCBnMAnAlgHYDmyEA9OdnkcQFCiQBGAhrohHgK4gVVNxQLQg3ARWALw4BGPhEJdUTELjp1KEYnAAmdAMYxCOaHA5pMOAiTIbz1K6X2HjrdkigtgWXho9eQTkbMLFJIshrSQA)

The difference is `let` can change, so TypeScript sets the type to `string`. `const` cannot change, so TypeScript narrows the type to a literal `'my string'`. TypeScript will do that will all types including `number` and `boolean` types.

### Avoid overtyping
TypeScript has powerful inference, especially when using `const`. A common problem is accidentally widening types when overtyping.

```ts
// bad
const foo: string = 'foo' // string

// good
const bar = 'bar' // 'bar'
```
[Overtyping Typescript Playground](https://www.typescriptlang.org/play?#code/PTAECMEMBMCgGMD2A7AzgF1AM0YgXKBgE4CWyA5qALygDkOitoIh6pFssL5ucSamKEWp0hTFrTFA)

### Avoid casting
Use of `as` should be used with extreme caution. It will often narrow types accidentally, often bypassing `null` checks.

```ts
function getFoo(): string | undefined {
  return false ? 'foo' : undefined
}

// bad
const foo = getFoo() as string // TypeScript won't complain
foo.length // oops, runtime error

// good
const bar = getFoo()
bar.length // Type error "Object is possibly 'undefined'"
```
[Casting Typescript Playground](https://www.typescriptlang.org/play?ssl=11&ssc=58&pln=1&pc=1#code/GYVwdgxgLglg9mABAcwKZQGJzgCgJQBciAzlAE4xjKIA+i4AJqsJag4gN4BQiiZ6IMkmABDADbFUiAPyIA5MGxzERRs1YMuAXy5cA9HsQAjEZogJSiRXEQBeFOiy48iEcRLlK1A4gAqATwAHVABlCApAqEQAdwQ5KPMAW0CxEUouawA6MVQqKAALRB9sQOIAGj5wWESpVDIyODJdH2RsMwsokzI7B0xsfC4u7NzkAqLDAODEOobugCIAeSMAK1RoRBh3QLhiYhgjMX95NRYwNjk5oA)

### Use `as const` to make objects immutable
The caveat to avoiding overtyping is `const` only narrows primitive types. If the `const` will represent an object, the values of each key will still be widened. This is because Typescript knows that objects are mutable. You can easily tell Typescript that the object should not be mutated by casting to a constant. This is a convenient way to declare an informal interface at runtime. Typescript will treat the object as if each key was `readonly`.

```ts
const foo = {
    bar: 'bar', // string
    baz: 10 // number
}

const bar = {
    bar: 'bar', // 'bar'
    baz: 10 // 10
} as const
```

[As const Typescript Playground](https://www.typescriptlang.org/play?ssl=1&ssc=1&pln=9&pc=11#code/MYewdgzgLgBAZiEMC8MDeAoG2YCMCGATgFwwDkBhZANDAPR0zSECWYA5ljgQF6kCMABnqMwAVwC2uAKaEMAXwwZQkWJRTou2SqQpEaI8pTJa8+PjCGGhCmPggwV0IA)

This can be very useful if you need to create an anonymous object (an object with an inferred interface) that needs to be applied to an interface with a narrower type restriction. Here's an example:

```ts
interface CSSObject {
  position: 'relative' | 'absolute' | 'auto' | 'static'
}

function css(obj: CSSObject) {
    return obj
}

const styles1 = {
    position: 'absolute'
}

css(styles1) // Type error: Type 'string' is not assignable to type '"relative" | "absolute" | "auto" | "static"'

const styles2 = {
    position: 'absolute'
} as const

css(styles2) // works
```

[Using as const Typescript Playground](https://www.typescriptlang.org/play?ssl=19&ssc=22&pln=1&pc=1#code/JYOwLgpgTgZghgYwgAgMIGV0HkBGArCBMZAbwChlkAHAewGdgxgaQAuZAciggBs4mAbhA7IAPpzg46NHgFdII8RzjyaiznTD9gCDmQC+ZMjFkgizEMgR06AChr52GbPkJgAlKQqVk3MLKhLBzwDIwQWTWRNAE8eCDoARmQAXi8fanpGC3ZlKRl5YVCyazsYuMTPAHpK5AAVaKoUaCgaKHZ6xo0wKFAAcxFgOmQQGmI4G2BekEk45DAaOYaUDgAibj5BCBWxZBXJaTlIbfE91WPdzW0EFb1iiOIy+IAmFLSfWgYmFhz9-IUDZDjKz3MI2WyPOhPKo1ADurQA1nQyEA)

### Prefer factory functions over explicit interfaces

This might be controversial, but there are some important benefits to using a factory function to create objects vs creating objects against an explicit interface.

```ts
interface CSSObject {
  position: 'relative' | 'absolute' | 'auto' | 'static'
}

const styles1: CSSObject = {
    position: 'absolute' // auto-complete and type checking
}

// factory function
function css(obj: CSSObject) {
    return obj
}

const styles2 = css({
    position: 'absolute' // auto-complete and type checking
})
```

[Factory function vs interface Typescript Playground](https://www.typescriptlang.org/play?ssl=1&ssc=1&pln=16&pc=3#code/JYOwLgpgTgZghgYwgAgMIGV0HkBGArCBMZAbwChlkAHAewGdgxgaQAuZAciggBs4mAbhA7IAPpzg46NHgFdII8RzjyaiznTD9gCDmQC+ZMghabkmgJ48IdAIzsM2fIWIBeUhUrV6jZmwlSMvLCyAD0ocgqYDQAtCYAtlTWkJEgACbIYBZUKAgAFoQA1qAA5gZG4cjwRDRQFlWyIER+ZDCNzSzICHR0ABQ0+A6YuAREAJQeXsjcYLJQIMgDeOXGpsSW1nQATMju3X3kU7QMTCzsyoFyCmERUbEJSRApcOmZ2bkFCMUgZfpjQA)

Typescript has good support for _inline_ anonymous objects which is how the factory method works. Typescript checks the passed in object in a type narrowing fashion. What I mean is Typescript treats the following differently:

```ts
const temp = {
  position: 'absolute' // string
}

css(temp) // type error since `position` is a string

css({
  position: 'absolute' // not an error. Object is used inline and narrowed to the restrictions of `position`
})
```

Some benefits of factory functions vs explicit interface declaration:
- Less syntax is easier to understand
- Factory functions are valid JavaScript, so JavaScript users will benefit equally
- Less user-error. Typescript users don't have to choose between `const foo = {}`, `const foo: Foo = {}`, or `const foo = {} as Foo`.
- User doesn't have to import extra interfaces

Factory functions are also better than `as const` because of the location and specificity of the type error.

```ts
interface CSSObject {
  position: 'relative' | 'absolute' | 'auto' | 'static'
}

// factory function
function css(obj: CSSObject) {
    return obj
}

const styles = {
    position: 'foobar'
} as const

css(styles) // error here, whole stack trace

css({
    position: 'foobar' // error here, specific error about 'position'
})
```

[Factory vs as const Typescript Playground](https://www.typescriptlang.org/play?ssl=18&ssc=3&pln=1&pc=1#code/JYOwLgpgTgZghgYwgAgMIGV0HkBGArCBMZAbwChlkAHAewGdgxgaQAuZAciggBs4mAbhA7IAPpzg46NHgFdII8RzjyaiznTD9gCDmQC+ZMgHpjyeERpQAnudkgizEGRj3HLZAjp0AFDXzsGNj4hGAAlKQUlMjcYLJQIMj+eAZGCCyayJrWPBB0yAC8kdHU9IxO7BwwNP5wUHr6yHD56SCaad4+2bl0EabI0FBWyAAW0BAANMgA7iMyKJqIANbIYFCIEB2+5CW0DEwsldW19cj9g8Nj3FN0VITAMDoDUENQTTg08px75SwNYUA)

Factory functions have the added benefit of type narrowing while still being constrained by an interface using generics. This can be helpful for things like action creators.

