+++
title = "为 Minecraft 命令设计的语言—— MCScript 介绍"
date = "2024-08-13"
updated = "2024-08-13"

[taxonomies]
tags=["玩具", "Minecraft", "MCScript"]
+++

[MCScript](https://github.com/Nervonment/mcscript) 是我最近设计的一门简易的编程语言, 它的目标语言为 Minecraft 命令 (`.mcfunction`). 通过 MCScript, 你可以将使用高级编程语言编写的逻辑轻松地移植到 Minecraft 数据包中. 

MCScript 的编译器是我实现的第一个初具基本功能的编译器. 虽然它功能不多, 存在一些 bug, 生成的代码效率也比较糟糕, 但是它还是能干一些比较有意思的事情的. 下面是几个例子. 

# 示例

- 下面的 MCScript 代码的功能是在自己头顶向上生成一个10格高, 黄色和黑色混凝土交替的柱子: 
```
fn generate_column() {
    let y = 2;
    while y < 12 {
        if y % 2 {
            run_command!("setblock ~ ~{} ~ yellow_concrete", y);
        } else {
            run_command!("setblock ~ ~{} ~ black_concrete", y);
        }
        y += 1;
    }
}
```

![生成柱子](/posts/2024-07-22_14.37.48.png)

- [example/maze.mcs](https://github.com/Nervonment/mcscript/blob/master/example/maze.mcs) 中的代码能够生成一个 45×45 的迷宫: 

![生成迷宫](/posts/2024-07-24_00.51.23.png)

- [example/snake.mcs](https://github.com/Nervonment/mcscript/blob/master/example/snake.mcs) 中的代码能够生成一个可以玩贪吃蛇游戏的屏幕: 

![贪吃蛇](/posts/2024-07-28_17.02.57.png)

# 使用方法

那么我该怎么使用 MCScript 呢?

MCScript 支持输出 Minecraft Java 版 1.21 版本的数据包 (数据包版本 48). 假如你已经编写了一些代码, 源代码文件是 `hello.mcs` `hi.mcs`. 想要将它编译为名为 `my_datapack` 的 Minecraft 数据包, 可以运行以下命令: 

```
mcsc hello.mcs hi.mcs -o my_datapack
```

之后, 编译器会输出两个数据包, 一个名为 `my_datapack`, 包含了你在 `hello.mcs` 和 `hi.mcs` 中编写的函数. 另一个名为 `mcscript`, 包含了运行 MCScript 所生成的数据包所依赖的一些函数. 

接下来, 将两个数据包复制到你的存档文件夹的 `datapack` 目录 (`.minecraft/saves/<存档名字>/datapacks/`) 下, 然后打开游戏, 进入存档. (如果在已经进入了游戏的时候更新了数据包, 需要在游戏内运行命令 `/reload` 重新加载. )

在使用任何 MCScript 生成的数据包中的函数前, 需要先运行一次以下命令来初始化: 

```
/function mscript:init
```

这条命令每个存档只用运行一次即可. *(除非你的代码出现了异常, 导致代表栈的命令存储 `memory:stack frame` 未能复位. 你可以使用 `/data get storage memory:stack frame` 查看栈是否正常, 正常情况下它的值应为 `[]`. )*

假设 `hello.mcs` 的内容如下: 

```
// hello.mcs

fn foo() {
    run_command!("say Hello, world! ");
}
```

在游戏内, 想要运行函数 `foo`, 你需要运行如下命令: 

```
/function hello:foo
```

然后你就可以在聊天栏看到消息 "Hello, world! ". 

注意, 如果你的源代码中定义了全局变量, 想要把全局变量设为你设定的初始值, 需要手动运行一些命令. 例如, 假如 `hi.mcs` 的内容如下: 

```
// hi.mcs

let c: int = 0;

fn bar() {
    c += 1;
    run_command!("say c = {}", c);
}
```

要将全局变量 `c` 的值设置为 `0`, 需要运行如下命令: 

```
/function hi:init
```

然后你可以尝试多次运行 `/function hi:bar`, 便可以看到 `c` 的值依次递增. 

如果你想了解更多内容, 有一篇[快速入门指南](https://github.com/Nervonment/mcscript/blob/master/MCScript.md)可供参考. 

# 实现机制

绝大多数的计算机程序都无可避免地要操作数据. 因此要运行一个程序, 就必须用什么东西来存储和操作数据. Minecraft 提供了两种机制来通过命令操作数据 - [命令存储](https://zh.minecraft.wiki/w/%E5%91%BD%E4%BB%A4%E5%AD%98%E5%82%A8%E5%AD%98%E5%82%A8%E6%A0%BC%E5%BC%8F?variant=zh-cn) 和 [记分板](https://zh.minecraft.wiki/w/%E8%AE%B0%E5%88%86%E6%9D%BF). 在 MCScript 的实现中, 我使用了**命令存储**来模拟内存, **记分板**来模拟寄存器.  

在 Minecraft 中, 可以用下面的命令来存储某个值: 

```
/data modify storage <target> <targetPath> set value <value>
```

其中 `storage <target>` 是**命令存储**的名字, 你可以把它理解成一个作用域或者命名空间; `<targetPath>` 是你要存储的数据在这条**命令存储**中的位置, 你可以把它理解成作用域中的变量; 而 `<value>` 是你要向这个变量存储的值. 例如: 

```
/data modify storage foo:bar var1 set value 114514
```

`<value>` 可以是一个数组甚至对象, 这样一来, 我们就可以模拟栈, 从而实现函数: 

```
# 初始化栈
/data modify storage memory:stack frame set value []

# 调用某个函数时, 压栈
# `append` 用于向数组中添加元素, 这里添加了一个对象, 
# 之后就可以向这个对象里面添加字段作为栈帧的局部变量使用. 
/data modify storage memory:stack frame append value {}

# 将局部变量 `a` 赋值为 `114514`
# `/data` 命令中, 使用 `[-1]` 可以取数组的最后一个元素, 即当前的栈帧. 
/data modify storage memory:stack frame[-1].a set value 114514

# 函数返回, 弹栈
# `remove` 可以从命令存储中删除某一项, 也可以删除数组中的某一个元素或者对象的某一个字段.
# 这里把当前栈帧删除, 从而将栈顶恢复到调用者的栈帧. 
/data remove storage memory:stack frame[-1]
```

看起来命令存储的功能很强大, 但是仅仅用命令存储, 是无法对数据进行运算的. 这时就必须依靠记分板. 

Minecraft 中的记分板有**记分项**和**分数持有者**的概念. 理解这两个概念很简单, 想象某一场考试的结果, 像数学, 物理这样的科目就是记分项, 而你在内的所有的学生就是分数持有者. 在 MCScript 中, 使用不同的记分项并没有意义, 所有需要参与运算的临时数据都存储在记分项 `registers` (寄存器)下, 而分数持有者则对应各个寄存器, 分别命名为 `r0`, `r1`, `r2`, ... 我们首先使用以下命令创建记分项 `registers`: 

```
/scoreboard objectives add registers dummy "寄存器"
# 如果你想实时观察各个寄存器的值, 可以使用下面这条命令
/scoreboard objectives setdisplay sidebar registers
```

可以用以下命令设置某个分数持有者(寄存器)的分数(值):

```
/scoreboard players set r0 registers 114514
# 只能设置整数值
```

要对寄存器中的值进行运算, 可以用以下命令: 

```
/scoreboard players operation <r_dest> registers <op> <r_src> registers
# 上面命令中的 `registers` 就是我们作为寄存器使用的记分项, 实际应用中也可能是其他的记分项
```

其中 `op` 是运算的种类, 有以下几种: 

- `=` 赋值: 将 `r_src` 赋给 `r_dest` .
- `+=` 求和赋值: 将 `r_src` 和 `r_dest` 的和赋给 `r_dest` .
- `-=` 求差赋值: 将 `r_src` 和 `r_dest` 的差赋给 `r_dest` .
- `*=` 求积赋值: 将 `r_src` 和 `r_dest` 的积赋给 `r_dest` .
- `/=` 求商(整数除法)赋值: 将 `r_src` 和 `r_dest` 相除, 取整数除法所得的商赋给 `r_dest`.
- `%=` 求余(取模)赋值: 将 `r_src` 和 `r_dest` 相除, 取整数除法所得的正整数余数赋给 `r_dest` .
- `><` 交换: 交换 `r_src` 和 `r_dest` .
- `<` 取较小值: 仅当 `r_src` 较小时, 将 `r_src` 赋给 `r_dest` .
- `>` 取较大值: 仅当 `r_src` 较大时, 将 `r_src` 赋给 `r_dest` .

我们还需要一种在记分板和命令存储中互相传递数据的方法. 经过我的调查, 只有 `/execute store` 命令能够实现这一点 (说实话, 谁第一时间会想得到竟然是这个命令啊):

```
# 将寄存器 `r0` 的值复制到局部变量 `a` 
/execute store result storage memory:stack frame[-1].a int 1.0 run scoreboard players get r0 registers

# 将局部变量 `a` 的值复制到寄存器 `r0` 
/execute store result score r0 registers run data get storage memory:stack frame[-1].a
```

有了上面的东西, 就已经能够实现一个初具雏形的编程语言. 然而要做到图灵完备, 还需要有跳转机制, 这个就留待下一篇文章再继续叙述了. 

# 写在最后

MCScript 并不是第一个把高级语言编译成 Minecraft 数据包的尝试, 类似的项目还有 [CBScript](https://github.com/SethBling/cbscript) 和 [MCFPP](https://github.com/MinecraftFunctionPlusPlus/MCFPP) 等. ~~(叠甲环节)~~ 我做 MCScript 最初只是为了试着自己写一个编译器玩玩, 结果没想到还真的挺有意思的. MCScript 的编译器 mcsc 是用 Rust 编写的, 然而事实上我并不是很熟悉 Rust (指脱离了 rust-analyzer 和 inlay hint 就没办法写代码), 也没有系统学过编译原理. 因此 mcsc 还有很多实现得不好和没有实现的地方, 也有一些漏洞. 我还是希望能够继续完善 MCScript 的, 只是个人精力和能力有限. 综上所述, 欢迎您[提供指导意见](https://github.com/Nervonment/mcscript/issues/new).