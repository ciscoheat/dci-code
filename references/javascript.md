# JavaScript

- Use a function (async when needed) annotated `@DCI-context`. Add `// @ts-check` in new files.
- Keep caller-facing Context parameters in lower camel case, then bind caller-supplied objects to PascalCase Role aliases inside their regions, for example `const SourceAccount = sourceAccount;`. This is an identity-preserving alias, not a copy or wrapper.
- Prefer an existing domain typedef or JSDoc contract for a Role when one already describes the needed data. Otherwise use a small inline JSDoc object contract; do not add unrelated properties.
- Name RoleMethods `Role_method`; use `Role__method` for same-Role-only methods. These are lexical functions, not methods attached to Data objects. Add a return value type to the RoleMethods.
- Group each Role's binding and methods in `//#region RoleName Role /////` / `//#endregion` blocks. Put the Role binding first; add `@type` when its contract cannot be inferred from the parameter or initializer.
- Bind all Roles before interaction unless there is a compelling reason to defer binding. Keep a transient Context's System Operation visible at the end of the function, after the Role regions. Use guards there if needed, then make one initial RoleMethod call. That method should start a coherent message chain through other Roles' RoleMethods, rather than having the Context call several apparently unrelated RoleMethods. Otherwise, return an object exposing System Operation methods for later invocation.

```js
// @ts-check

/**
 * A Speaker proclaims something to the World, that dutifully notes it.
 *
 * @DCI-context
 * @param {{ phrase: string }} speaker
 */
function SayHello(speaker) {
  //#region Speaker Role /////

  const Speaker = speaker;

  function Speaker_proclaim() {
    World_note(Speaker.phrase);
  }

  //#endregion

  //#region World Role /////

  /** @type {{ log: (message: string) => void }} */
  const World = console;

  /** @param {string} message */
  function World_note(message) {
    World.log(message);
  }

  //#endregion

  {
    Speaker_proclaim();
  }
}
```

Only `World_*` methods access `World` directly. The `Speaker` Role is the original `speaker` object, bound inside its region; the System Operation sends one message to `Speaker_proclaim`, which sends the next to `World_note`. For a collection Role, declare its item contract, for example `@param {Iterable<{ show: (message: string) => void }>} Messages`, and iterate within `Messages_*` methods.
