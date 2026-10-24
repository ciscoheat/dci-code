---
name: dci
description: Write, refactor, or review code using the DCI (Data, Context and Interaction) paradigm. Use when asked for DCI architecture, role contracts, role interactions, or translating a user story or mental model into a DCI Context. Includes TypeScript, JavaScript, and PHP conventions.
---

# DCI Coding Skill

## Workflow

1. Read the target code, nearby tests, and project conventions. For new code, identify the user story, participating objects, and desired outcome. Ask only when missing behavior blocks implementation. If the task is simple CRUD, functional data transformation or there seems to be only a single Role, explain that DCI adds little value in this case and ask for confirmation before proceeding.
2. Model one use case as a Context: identify its Data objects, meaningful Roles, minimal Role Contracts, and System Operations. Decide its lifetime before implementation: the Context can be transient, i.e. it runs a System Operation and then goes out of scope; alternatively, the Context returns an object, representing itself, whose methods are System Operation entrypoints for later invocation. Do not invent technical Roles merely to split code.
3. If the Role should be supplied by the caller, explicitly request it as a parameter; if created internally, instantiate it within the Context.
4. Load only the target language reference: [TypeScript](./references/typescript.md), [JavaScript](./references/javascript.md), or [PHP](./references/php.md). For another language, apply the shared rules but disclose that no language-specific recipe is supplied; do not silently substitute one.
5. Implement the smallest complete interaction, preserving existing behavior and public APIs during refactors. Bind Roles before interaction; either run the transient Context's System Operation or return an object with callable System Operation entrypoints. Keep Roles and RoleMethods encapsulated in both forms.
6. Check the rules below and run relevant existing tests and type/lint checks. Add focused behavior tests using the project's conventions. Report the Context, Roles, changes, and verification briefly. For review-only requests, report findings without editing.

## Shared Rules

- Data owns state and general domain behavior, never Context-specific interaction logic.
- Preserve the user's data model and source of truth. Do not invent fields, identifiers, timestamps, units, or state mutations unless the use case requires them.
- A Role is a Context-local identifier for an existing object, not a class, wrapper, or external role factory. Preserve object identity and do not expose Roles outside the Context.
- A Role Contract specifies only what its Role needs. Collections can play Roles; include the item shape in their contract.
- A RoleMethod directly reads or writes only its own Role Contract. Cross-Role collaboration calls the other Role's RoleMethods, never accesses its object directly. Private RoleMethods are same-Role only.
- RoleMethods must be kept inside the Context, grouped by Role. Every RoleMethod must belong to a bound Role and advance the use case; do not disguise generic helpers as RoleMethods.
- Bind all Roles at initialization. If rebinding is necessary, reassign all Roles in one `rebind` operation; never change one binding mid-interaction.
- Treat Context execution as a distributed algorithm: the System Operation sends one initial message to a RoleMethod, and RoleMethods advance the use case by sending messages to other Roles' RoleMethods. Keep the Context itself to setup, precondition guards, and that initial call; do not implement the use case as a flat sequence of unrelated RoleMethod calls from the Context. Return values are fine when clearer. Use a nested Context only for a distinct, reusable use case.

## Final Check

Inspect every RoleMethod for cross-Role property access, missing contracts, incorrect naming/visibility, and unbound Roles. Confirm identity is preserved, bindings are complete before interaction, and that a System Operation starts a coherent RoleMethod message chain (immediately or through returned System Operations, matching the chosen lifetime). Verify behavior, including relevant failure paths; report checks that could not run.
