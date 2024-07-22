---
title: commit规范
sidebar_position: 2
---



## 一、格式

> 每次commit都包括三个部分：header，body 和 footer。

```shell
<type>(<scope>): <subject>

<body>

<footer>


# 其中，header 是必需的，body 和 footer 可以省略。不管是哪一个部分，任何一行都不得超过72个字符（或100个字符）。这是为了避免自动换行影响美观。
```

## 二、Header

> Header部分只有一行，包括三个字段：`type`（必需）、`scope`（可选）和`subject`（必需）。

### 2.1 type

> 用于说明 commit 的类别，只允许使用下面7个标识。
>
> - feat：新功能（feature）
> - fix：修补bug
> - docs：仅仅修改了文档，如readme.md
> - perf: 更改代码，以提高性能（在不影响代码内部行为的前提下，对程序性能进行优化）
> - style： 仅仅修改了空格、格式缩进、逗号等等，不改变代码逻辑
> - refactor：重构（即不是新增功能，也不是修改bug的代码变动）
> - test：增加测试
> - revert: 恢复上一次提交
> - ci: 持续集成相关文件修改
> - build: 影响项目构建或依赖项修改
> - release: 发布新版本
> - workflow: 工作流相关文件修改
> - chore：构建过程或辅助工具的变动
>
> 如果type为`feat`和`fix`，则该 commit 将肯定出现在 Change log 之中。其他情况（`docs`、`chore`、`style`、`refactor`、`test`）由你决定，要不要放入 Change log，建议是不要

### 2.2 scope

> scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

### 2.3 subject

> `subject`是 commit 目的的简短描述，不超过50个字符。
>
> 其他注意事项：
>
> - 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
> - 第一个字母小写
> - 结尾不加句号（.）

## 三、Body

> Body 部分是对本次 commit 的详细描述，可以分成多行。下面是一个范例。

```shell
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent

"""

    有两个注意点:
    使用第一人称现在时，比如使用change而不是changed或changes。
    永远别忘了第2行是空行
    应该说明代码变动的动机，以及与以前行为的对比。
  
"""
```

## 四、Footer

> Footer 部分只用于以下两种情况

### 4.1 不兼容变动

> 如果当前代码与上一个版本不兼容，则 Footer 部分以BREAKING CHANGE开头，后面是对变动的描述、以及变动理由和迁移方法。

```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

### 4.2 Revert

> 还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以revert:开头，后面跟着被撤销 Commit 的 Header。

```shell
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.


"""

    Body部分的格式是固定的，必须写成`This reverts commit <hash>`.，其中的hash是被撤销 commit 的 SHA 标识符。
    如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的Reverts小标题下面。
  
"""