---
layout: default
title: Prompt Override
parent: Customize Features
nav_order: 15
---

Prompt Override ([#54](https://github.com/unit-mesh/auto-dev/issues/54)) will override the AutoDev prompt with your own
prompt.

## How to use

create a folder named `prompt/` in your project root directory, then create the prompt file which defined in
Supported Action.

For example, create a file named `prompts/sql/sql-gen-clarify.vm`, will override the clarify prompt of AutoSQL/GenSQL

## Supported Action

```bash
├── cicd
│   └── generate-github-action.vm
├── code
│   ├── api-test-gen.vm
│   ├── code-gen.vm
│   ├── bridge.vm
│   ├── sketch.vm
│   ├── plan.vm
│   ├── plan-reviewer.vm
│   ├── repair-diff.vm
│   └── test-gen.vm
├── error
│   └── fix-error.vm
├── harmonyos
│   ├── arkui-clarify.vm
│   └── arkui-design.vm
├── page
│   ├── page-gen-clarify.vm
│   └── page-gen-design.vm
├── practises
│   ├── code-review.vm
│   ├── gen-commit-msg.vm
│   ├── refactoring.vm
│   ├── release-note.vm
│   └── shell-suggest.vm
├── quick
│   └── quick-action.vm
├── sql
│   ├── sql-gen-clarify.vm
│   └── sql-gen-design.vm
└── sre
    └── generate-dockerfile.vm
```

The system's built-in Prompt files are located in the `src/main/resources/genius/` directory. You can customize your own prompts by referring to these files (keeping the variable names consistent).

For details, see: https://github.com/unit-mesh/auto-dev/tree/master/core/src/main/resources/genius