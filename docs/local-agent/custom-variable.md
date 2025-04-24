---
layout: default
title: Custom Variable
parent: Shire Language
nav_order: 5
---

Shire 自定义变量的核心是 Variable -> Pattern-Action 模型。在 Shire 中，Pattern-Action 模型是一种用于处理数据的模式匹配和动作执行模型。
如下图所示：

![](/img/shire-pattern-action.svg)

- variable 是用户自定义的变量。
- pattern 是用于筛选输入数据的规则或标准，可以是一组文件名模式、正则表达式，用于识别哪些数据需要进一步处理。
- action 是当数据符合 pattern 时需要执行的任务，由一系列命令组成，描述了如何处理匹配的数据。
    - function name 是 Shire 内置的函数，用于处理数据。
    - arguments 是函数的参数，用于传递数据。
    - pipe(`|`) 是用于连接多个操作的管道操作符，每个操作都是一个函数，其输出作为下一个函数的输入。

Custom Variable 的三种实现方式：

```shire
---
variables:
  "var1": "demo" // Value
  "var2": /.*.java/ { find("error.log") | sort | xargs("rm")} // Pattern-Action
  "var3": /.*.log/ {
    case "$0" {
      "error" { find("ERROR") | sort | xargs("notify_admin") }
      "warn" { find("WARN") | sort | xargs("notify_admin") }
      "info" { find("INFO") | sort | xargs("notify_user") }
      default  { find("ERROR") | sort | xargs("notify_admin") }
    }
  }
---
```

多变量示例：

```shire
---
name: "类图分析"
variables:
  "controllers": /.*.java/ { cat }
  "tokens": /any/ { tokenizer($controllers, "word") }
  "chinese": /any/ { tokenizer("孩子上了幼儿园 安全防拐教育要做好", "jieba") }
---

$controllers
```

在这个示例中，我们在 tokens 变量中使用了 controllers 变量的值，这样就可以在变量之间传递数据。

## Shire 常规自定义变量

常规自定义变量，可以用于：

- 对应不同类型文件，自定义 prompt。

## Variable Pattern Action

在Shire中，我们借鉴了 Unix/Linux 的设计理念和 Shell 编程模式，特别是 Pattern-Action 模型。该模型通过定义模式和动作来处理数据：

1. **模式（Pattern）**：这代表用于筛选输入数据的规则或标准。在Unix/Linux中，这可以是一组文件名模式、正则表达式或其他条件，用于识别哪些数据需要进一步处理。
2. **动作（Action）**：这是当数据符合模式时需要执行的任务。它由一系列命令组成，描述了如何处理匹配的数据。

注意：如果 pattern 为 `any` 或者 `null`，表示不进行筛选，直接执行动作。

例如，在Shire中，我们可以这样定义一个 Pattern-Action：

```text
/.*.java/ { find("error.log") | print }
```

这里，`/*.java/` 是模式部分，用于匹配所有以 `.java` 结尾的文件，而 `{ find("error.log") | print }` 是动作部分，
表示对匹配的文件执行一系列操作：首先搜索包含 "error.log" 的行，然后对这些行进行排序，最后将结果输出到标准输出。

在 Shire 中，我们利用了 Intellij 的强大功能，如正则表达式匹配、代码高亮和语法检查，以帮助用户更高效地编写代码。例如，
使用正则表达式 `.*.java` 可以轻松地匹配所有 Java 源文件。

明白了！让我为您完整优化一下文档，包括对示例的详细解释：

### 示例 1：Pattern-Action Pipeline

```shire
---
variables:
  "var2": /.*.java/ { cat | find("error.log") | sort | cat }
  "extContext": /build\.gradle\.kts/ { cat | find("org.springframework.boot:spring-boot-starter-jdbc") | print("This project use Spring Framework") }
---
```

在这个示例中：

- **`var2` 变量**：匹配所有以 `.java` 结尾的文件。动作部分使用了管道操作符 `|`，依次执行了 `find("error.log")`、`sort`
  ，然后再次使用 `cat` 输出结果。

- **`extContext` 变量**：匹配所有名为 `build.gradle.kts`
  的文件。动作部分执行了 `find("org.springframework.boot:spring-boot-starter-jdbc")`，并输出一条指示该项目使用 Spring
  Framework 的信息。

