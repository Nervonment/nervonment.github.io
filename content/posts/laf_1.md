+++
title = "Aseprite 的桌面应用程序库 LAF 体验 #1"
date = "2025-03-03"
updated = "2025-03-03"

[taxonomies]
tags=["C++", "LAF"]
+++


[LAF](https://github.com/aseprite/laf) 是著名的像素画绘画软件 [Aseprite](https://www.aseprite.org/) 使用的库，使用的语言是 C++。在此之前，我听说过的 C++ 桌面应用程序库很少，所以今天就来试一试，编译一下所给的例子。我不熟悉 CMake，所以配置过程会记录得比较详细，方便自己以后查阅。

# 安装依赖

- Skia。LAF 和 Aseprite 使用的版本是[这个](https://github.com/aseprite/skia/releases/tag/m102-861e4743af)。我使用的是 Windows 系统，所以下载 [Skia-Windows-Release-x64.zip](https://github.com/aseprite/skia/releases/download/m102-861e4743af/Skia-Windows-Release-x64.zip)，并解压到 `D:\skia` 中。
- Ninja。可以去其[官网](https://ninja-build.org/)下载。Windows 上可以用 scoop 安装：`scoop install ninja`。

# 在 Visual Studio 中配置和编译

克隆 [LAF 的仓库](https://github.com/aseprite/laf)。注意，LAF 的仓库里包含子模块，克隆时要加上 `--recursive` 选项一并克隆：

```
git clone --recursive https://github.com/aseprite/laf
```

在 Windows 上写 C++ 项目，用 Visual Studio 会比较方便。用 Visual Studio 打开 `laf` 文件夹。选择「项目」->「laf 的 CMake 设置」，点击右上角的「编辑 JSON」打开 CMake 配置文件 `CMakeSettings.json`，在 x64-Debug 配置的 `cmakeCommandArgs` 字段中加上下面的内容，以指定 Skia 的位置：

```
-DLAF_BACKEND=skia  -DSKIA_DIR=D:\\skia  -DSKIA_LIBRARY_DIR=D:\\skia\\out\\Release-x64
```

保存 `CMakeSettings.json`，Visual Studio 会自动执行一次 CMake。输出中可以看到下面的信息：

```
[CMake] -- skia dir: D:/skia
[CMake] -- skia library: D:/skia/out/Release-x64/skia.lib
[CMake] -- skia library dir: D:/skia/out/Release-x64
[CMake] -- Configuring done (0.7s)
[CMake] -- Generating done (0.4s)
[CMake] -- Build files have been written to: D:/source/laf/out/build/x64-Debug
已提取 CMake 变量。
已提取源文件和标头。
已提取代码模型。
已提取工具链配置。
已提取包含路径。
CMake 生成完毕。
```

然后在「解决方案资源管理器」的顶部找到「房子图标」右边的「左下角带有 Visual Studio LOGO 的列表图标」，点击它，下面会出现「文件夹视图」和「CMake 目标视图」。默认的视图是「文件夹视图」，现在我们双击「CMake 目标视图」，就可以看到 VS 自动生成的所有 CMake 目标，里面大部分都是测试，今天要编译的是 `laf-examples` 目标。找到它，右键选择「生成 laf-examples」。如果顺利的话，你会看到 `complextextlayout.cpp` 中报了很多错误。我把其中的阿拉伯文删除后，仍然无法解决，只好打开 `examples/CMakeLists.txt`，注释掉 `laf_add_example(complextextlayout GUI)`，放弃编译这个示例。

然后重新尝试生成。接下来迎接我们的是 13624 个 LNK2038 错误，其中大部分说的是「值 0 不匹配值 2」。经查阅发现是在 Debug 模式下使用了 Release 版本的静态库所致。`aseprite/skia` 的 Releases 页面中只提供了 Release 版本的静态库，如果要用 Debug 版本的话，还要自己用源代码编译一遍，今天就先不折腾了。再次打开 CMake 配置文件 `CMakeSettings.json`，把 x64-Debug 配置复制一份，然后把 `name` 字段改成 `x64-Release`，`configurationType` 字段改成 `Release`（等效于手动使用 CMake 时添加 `-DCMAKE_BUILD_TYPE=Release` 选项）。然后在「绿色三角形调试按钮」左边的下拉框中选择 x64-Release。重新尝试生成即可。

# 示例分析

`examples` 中提供了很多示例，包括悬浮窗、文件拖拽甚至着色器等示例，展示了 LAF 比较完善的功能。这次先从最基础的 `hello_laf.cpp` 开始，看看怎么用 LAF 创建一个桌面程序。

可以看到，程序的入口是 `int app_main(int argc, char* argv[])`。（*需要提醒的是，函数签名中的 `argc` 和 `argv` 参数，以及最后的 `return 0` 都是必须的，不要像平常一样因为偷懒省掉。*）上来之后，先 make 一个 `System` 对象 `system`（`System` 类中提供了创建窗口等接口），然后用它设置应用模式为 GUI 模式，创建一个 400 × 300 大小的窗口 `window`。接下来设置标题、鼠标指针（非必须）：

```cpp
os::SystemRef system = os::make_system();
system->setAppMode(os::AppMode::GUI);

os::WindowRef window = system->makeWindow(400, 300);
window->setTitle("Hello World");
window->setCursor(os::NativeCursor::Arrow);
```

然后设置窗口大小变化时的回调函数，直接设置为窗口绘制的函数 `draw_window` 即可。`finishLaunching` 和 `activateApp` 照抄即可，想了解的话可以参考代码注释中给出的说明。

```cpp
system->handleWindowResize = draw_window;
system->finishLaunching();
system->activateApp();
```

接下来是主循环。创建一个 `EventQueue` 对象，然后在循环中等待事件（鼠标、键盘等）发生。注意在循环中使用的是 `EventQueue` 的 `getEvent` 方法，这个方法会一直等待到有事件发生，也就是说如果你什么也不做的话，之后的代码就不会执行。与之相对的是 `queueEvent` 方法，如果没有事情发生的话，就会得到一个 `Event::None` 类型的事件。*（至于注释里面提到的 `true` 参数，我反正是没有看到有什么 `true` 参数，可能是改了代码忘记改注释了吧 (;_:)，还真是如 `README.md` 中所说「API 尚未稳定」。）*

这个示例使用的是懒惰更新策略，在主循环中维护了一个 `redraw` 变量，只在需要更新窗口内容时，将这个变量设置为 `true`，重新绘制一次窗口。

然后我们来看窗口绘制函数 `draw_window`。首先获取窗口的 `surface` 对象，接下来的绘制都是在这个 `surface` 上，用一个 `Paint` 对象进行。注意，和平常画画一样，绘制之前需要先把画板（窗口）清理干净：

```cpp
p.color(gfx::rgba(0, 0, 0));
p.style(os::Paint::Fill);
surface->drawRect(rc, p);
```

否则窗口内容更新时，重新绘制的内容会叠在旧内容的上面。

绘制的操作都很容易理解，这里说一下 `draw_text` 函数，要绘制中文字符的话，不能直接传中文字符串，而是要先转换成 UTF-8 编码：

```cpp
os::draw_text(surface, nullptr, base::to_utf8(L"你好世界"), gfx::Point(10, 20), &p, os::TextAlign::Left);
```

最后这三句也是先照抄即可：

```cpp
if (window->isVisible())
    window->invalidateRegion(gfx::Region(rc));
else
    window->setVisible(true);
window->swapBuffers();
```