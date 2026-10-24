# DCI Coding Skill

An agentic coding skill for applying the [Data, Context and Interaction](https://en.wikipedia.org/wiki/Data,_context_and_interaction) (DCI) paradigm to implementation, refactoring, and code review. It helps model a use case as a Context, identify its Data and Roles, define minimal Role Contracts, and implement interactions while preserving existing behavior and APIs.

The skill includes language-specific guidance for TypeScript, JavaScript, and PHP in `references/`. It can also be used for other languages, with the limitation that no language-specific recipe is provided.

## Install

This skill is published from this GitHub repository and can be installed with the [skills CLI](https://skills.sh/):

```sh
npx skills add ciscoheat/dci-code
```

The CLI installs the `dci` skill for a supported coding agent. To check for and install updates later:

```sh
npx skills update dci
```

## More information about DCI

- [Official DCI homepage](https://fulloo.info)
- [DCI tutorial for TypeScript](https://blog.encodeart.dev/dci-tutorial-for-typescript-part-1)
