# TypeScript

- Use a function (async when needed) with `/** @DCI-context */` in a JSDoc comment.
- Keep caller-facing Context parameters in lower camel case, then bind caller-supplied objects to PascalCase Role aliases inside their regions, for example `const SourceAccount = sourceAccount;`. This is an identity-preserving alias, not a copy or wrapper.
- Prefer an existing domain interface or type for a Role Contract when one already describes the needed data. Use a small inline object-literal contract for a local Role without an existing model type; do not expand a Role Contract to unrelated properties.
- Name RoleMethods `Role_method`; use `Role__method` for same-Role-only methods. These are lexical functions, not methods attached to Data objects. Add a return value type to the RoleMethods.
- Group each Role's binding and methods in `//#region RoleName Role /////` / `//#endregion` blocks. Put the Role binding first in its region.
- Bind all Roles before interaction unless there is a compelling reason to defer binding. Keep a transient Context's System Operation visible at the end of the function, after the Role regions. Use guards there if needed, then make one initial RoleMethod call. That method should start a coherent message chain through other Roles' RoleMethods, rather than having the Context call several apparently unrelated RoleMethods. Otherwise, return an object exposing System Operation methods for later invocation.

## Validation

Run the project's configured TypeScript checker and inspect editor diagnostics, such as VS Code Problems, workspace-wide after editing. Some diagnostics providers can omit compiler errors when queried with a file filter, so a filtered result with no errors is not enough to claim the workspace is clean. If errors persist or scoped results conflict with what the user sees, rerun diagnostics without file filters and resolve the reported errors. Passing tests or running Node with `--experimental-strip-types` only verifies runtime behavior; it does not type-check the code. If no type checker is configured or available, state that type checking could not be run.

## Hello world example

A transient Context, demonstrating Role binding and interaction. The Speaker Role is passed to the Context, the World Role is assigned directly to console in this simple example.

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

## Money transfer example

A simplified use case of a Money transfer in a banking domain. Demonstrates the distributed algorithm and how it builds the state throughout the Context, contract-like assertions, and returning an object for future use, compared to a transient context (even though this Context could be written as transient). Also shows how to use an interface instead of literal types, when multiple roles have the same contract.

```ts
export interface TransferRecord {
  id: string;
  fromAccountId: string;
  toAccountId: string;
  amount: number;
  occurredAt: string;
}

export interface Account {
  id: string;
  readonly balance: number;
  ledger: TransferRecord[];
}

/** @DCI-context */
export function MoneyTransfer(
  sourceAccount: Account,
  destinationAccount: Account,
  amount: number,
): void {
  //#region SourceAccount Role /////

  const SourceAccount = sourceAccount;

  function SourceAccount_validate(): void {
    if (
      !Number.isSafeInteger(SourceAccount.balance) ||
      SourceAccount.balance < 0
    ) {
      throw new RangeError(
        "Source account balance must be a non-negative safe integer",
      );
    }
    if (amount > SourceAccount.balance) {
      throw new RangeError("Insufficient funds");
    }
    if (!Array.isArray(SourceAccount.ledger)) {
      throw new TypeError("Source account ledger must be an array");
    }

    DestinationAccount_validate();
  }

  function SourceAccount_transfer(): void {
    const transfer: TransferRecord = {
      id: Math.random().toString(36).substring(2, 15),
      fromAccountId: SourceAccount.id,
      toAccountId: DestinationAccount.id,
      amount: amount,
      occurredAt: new Date().toISOString(),
    };
    SourceAccount.ledger.push(transfer);

    DestinationAccount_receive(transfer);
  }
  //#endregion

  //#region DestinationAccount Role /////

  const DestinationAccount = destinationAccount;

  function DestinationAccount_validate(): void {
    if (
      !Number.isSafeInteger(DestinationAccount.balance) ||
      DestinationAccount.balance < 0
    ) {
      throw new RangeError(
        "Destination account balance must be a non-negative safe integer",
      );
    }
    if (!Number.isSafeInteger(DestinationAccount.balance + amount)) {
      throw new RangeError(
        "Destination account balance would exceed the safe integer range",
      );
    }
    if (!Array.isArray(DestinationAccount.ledger)) {
      throw new TypeError("Destination account ledger must be an array");
    }

    SourceAccount_transfer();
  }

  function DestinationAccount_receive(transfer: TransferRecord): void {
    DestinationAccount.ledger.push(transfer);
  }

  //#endregion

  {
    if (SourceAccount === DestinationAccount) {
      throw new RangeError("Source and destination accounts must be different");
    }

    if (!Number.isSafeInteger(amount) || amount <= 0) {
      throw new RangeError("Transfer amount must be a positive safe integer");
    }

    return {
      transfer: () => SourceAccount_validate();
    }
  }
}
```
