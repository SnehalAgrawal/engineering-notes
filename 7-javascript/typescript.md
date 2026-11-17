# TypeScript

## Overview
TypeScript adds types to JavaScript. Types are checked when you compile and then removed. They catch mistakes early but do nothing at runtime. Typing is structural: two types match if their shapes match, whatever their names.

## Key concepts
*   **Basic types**: `string`, `number`, `boolean`, `bigint`, `symbol`, `null`, `undefined`, arrays (`number[]`), tuples (`[string, number]`).
*   **`any`**: Turns type checking off. Avoid it.
*   **`unknown`**: Accepts any value, but you must check its type before using it. Use it for untrusted input.
*   **`void`**: The function returns nothing useful.
*   **`never`**: No value can exist. Example: a function that always throws.
*   **Inference**: TypeScript guesses types from values. `let count = 3` is a `number`. Annotate function parameters and public APIs.
*   **Optional and readonly**:
    ```typescript
    interface Profile {
      readonly id: number;  // cannot be reassigned
      bio?: string;         // may be missing
    }
    ```
*   **Union (`A | B`)**: A value of either type. Example: `type Status = 'active' | 'inactive'`.
*   **Intersection (`A & B`)**: A value with the members of both. Example: `type Admin = User & { isAdmin: boolean }`.
*   **Narrowing**: Checks like `typeof`, `instanceof`, `in`, or a `kind` field tell TypeScript which type you have.
    ```typescript
    function printId(id: string | number) {
      if (typeof id === 'string') console.log(id.toUpperCase());
      else console.log(id.toFixed(2));
    }
    ```
*   **Discriminated union**: Each variant has a literal `kind` field. A `switch` on `kind` narrows to the right variant.
*   **Generics**: Type parameters link input and output types. `extends` limits what a type parameter can be.
    ```typescript
    function wrap<T>(value: T): T[] { return [value]; } // wrap('a') is string[]

    function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
      return obj[key];
    }
    getProp({ name: 'Alice', age: 30 }, 'age'); // number
    ```
*   **Utility types**: `Partial`, `Required`, `Pick`, `Omit`, `Record`, `ReturnType`, `Awaited`.
*   **`as`**: Tells the compiler "trust me". No runtime check. Example: `document.getElementById('email') as HTMLInputElement`.
*   **`satisfies`**: Checks a value against a type but keeps its exact inferred type.
*   **Enums**: Named constants, like `enum Role { Admin, User, Guest }` (`Role.Admin` is `0`). They create real objects at runtime. A union of string literals (`'admin' | 'user'`) is usually simpler. Literal unions also work with tools that only strip types, such as Node.js's built-in type stripping or the `erasableSyntaxOnly` option.

## `interface` vs `type`
Both describe object shapes. For plain objects it is mostly style.

| | `interface` | `type` |
| --- | --- | --- |
| Object shape | Yes | Yes |
| Extending | `extends`, conflicts are an error | `&`, conflicts become `never` |
| Unions, tuples, primitives | No | Yes |
| Mapped and conditional types | No | Yes |
| Merging two declarations of the same name | Yes | No |

A common habit: `interface` for object shapes others extend (public APIs), `type` for unions, function types, and computed types.

## Tradeoffs
*   **Safety vs effort**: Types catch many bugs and make refactoring safe. Very clever types can be harder to read than the code.
*   **Compile time only**: Data from the network is not checked. Use a schema library (Zod, Valibot) at those edges.

## When not to use
*   **`any` to silence errors**: Use `unknown` and narrow, or fix the type.
*   **Complex types in app code**: If a type needs a paragraph to explain, use a simpler type and a runtime check.

## Interview focus
*   **`any` vs `unknown`**: Both accept anything. `unknown` makes you check before use.
*   **`interface` vs `type`**: See the table. Mention declaration merging and unions.
*   **Structural typing**: Any object with the needed properties fits.
*   **Exhaustive `switch`**: In `default`, assign the value to a `never` variable. A missed case becomes a compile error.
*   **What `as` does at runtime**: Nothing. It is removed.

## Common mistakes
*   **`strict` turned off**: Without `strictNullChecks`, `null` fits every type and hides bugs.
*   **Hiding `null` with `as` or `!`**: `getElementById(...)!` can still be `null` at runtime.
*   **Trusting API data**: `(await res.json()) as User` checks nothing.
