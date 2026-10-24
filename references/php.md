# PHP

- Use a `final` Context class annotated `@DCIContext` or `@context`.
- Define minimal Role Contract interfaces in the same file, before the Context. Bind Roles to private, interface-typed properties in the constructor or one complete `rebind` method.
- Name RoleMethods `Role_method`, matching the property name. Use `protected` for cross-Role calls; `private` methods may be called only by their own Role's methods.
- Group methods with `///// RoleName /////` comments. Prefix ordinary, non-Role properties with `_`. Avoid public state; if unavoidable, mark non-Role public properties `@noDCIRole`.
- A public System Operation has a use-case name such as `greet` or `transfer`, not `run` or `execute`. It initiates a RoleMethod chain. Call it after construction; construction alone is not the use case.
- The narrow `@$this->role` exception permits passing a roleplayer to a nested Context, not general cross-Role access.

```php
<?php
interface Speaker {
    public function phrase(): string;
}

interface World {
    public function log(string $message): void;
}

/** @DCIContext */
final class SayHello {
    private Speaker $Speaker;
    private World $World;

    public function __construct(Speaker $speaker, World $world) {
        $this->Speaker = $speaker;
        $this->World = $world;
    }

    public function greet(): void {
        $this->Speaker_proclaim();
    }

    ///// speaker /////
    protected function Speaker_proclaim(): void {
        $this->World_note($this->Speaker->phrase());
    }

    ///// world /////
    protected function World_note(string $message): void {
        $this->World->log($message);
    }
}
```

The caller invokes `(new SayHello($speaker, $world))->greet()` with objects implementing the contracts. Only `world_*` methods directly access `$this->world`.
