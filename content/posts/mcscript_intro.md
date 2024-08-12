+++
title = "为 Minecraft 命令设计的语言—— MCScript 介绍 "
date = "2024-08-13"
updated = "2024-08-13"

[taxonomies]
tags=["玩具"]
+++

MCScript 是我最近创造的一门简易的编程语言, 它的目标语言为 Minecraft 命令 (`.mcfunction`). 通过 MCScript, 你可以将使用高级编程语言编写的逻辑轻松地移植到 Minecraft 数据包中. 

MCScript 的编译器是我的第一个初具基本功能的编译器尝试. 虽然它生成的代码效率比较糟糕, 但是它还是能干一些比较有意思的事情的. 下面是几个例子. 

## 示例

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

![生成柱子](https://github.com/Nervonment/mcscript/raw/master/pictures/2024-07-22_14.37.48.png)

- [example/maze.mcs](https://github.com/Nervonment/mcscript/blob/master/example/maze.mcs) 中的代码能够生成一个 45×45 的迷宫: 

![生成迷宫](https://github.com/Nervonment/mcscript/raw/master/pictures/2024-07-24_00.51.23.png)

- [example/snake.mcs](https://github.com/Nervonment/mcscript/blob/master/example/snake.mcs) 中的代码能够生成一个可以玩贪吃蛇游戏的屏幕: 

![贪吃蛇](https://github.com/Nervonment/mcscript/raw/master/pictures/2024-07-28_17.02.57.png)

## 使用方法

详细使用说明参见[此处](https://github.com/Nervonment/mcscript/blob/master/usage.md). 

MCScript 支持输出 Minecraft Java 版 1.21 版本的数据包 (数据包版本 48). 以下命令指定了输入文件为 `my_datapack.mcs`, 输出名为 `my_datapack` 的数据包 (包含输入文件中的函数). 

```sh
mcsc my_datapack.mcs -o my_datapack
```

可以指定多个输入文件: 


```sh
mcsc namespace_1.mcs namespace_2.mcs -o my_datapack
```

此外, 编译器还会输出名为 `mcscript` 的数据包, 其中包含一些通用函数. 

将两个数据包复制到 `.minecraft/saves/<存档名字>/datapacks/` 后, 进入游戏. 首次使用 mcscript 生成的数据包, 需要运行命令 `/function mcscript:init` 进行初始化. 要在游戏中调用 `file_name.mcs` 中名为 `func` 的函数, 输入以下命令即可: 

```
/function file_name:func
```

## 快速入门

参见[此处](https://github.com/Nervonment/mcscript/blob/master/MCScript.md). 

## 运行测试

参见[此处](https://github.com/Nervonment/mcscript/blob/master/test.md). 

## 计划

- 对象和元组. 
- 从实体和方块实体中获取数据的方法. 
- 设计 IR 和优化. 
- 编译时常量. 
- 引用. 