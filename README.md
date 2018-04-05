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

The difference is `let` can change, so TypeScript sets the type to `string`. `const` cannot change, so TypeScript narrows the type to a literal `'my string'`. TypeScript will do that will all types including `number` and `boolean` types.

### Avoid overtyping
TypeScript has powerful inference, especially when using `const`. A common problem is accidentally widening types when overtyping.

```ts
// bad
const foo: string = 'foo' // string

// good
const foo = 'foo' // 'foo'
```

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
const foo = getFoo()
foo.length // Type error
```
