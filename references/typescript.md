# TypeScript

- Use a function (async when needed) with `/** @DCI-context */` in a JSDoc comment.
- Keep caller-facing Context parameters in lower camel case, then bind caller-supplied objects to PascalCase Role aliases inside their regions, for example `const SourceAccount = sourceAccount;`. This is an identity-preserving alias, not a copy or wrapper.
- Prefer an existing domain interface or type for a Role Contract when one already describes the needed data. Use a small inline object-literal contract for a local Role without an existing model type; do not expand a Role Contract to unrelated properties.
- Name RoleMethods `Role_method`; use `Role__method` for same-Role-only methods. These are lexical functions, not methods attached to Data objects. Add a return value type to the RoleMethods.
- Group each Role's binding and methods in `//#region RoleName Role /////` / `//#endregion` blocks. Put the Role binding first in its region.
- Bind all Roles before interaction unless there is a compelling reason to defer binding. Keep a transient Context's System Operation visible at the end of the function, after the Role regions. Use guards there if needed, then make one initial RoleMethod call. That method should start a coherent message chain through other Roles' RoleMethods, rather than having the Context call several apparently unrelated RoleMethods. Otherwise, return an object exposing System Operation methods for later invocation.

```ts
/**
 * A Speaker proclaims something to the World, that dutifully notes it.
 *
 * @DCI-context
 */
function SayHello(speaker: { phrase: string }): void {
  //#region Speaker Role /////

  const Speaker = speaker;

  function Speaker_proclaim(): void {
    World_note(Speaker.phrase);
  }
  //#endregion

  //#region World Role /////

  const World: { log(message: string): void } = console;

  function World_note(message: string): void {
    World.log(message);
  }

  //#endregion

  {
    Speaker_proclaim();
  }
}
```

Only `World_*` methods access `World` directly. `Speaker_proclaim` only sends it a message. The `Speaker` Role is the original `speaker` object, bound inside its region; the System Operation sends one message to `Speaker_proclaim`, which sends the next to `World_note`. For a collection Role, use a contract such as `Messages: Iterable<{ show(message: string): void }>` and iterate only inside `Messages_*` methods.
