---
layout: default
title: Config (Hobbit Hole)
parent: Shire Language
nav_order: 3
---

Hobbit Hole 用于定义数据处理流程与 IDE 交互逻辑。

## Hobbit Hole 概述

```shire
---
name: "Summary"
description: "Generate Summary"
interaction: AppendCursor
actionLocation: ContextMenu
---
```

### 属性说明

- `name`：Shire 命令的显示名称，将显示在 IDE 的 UI 中，基于 [HobbitHole.interaction](#interaction)。
- `description`：操作的提示信息，将显示在 UI 的 Hover tips 中。
- `interaction`：操作的输出可以是编辑器中的流文本，当使用 [InteractionType.AppendCursorStream](#interaction) 时。
- `actionLocation`：操作的位置，应该是 [ShireActionLocation](#actionlocation) 中的一个，默认为 [ShireActionLocation.RUN_PANEL]。
- `selectionStrategy`：选择要应用操作的元素的策略。
- `variables`：用于构建变量的带有 PatternAction 的变量列表。
- `when`：用于 [com.intellij.codeInsight.intention.IntentionAction.isAvailable]、[com.intellij.openapi.project.DumbAwareAction.DumbAwareAction.update] 检查是否显示菜单的条件。
- `ruleBasedFilter`：应用于操作的规则文件列表。
- `onStreamingEnd`：在流处理结束后执行的后中间件操作列表，如日志记录、指标收集、代码验证、运行代码、解析代码等。
- `afterStreaming`：流结束后执行任务的决策，路由到不同的任务。
- `shortcut`：操作的 IDE 快捷键，使用 IntelliJ IDEA 的快捷键格式。
- `enabled`：是否启用操作。
- `model`: 使用的大语言模型，使用 `.shireEnv.json` 定义。
- `userData`：其余数据。

### 示例代码

```shire
---
name: "Summary"
description: "Generate Summary"
interaction: AppendCursor
actionLocation: ContextMenu
when: $fileName.matches("/.*.java/")
variables:
  "var1": "demo"
  "var2": /**.java/ { find("error.log") | sort | xargs("rm")}
---
```

## Hobbit Hole 属性

### Interaction Type

```kotlin
enum class InteractionType(val description: String) {
    AppendCursor("Append content at the current cursor position"),
    AppendCursorStream("Append content at the current cursor position, stream output"),
    OutputFile("Output to a new file"),
    ReplaceSelection("Replace the currently selected content"),
    ReplaceCurrentFile("Replace the content of the current file"),
    InsertBeforeSelection("Insert content before the currently selected content"),
    RunPanel("Show Result in Run panel which is the bottom of the IDE"),
    OnPaste("Copy the content to the clipboard"),
    RightPanel("Show Result in Right panel which is the right of the IDE"),
    StreamDiff("Use streaming diff to show the result")
    ;
}
```

由于性能原因，OnPaste 暂时只支持 Java 和 Kotlin 语言，并且需要行数多于 5 行。

### Action Location

```kotlin
enum class ShireActionLocation(val location: String, val description: String) {
    CONTEXT_MENU("ContextMenu", "Show in Context Menu by Right Click"),
    INTENTION_MENU("IntentionMenu", "Show in Intention Menu by Alt+Enter"),
    TERMINAL_MENU("TerminalMenu", "Show in Terminal panel menu bar"),
    COMMIT_MENU("CommitMenu", "Show in Commit panel menu bar"),
    RUN_PANEL("RunPanel", "Show in Run panel which is the bottom of the IDE"),
    INPUT_BOX("InputBox", "Show in Input Box"),
    DATABASE_MENU("DatabaseMenu", "Show in Database panel menu bar"),
    CONSOLE_MENU("ConsoleMenu", "Show in Console panel menu bar"),
    VCS_LOG_MENU("VcsLogMenu", "Show in VCS Log panel menu bar"),
    CHAT_BOX("ChatBox", "Show in Chat Box"), // 将默认使用 RigthPanel 作为展示位置
    INLINE_CHAT("InlineChat", "Show in Inline Chat"),

    /// external plugins
    EXT_SONARQUBE_MENU("ExtSonarQubeMenu", "Show in SonarQube panel menu bar"),
    ;
}
```

当 COMMIT_MENU 项多于一个时，将会用 PopupMenu 显示；当只有一个时，将直接显示在 Commit 菜单中。

#### ChatBox 示例

ChatBox 是在 Shire ToolWindow 中的一个输入框，用户可以在这里输入内容，然后调用大语言模型。使用事项如下：

- 默认使用 `RigthPanel` 作为展示位置，使用 `ChatBox` 作为 `actionLocation`。
- 当用户创建了 `actionLocation: ChatBox` 的 Shire 代码时，将会读取用户的输入作为提示词的一部分。 

如下是一个自定义 ChatBox 的 Shire 示例：

```shire
---
name: "shire-chat-box"
description: "Shire Chat Box"
interaction: RightPanel
actionLocation: ChatBox
---

根据用户的输入生成 Java 代码

$chatPrompt

```

此时，当用户在 ChatBox 中输入 `create hello world` 时，会将 `hello world` 作为 `chatPrompt` 的值。生成最终的提示词：

```shire
根据用户的输入生成 Java 代码 

create hello world
```

### Inline Chat 示例

Inline Chat 是在当前文件器的行号附近的一个由 Shire Icon 触发的输入框，用户可以在这里输入内容，然后调用大语言模型。使用事项如下：

- 用户需要自行创建一个 Shire 文件，然后将 `actionLocation` 设置为 `InlineChat`。

