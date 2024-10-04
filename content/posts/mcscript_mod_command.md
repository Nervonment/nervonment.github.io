+++
title = "在 MCScript 中调用模组命令"
date = "2024-10-05"
updated = "2024-10-05"

[taxonomies]
tags=["玩具", "Minecraft", "MCScript"]
+++

最近做了个[粘土人](https://www.bilibili.com/video/BV1wt4AeiENk), 想剪一个逐层切片教程, 要展示建筑的每一个横截面, 而且要有平滑石头这种有边框的方块作为背景参考. 突发奇想, 想试试能不能通过 MCScript 来调用 WorldEdit 的命令来实现. 

首先验证一下选区和填充方块能不能实现: 

```
let X1: int = -2;
let Z1: int = -2;
let X2: int = 2;
let Z2: int = 2;

fn main() {
    // 设置选区的起点和终点
    run_command!("/pos1 {},0,{}", X1, Z1);
    run_command!("/pos2 {},0,{}", X2, Z2);
    // 用石头填充选区
    run_command!("/set stone");
}
```

编译运行, 发现选区(前两条命令)能够正常选取, 但是到第三条命令就没有反应了.

原理上, MCScript 编译器会为每一个 run_command! 调用生成一个 mcfunction 函数文件. 例如上面的三条命令会被分别生成在 `custom_cmd_0.mcfunction`, `custom_cmd_1.mcfunction` 和 `custom_cmd_2.mcfunction` 中. 检查其中后两个文件, 内容如下:

```
// custom_cmd_1.mcfunction

$/pos2 $(0),0,$(1)
```

```
// custom_cmd_2.mcfunction

/set stone
```

有的玩家可能没有见过 `$` 符号. 这里解释一下, 这是 Java 版 1.20.2 中加入的新功能 - [宏](https://zh.minecraft.wiki/w/Java%E7%89%88%E5%87%BD%E6%95%B0). 从此版本起, mcfunction 函数中的命令可以以 `$` 开头, 其后必须包含一个或以上的宏参数 `$(<parameter>)`. 运行该函数时, 将把命令中的 `$(<parameter>)` 部分直接替换为相应参数 `<parameter>` 的值的字符串形式. 例如, 可以通过下面的方式调用 `custom_cmd_1.mcfunction`: 

```
function my_namespace:custom_cmd_1 {0: 114, 1: 514}
```

`custom_cmd_1.mcfunction` 中的命令将会被解析为

```
/pos2 114,0,514
```

Minecraft 在运行 mcfunction 函数前, 会先判断函数中的每一条命令是否是一条正确的 Minecraft 命令 (不包括模组命令). 也就是说, 如果在函数中直接加入一条类似于 `/set stone` 的模组命令, Minecraft 是会拒绝运行的. 这也就是上面的 `custom_cmd_2.mcfunction` 没有成功运行的原因. 可是为什么前两个带有宏的函数能够成功运行呢? 因为在运行带有宏的命令前, Minecraft 无法提前判断这条命令是否合法, 只能假设他是合法的, 并尝试解析和执行. 前两个函数正是因为带有宏, 所以绕过了游戏的检查, 得以成功执行. 

这么一来, 想要通过函数运行 `/set stone` 这样的模组命令, 只要在命令的末尾加上一个宏参数, 并在运行的时候给这个参数传入一个空字符串即可: 

```
// custom_cmd_2.mcfunction
$/set stone$(empty_str)
```

```
// 调用方式
function my_namespace:custom_cmd_2 {empty_str: ""}
```

于是我在 MCScript 中加入了 `run_mod_command!`. 有别于 `run_command!`, 在命令中不含任何参数时, `run_mod_command!` 会在命令的末尾自动加上一个 `$(empty_str)` 参数, 以实现对模组命令的调用. 

下面就是切片展示工具的完整代码: 

```
let X1: int = -15;
let Z1: int = -15;
let X2: int = 15;
let Z2: int = 15;

let Y1: int = 14;
let Y2: int = -12;

let y: int = Y1;

// 每调用一次此函数, 展示下一层横截面
fn slice_step() {
    if y < Y2 {
        run_mod_command!("/undo");
        return;
    }
    run_command!("say current y: {}", y - 1);

    if y != Y1 {
        run_mod_command!("/undo");
    }
    run_mod_command!("/pos1 {},{},{}", X1, y, Z1);
    run_mod_command!("/pos2 {},{},{}", X2, y, Z2);
    run_mod_command!("/set air");
    run_mod_command!("/pos1 {},{},{}", X1, y - 2, Z1);
    run_mod_command!("/pos2 {},{},{}", X2, y - 2, Z2);
    run_mod_command!("/set smooth_stone");
    run_command!("tp @s {} {} {} 0 90", (X1 + X2) / 2, y + 20, (Z1 + Z2) / 2);
    y -= 1;
}
```

可以看到[效果](https://www.bilibili.com/video/BV1wt4AeiENk)还是挺不错的. 最主要的是, 它真的很快! 只要修改起点和终点坐标, 一分钟之内就能得到建筑的各层切片截图. ~~拖更理由减一~~