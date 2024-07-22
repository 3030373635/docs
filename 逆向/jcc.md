`jcc` 指的是一组条件跳转指令。这些指令根据特定的条件进行跳转，而条件通常由之前的指令（例如比较指令）设置的标志决定。在 x86 汇编中，这些指令通常以 `J` 开头，后跟两个字母，表示具体的条件。

常见的 `jcc` 指令

以下是一些常见的条件跳转指令及其含义：

# JE 或 JZ

> Jump if Equal / Jump if Zero

- **条件**：如果相等或零标志被设置
- **标志位**：ZF（Zero Flag）
- **说明**：当 `ZF = 1` 时跳转

# JNE 或 JNZ

> Jump if Not Equal / Jump if Not Zero)

- **条件**：如果不相等或零标志未被设置
- **标志位**：ZF（Zero Flag）
- **说明**：当 `ZF = 0` 时跳转

# JS 

> Jump if Sign

- **条件**：如果符号标志被设置
- **标志位**：SF（Sign Flag）
- **说明**：当 `SF = 1` 时跳转



**`JNS` (Jump if Not Sign)**：

- **条件**：如果符号标志未被设置
- **标志位**：SF（Sign Flag）
- **说明**：当 `SF = 0` 时跳转



**`JP` 或 `JPE` (Jump if Parity / Jump if Parity Even)**：

- **条件**：如果奇偶校验标志被设置（偶数个1）
- **标志位**：PF（Parity Flag）
- **说明**：当 `PF = 1` 时跳转



**`JNP` 或 `JPO` (Jump if Not Parity / Jump if Parity Odd)**：

- **条件**：如果奇偶校验标志未被设置（奇数个1）
- **标志位**：PF（Parity Flag）
- **说明**：当 `PF = 0` 时跳转



**`JO` (Jump if Overflow)**：

- **条件**：如果溢出标志被设置
- **标志位**：OF（Overflow Flag）
- **说明**：当 `OF = 1` 时跳转



**`JNO` (Jump if Not Overflow)**：

- **条件**：如果溢出标志未被设置
- **标志位**：OF（Overflow Flag）
- **说明**：当 `OF = 0` 时跳转



**`JB` 或 `JNAE` (Jump if Below / Jump if Not Above or Equal)**：

- **条件**：如果低于（无符号比较）
- **标志位**：CF（Carry Flag）
- **说明**：当 `CF = 1` 时跳转



**`JAE` 或 `JNB` (Jump if Above or Equal / Jump if Not Below)**：

- **条件**：如果高于或等于（无符号比较）
- **标志位**：CF（Carry Flag）
- **说明**：当 `CF = 0` 时跳转



**`JBE` 或 `JNA` (Jump if Below or Equal / Jump if Not Above)**：

- **条件**：如果低于或等于（无符号比较）
- **标志位**：CF（Carry Flag），ZF（Zero Flag）
- **说明**：当 `CF = 1` 或 `ZF = 1` 时跳转



# JNBE 或 JA

> Jump if Not Below or Equal / Jump if Above

- **条件**：如果无符号比较大于
- **标志位**：CF（Carry Flag），ZF（Zero Flag）
- **说明**：当 `CF = 0` 且 `ZF = 0` 时跳转

在无符号比较中，`JNBE` 或 `JA` 指令用于跳转到指定的标签，如果目标值大于源值（即没有进位且比较结果不为零）。



**`JNGE` 或 `JL` (Jump if Not Greater or Equal / Jump if Less)**：

- **条件**：如果有符号比较小于
- **标志位**：SF（Sign Flag），OF（Overflow Flag）
- **说明**：当 `SF ≠ OF` 时跳转

在有符号比较中，`JNGE` 或 `JL` 指令用于跳转到指定的标签，如果目标值小于源值（即符号标志和溢出标志不相等）。



`JLE` 或 `JNG` (Jump if Less or Equal / Jump if Not Greater)

- **条件**：如果有符号比较小于或等于
- **标志位**：ZF（Zero Flag），SF（Sign Flag），OF（Overflow Flag）
- **说明**：当 `ZF = 1` 或 `SF ≠ OF` 时跳转

在有符号比较中，`JLE` 或 `JNG` 指令用于跳转到指定的标签，如果目标值小于或等于源值（即零标志被设置，或符号标志和溢出标志不相等）。



# JNLE 或 JG

> Jump if Not Less or Equal / Jump if Greater

- **条件**：如果有符号比较大于
- **标志位**：ZF（Zero Flag），SF（Sign Flag），OF（Overflow Flag）
- **说明**：当 `ZF = 0` 且 `SF = OF` 时跳转

在有符号比较中，`JNLE` 或 `JG` 指令用于跳转到指定的标签，如果目标值大于源值（即零标志未设置，并且符号标志和溢出标志相等）。