### 示例 2：Pattern-Action 多 CASE

```shire
---
variables:
  "testTemplate": /\(.*\).java/ {
    case "$1" {
      "Controller" { cat(".shire/templates/ControllerTest.java") }
      "Service" { cat(".shire/templates/ServiceTest.java") }
      default  { cat(".shire/templates/DefaultTest.java") }
    }
  }
---
```

在这个示例中：

- **`testTemplate` 变量**：匹配所有以 `(.*)` 开头、`.java` 结尾的文件。根据不同的匹配结果执行不同的动作。
    - 如果匹配到 `Controller`，则输出 `ControllerTest.java` 的内容。
    - 如果匹配到 `Service`，则输出 `ServiceTest.java` 的内容。
    - 如果没有匹配到上述任何值（`default`），则输出 `DefaultTest.java` 的内容。

### 示例 3：变量二次处理

用户自定义变量可以对 Shire 自带变量进行二次处理，例如：

```shire
---
name: "添加测试"
actionLocation: ContextMenu
variables:
  "sourceCode": /any/ { print($filePath) | sed("src\/test\/", "src/main/") | sed("Test.java", ".java") | cat }
onStreamingEnd: { parseCode | patch($filePath, $output) }
---
```

## Pattern Function

| 函数类别      | 功能描述             | 参数                                                                          | 示例                                          |
|-----------|------------------|-----------------------------------------------------------------------------|---------------------------------------------|
| find      | 基于文本搜索           | `text`: 要搜索的文本                                                              | `find("error")`                             |
| grep      | 使用模式进行搜索         | `patterns`: 要搜索的模式                                                          | `grep("[a-zA-Z]+Controller")`               |
| sed       | 查找和替换操作          | `pattern`: 要查找的模式<br />`replacements`: 替换的字符串<br />`isRegex`: 是否为正则表达式      | `sed("s/old/new/g")`                        |
| sort      | 排序操作             | `arguments`: 排序所需的参数                                                        | `sort`                                      |
| uniq      | 去除重复行            | `texts`: 要处理的文本                                                             | `uniq("line1", "line2", "line1")`           |
| head      | 获取文件的前几行         | `number`: 要获取的行数                                                            | `head(10)`                                  |
| tail      | 获取文件的末尾几行        | `number`: 要获取的行数                                                            | `tail(5)`                                   |
| xargs     | 处理变量             | `variables`: 要处理的变量                                                         | `xargs("arg1", "arg2")`                     |
| print     | 打印文本             | `texts`: 要打印的文本                                                             | `print("Hello", "World")`                   |
| cat       | 连接文件             | `paths`: 要连接的文件路径                                                           | `cat("file1.txt", "file2.txt")`             |
| notify    | 使用 IDE 通知        | `message`: 要显示的通知消息                                                         | `notify("Process completed successfully.")` |
| redact    | 屏蔽敏感数据           |                                                                             | `redact()`                                  |
| jsonpath  | 使用 JsonPath 选择数据 | `jsonPath`: JsonPath 表达式,其中 jsonString 为可选                                  | `jsonpath(jsonString, "$.store.*")`         |
| batch     | 批处理操作            | `fileName`: Shire  文件名，: paths`: 要处理的文件路径                                   | `batch("file1.shire", "file2.txt")`         |
| tokenizer | 分词操作             | `text`: 待分词的文本, type: 分词类型（`word`, `naming`, `stopwords`，`jieba`，默认 `word`） | `tokenizer("text") `                        |
| lineNo    | 行号操作             | `text`: 待处理的文本                                                              | `lineNo("text")`                            |

编排函数

| 函数类别           | 功能描述                       | 参数                 | 示例                               |
|----------------|----------------------------|--------------------|----------------------------------|
| execute        | 异步执行 Shire 脚本、GradleTask 等 | `string`: 要执行的脚本内容 | `execute("next-script.shire")`   |
| thread         | 线程执行（异步）                   | `path`: 要执行的脚本路径   | `thread("script.shire")`         |
| approveExecute | 等待确认再执行                    | `path`: 要执行的脚本路径   | `approveExecute("script.shire")` |

Execute

- Shire 脚本执行: `execute("next-script.shire")`
- GradleTask 执行: `execute(":bootRun")`
- 文件执行: `execute("next-script.sh")`

