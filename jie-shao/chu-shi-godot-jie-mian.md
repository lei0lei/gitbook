# 初识 Godot 界面

### 项目管理器

启动 Godot 后，你首先会看到项目管理器窗口。在默认选项卡**项目**中，你可以管理已有项目、导入或创建新项目等等。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_project_manager.webp" alt=""><figcaption></figcaption></figure>

在窗口顶部，还有另一个名为 **资产库** 的选项卡。第一次进入该选项卡时，你会看到一个“连接网络”按钮。出于隐私原因，Godot 项目管理器默认不访问互联网。要更改该设置，请点击“连接网络”按钮。你也可以稍后在设置中更改此选项。

一旦你的网络模式被设置成“在线”，你就可以在开源资产库中搜索演示项目，资产库中包括了许多社区开发的项目：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_project_templates.webp" alt=""><figcaption></figcaption></figure>

项目管理器设置可以通过 **设置** 菜单打开：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_settings.webp" alt=""><figcaption></figcaption></figure>

在这里，你可以更改编辑器的语言（默认是系统语言）、界面主题、显示缩放、网络模式，以及目录命名规则。

### 初识 Godot 编辑器

打开新建项目或者已有项目，就会出现编辑器界面。我们来看看它的主要区域：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_editor_empty.webp" alt=""><figcaption></figcaption></figure>

默认情况下，窗口的顶部左侧为**主菜单**，中间为**工作区**切换按钮（当前工作区会高亮显示），右侧为**游戏测试**按钮和**电影制作模式**切换按钮：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_top_menus.webp" alt=""><figcaption></figcaption></figure>

在工作区按钮下方，可以看到打开的 scenes 标签。紧挨标签的加号 (+) 按钮将向项目中添加一个新场景。最右侧的按钮可以切换无干扰模式，通过隐藏界面中的 **面板** 来最大化或恢复 **视口** 的大小：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_scene_selector.webp" alt=""><figcaption></figcaption></figure>

在中间位置，场景选择器下方是**视口**，它的**工具栏**位于顶部，里面包含了不同的工具，用于移动、缩放或锁定场景中的节点（当前激活的是 3D 工作区）：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_3d_viewport.webp" alt=""><figcaption></figcaption></figure>

工具栏会随着上下文和所选节点改变。这里展示的是 2D 工具栏：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_toolbar_2d.webp" alt=""><figcaption></figcaption></figure>

下面这个是 3D 的：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_toolbar_3d.webp" alt=""><figcaption></figcaption></figure>

视口的两边是**停靠面板**。窗口底部则是**底部面板**。

我们来看看停靠面板。**文件系统**面板会列出项目中的文件，包括脚本、图片、音频采样等：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_filesystem_dock.webp" alt=""><figcaption></figcaption></figure>

**场景**面板会列出活动场景中的节点：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_scene_dock.webp" alt=""><figcaption></figcaption></figure>

你可以在**检查器**中编辑所选节点的属性：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_inspector_dock.webp" alt=""><figcaption></figcaption></figure>

视口底部的**底部面板**中包含了调试控制台、动画编辑器、混音器等。它们所占的空间非常宝贵，所以默认都是折叠状态：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_bottom_panels.webp" alt=""><figcaption></figcaption></figure>

点击某一个就会在垂直方向展开。下面展示的是打开的动画编辑器：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_bottom_panel_animation.webp" alt=""><figcaption></figcaption></figure>

底部面板也可以通过在 **编辑器设置 > 快捷键** 中定义的快捷键进行显示或隐藏，位于 **底部面板** 类别下。

### 五个主屏幕

编辑器顶部的中央有五个主屏幕按钮：2D、3D、Script、Game、AssetLib。

**2D 屏幕**可以用于任何类型的游戏。除了 2D 游戏，2D 屏幕也会用于界面的构建。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_workspace_2d.webp" alt=""><figcaption></figcaption></figure>

在**3D 屏幕**中，你可以操作网格、灯光、设计 3D 游戏的关卡。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_workspace_3d.webp" alt=""><figcaption></figcaption></figure>

**游戏屏幕** 是在从编辑器运行项目时，项目显示的地方。你可以通过项目进行测试，并在实时中暂停和调整它。请注意，这仅用于测试调整的效果，游戏停止运行时，在此处所做的任何更改不会被保存。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_workspace_game.webp" alt=""><figcaption></figcaption></figure>

**Script 屏幕**是一个完整的代码编辑器，包含调试器、丰富的自动补全、以及内置的代码参考手册。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_workspace_script.webp" alt=""><figcaption></figcaption></figure>

最后，**AssetLib** 是插件、脚本、资产的仓库，这些内容是自由开源的，可以在你的项目中使用。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_workspace_assetlib.webp" alt=""><figcaption></figcaption></figure>

### 内置类参考手册

Godot 自带内置的类参考手册。

要搜索类、方法、属性、常量、信号相关的信息，可以使用以下任意方法：

* 在编辑器中的任意位置按下 <kbd>F1</kbd>（或在 macOS 上按下 <kbd>Opt + Space</kbd>，或在带有 <kbd>Fn</kbd> 键的笔记本电脑上按下 <kbd>Fn + F1</kbd>）。
* 点击 Script 主屏幕右上角的“搜索帮助”按钮。
* 点击“帮助”菜单的“搜索帮助”。
* 在脚本编辑器中 <kbd>Ctrl + 点击</kbd>（macOS 上为 <kbd>Cmd + 点击</kbd>）类名、函数名、内置变量。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_search_help_button.webp" alt=""><figcaption></figcaption></figure>

执行其中的任意操作都会弹出一个窗口。通过输入进行搜索。你也可以用它来查看所有对象和方法。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_search_help.webp" alt=""><figcaption></figcaption></figure>

在条目上双击就会在脚本主屏幕中打开对应的页面。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/editor_intro_help_class_animated_sprite.webp" alt=""><figcaption></figcaption></figure>

替代方案，

* 在脚本编辑器中按下 <kbd>Ctrl</kbd>键（在 macOS 上为 <kbd>Cmd</kbd>键）时点击类名、函数名、内置变量。
* 右键点击节点并选择**打开文档**，或者在脚本编辑器中选择**查找符号**，都会直接打开对应的文档。
