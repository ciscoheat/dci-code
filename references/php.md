# PHP

- Use a `final` Context class annotated `@DCI-context`.
- Define minimal Role Contract interfaces in the same file, before the Context. Bind Roles to private, interface-typed properties in the constructor or one complete `rebind` method.
- Name RoleMethods `Role_method`, matching the property name. Use `protected` for cross-Role calls; `private` methods may be called only by their own Role's methods.
- Group each Role's binding and methods in `//#region RoleName Role /////` / `//#endregion` blocks.
- Prefix ordinary, non-Role properties with `_`. Avoid public state.
- A public System Operation has a use-case name such as `greet` or `transfer`, not `run` or `execute`. It initiates a RoleMethod chain. Call it after construction; construction alone is not the use case.
- The narrow `@$this->Role` exception permits passing a roleplayer to a nested Context, not general cross-Role access.

## Hello world example

A transient Context, demonstrating Role binding and interaction. The Speaker Role is passed to the Context, as is the World Role.

```php
<?php
interface Speaker {
    public function phrase(): string;
}

interface World {
    public function log(string $message): void;
}

/**
 * A Speaker proclaims something to the World, that dutifully notes it.
 *
 * @DCI-context
 */
final class SayHello {
    public function __construct(Speaker $speaker, World $world) {
        $this->Speaker = $speaker;
        $this->World = $world;
    }

    public function greet(): void {
        $this->Speaker_proclaim();
    }

    //#region Speaker Role /////

    private Speaker $Speaker;

    protected function Speaker_proclaim(): void {
        $this->World_note($this->Speaker->phrase());
    }

    //#endregion

    //#region World Role /////

    private World $World;

    protected function World_note(string $message): void {
        $this->World->log($message);
    }

    //#endregion
}
```

The caller invokes `(new SayHello($speaker, $world))->greet()` with objects implementing the contracts. Only `World_*` methods directly access the `$World` roleplayer; `Speaker_proclaim` sends it a message through `World_note`.

## Money transfer example

A simplified money transfer. The Context is constructed with the two account roleplayers and the amount, then its `transfer()` System Operation starts a message chain. The transfer record is created as the interaction progresses and added to both ledgers. Account and transfer-record contracts are shared by their respective roles.

```php
<?php
interface Account {
    public function id(): string;
    public function balance(): int;
    public function addTransfer(TransferRecord $transfer): void;
}

final class TransferRecord {
    public function __construct(
        private string $id,
        private string $fromAccountId,
        private string $toAccountId,
        private int $amount,
        private string $occurredAt
    ) {}

    public function id(): string {
        return $this->id;
    }

    public function fromAccountId(): string {
        return $this->fromAccountId;
    }

    public function toAccountId(): string {
        return $this->toAccountId;
    }

    public function amount(): int {
        return $this->amount;
    }

    public function occurredAt(): string {
        return $this->occurredAt;
    }
}

/**
 * Transfer money between two accounts.
 *
 * @DCI-context
 */
final class MoneyTransfer {
    private readonly int $_amount;

    public function __construct(
        Account $sourceAccount,
        Account $destinationAccount,
        int $amount
    ) {
        if ($sourceAccount === $destinationAccount) {
            throw new InvalidArgumentException(
                'Source and destination accounts must be different'
            );
        }

        if ($amount <= 0) {
            throw new InvalidArgumentException(
                'Transfer amount must be a positive integer'
            );
        }

        $this->SourceAccount = $sourceAccount;
        $this->DestinationAccount = $destinationAccount;
        $this->_amount = $amount;
    }

    public function transfer(): void {
        $this->SourceAccount_validate();
    }

    //#region SourceAccount Role /////

    private Account $SourceAccount;

    protected function SourceAccount_validate(): void {
        if ($this->SourceAccount->balance() < 0) {
            throw new DomainException(
                'Source account balance must be non-negative'
            );
        }
        if ($this->_amount > $this->SourceAccount->balance()) {
            throw new DomainException('Insufficient funds');
        }
        $this->DestinationAccount_validate();
    }

    protected function SourceAccount_transfer(): void {
        $transfer = new TransferRecord(
            bin2hex(random_bytes(8)),
            $this->SourceAccount->id(),
            $this->DestinationAccount->id(),
            $this->_amount,
            (new DateTimeImmutable('now', new DateTimeZone('UTC')))
                ->format('Y-m-d\\TH:i:s.v\\Z')
        );
        $this->SourceAccount->addTransfer($transfer);

        $this->DestinationAccount_receive($transfer);
    }

    //#endregion

    //#region DestinationAccount Role /////

    private Account $DestinationAccount;

    protected function DestinationAccount_validate(): void {
        $balance = $this->DestinationAccount->balance();
        if ($balance < 0) {
            throw new DomainException(
                'Destination account balance must be non-negative'
            );
        }
        if ($balance > PHP_INT_MAX - $this->_amount) {
            throw new DomainException(
                'Destination account balance would exceed the integer range'
            );
        }
        $this->SourceAccount_transfer();
    }

    protected function DestinationAccount_receive(
        TransferRecord $transfer
    ): void {
        $this->DestinationAccount->addTransfer($transfer);
    }

    //#endregion
}
```

`Account::balance()` makes the balance contract explicit as a PHP integer, while the account implementations own their ledger state and update it through `addTransfer()`. The Context verifies non-negative balances, sufficient funds, and that the destination addition stays within `PHP_INT_MAX`.
