# 创建第一个脚本

在本课中，你将会用 GDScript 编写第一个脚本，使 Godot 图标转圈。正如我们在[介绍](https://docs.godotengine.org/zh-cn/4.x/getting_started/introduction/introduction_to_godot.html#doc-introduction-learning-programming)中提到的，我们假设你有编程基础。

本教程使用GDScript编写，每个代码块均附带独立标签页展示等效的C#实现以便参考。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_rotating_godot.gif" alt=""><figcaption></figcaption></figure>

### 项目设置

请从头开始[创建一个新项目](https://docs.godotengine.org/zh-cn/4.x/tutorials/editor/project_manager.html#doc-creating-and-importing-projects)。你的项目应该包含一张图片：Godot 图标，我们经常在社区中使用它来制作原型。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_icon.svg" alt=""><figcaption></figcaption></figure>

我们需要创建一个 Sprite2D 节点来在游戏中显示它。在 Scene 停靠面板中，点击 Other Node 按钮。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_click_other_node.webp" alt=""><figcaption></figcaption></figure>

在搜索栏中输入“Sprite2D”来过滤节点，双击 Sprite2D 来创建节点。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_add_sprite_node.webp" alt=""><figcaption></figcaption></figure>

你的 Scene 选项卡现在应该只包含一个 Sprite2D 节点。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_scene_tree.webp" alt=""><figcaption></figcaption></figure>

Sprite2D 节点需要一个贴图来显示。在右侧 Inspector 中，可以看到 Texture 属性写着 `<空>`。要显示 Godot 图标，请从“文件系统”停靠面板，将 `icon.svg` 文件拖放到 Texture 槽位上。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_setting_texture.webp" alt=""><figcaption></figcaption></figure>

然后，点击并拖动视口中的图标，使其在游戏视图中居中。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_centering_sprite.webp" alt=""><figcaption></figcaption></figure>

### 新建脚本

右键点击场景面板中的 Sprite2D，选择 Attach Script，以创建新的脚本并附加到我们的节点。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_attach_script.webp" alt=""><figcaption></figcaption></figure>

将弹出 Attach Node Script 窗口。你可以在此选择脚本的语言、设置文件路径及其他选项。

把 Template 从 `Node: Default` 改为 `Object: Empty` 从而得到一个干净的脚本文件。其他选项保持默认，然后点击 Create 按钮来创建脚本。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_attach_node_script.webp" alt=""><figcaption></figcaption></figure>

此时 Script 工作区将自动打开并显示新建的 `sprite_2d.gd` 文件，包含以下代码：

```gdscript
extends Sprite2D
```

每个 GDScript 文件都是一个隐含的类。`extends` 关键字定义了这个脚本所继承或扩展的类。本例中为 `Sprite2D`，意味着我们的脚本将获得 Sprite2D 节点的所有属性和方法，包括它继承的 `Node2D`、`CanvasItem`、`Node` 等类。

{% hint style="info" %}
在 GDScript 中，如果你略写了带有 `extends` 关键字的一行，那么你的类将隐式扩展自 [RefCounted](https://docs.godotengine.org/zh-cn/4.x/classes/class_refcounted.html#class-refcounted)，Godot 使用这个类来管理你的应用程序的内存。
{% endhint %}

继承的属性包括你在 Inspector 停靠面板中能看到的属性，例如我们这个节点的 `texture`。

{% hint style="info" %}
默认情况下，Inspector 会以“标题式大小写”（Title Case）显示节点的属性名称，即各个单词首字母大写，并以空格分隔。而在 GDScript 代码中，这些属性使用“蛇形命名法”（snake\_case），即全部小写，并以下划线分隔。

你可以将鼠标悬停在 Inspector 中任何属性的名称上，查看其描述和在代码中的标识符。
{% endhint %}

### 你好，世界！

我们的脚本目前没有做任何事情。让我们开始打印文本“Hello, world!”到底部输出面板。

往脚本中添加以下代码：

```gdscript
func _init():
	print("Hello, world!")
```

让我们把它分解一下。`func` 关键字定义了一个名为 `_init` 的新函数。这是类构造函数的一个特殊名称。如果你定义了这个函数，引擎会在内存中创建每个对象或节点时调用 `_init()`。

{% hint style="info" %}
GDScript 是基于缩进的语言。行首的制表符是 `print()` 代码正常工作的必要条件。如果你省略了这个制表符，或者没有正确缩进一行，编辑器则将以红色标注高亮显示以下错误信息：“Indented block expected”（应有缩进块）。
{% endhint %}

如果你还没有保存场景，请将其保存为 `sprite_2d.tscn`，然后按 <kbd>F6</kbd>（macOS 上为 <kbd>Cmd + R</kbd>）来运行它。看一下底部展开的输出面板，它应该显示“Hello, world!”。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_print_hello_world.webp" alt=""><figcaption></figcaption></figure>

将 `_init()` 函数删除，这样你就只有一行 `extends Sprite2D` 了。

### 四处旋转

是时候让节点移动并旋转了。为此，我们要向脚本添加两个成员变量：移动速度（像素/秒）和角速度（弧度/秒）。请在 `extends Sprite2D` 后面另起一行，添加如下代码。

```gdscript
var speed = 400
var angular_speed = PI
```

成员变量位于脚本的顶部，在“extends”之后、函数之前。附加了此脚本的每个节点实例都将具有自己的 `speed` 和 `angular_speed` 属性副本。

为了移动我们的图标，我们需要在游戏循环中每一帧更新其位置和旋转。我们可以使用 `Node` 类中的虚函数 `_process()`。如果你在任何扩展自 Node 类的类中定义它，如 Sprite2D，Godot将在每一帧调用该函数，并传递给它一个名为 `delta` 的参数，即从上一帧开始经过的时间。

{% hint style="info" %}
游戏的工作方式是每秒钟渲染许多图像，每幅图像称为一帧，而且是循环进行的。我们用每秒帧数（FPS）来衡量一个游戏产生图像的速度。大多数游戏的目标是60FPS，尽管你可能会发现在较慢的移动设备上的数字是30FPS，或者是虚拟现实游戏的90至240。

引擎和游戏开发者尽最大努力以恒定的时间间隔更新游戏世界和渲染图像，但在帧的渲染时间上总是存在着微小的变化。这就是为什么引擎为我们提供了这个delta时间值，使我们的运动与我们的帧速率无关。
{% endhint %}

在脚本的底部，定义该函数：

```gdscript
func _process(delta):
	rotation += angular_speed * delta
```

`func` 关键字定义了一个新函数。在它之后，我们必须在括号里写上函数的名称和它所接受的参数。冒号结束定义，后面的缩进块是函数的内容或指令。

{% hint style="info" %}
请注意 `_process()` 和 `_init()` 一样都是以下划线开头的。按照约定，这是 Godot 的虚函数，也就是你可以覆盖的与引擎通信的内置函数。
{% endhint %}

函数内部的那一行 `rotation += angular_speed * delta` 每一帧都会增加我们的精灵的旋转量。这里 `rotation` 是从 `Sprite2D` 所扩展的 `Node2D` 类继承的属性。它可以控制我们节点的旋转，以弧度为单位。

{% hint style="info" %}
在代码编辑器中，你可以按住 <kbd>Ctrl</kbd> 键（在macOS上使用 <kbd>Command</kbd> 键） 单击任何内置的属性或函数，如 `position`、`rotation`、`_process` 以在新标签页中打开相应的文档。
{% endhint %}

运行该场景，可以看到 Godot 的图标在原地转动。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_godot_turning_in_place.gif" alt=""><figcaption></figcaption></figure>

#### 前进

现在我们来让节点移动。在 `_process()` 函数中添加下面两行代码，确保每一行都和之前的 `rotation += angular_speed * delta` 行的缩进保持一致。

```gdscript
var velocity = Vector2.UP.rotated(rotation) * speed

position += velocity * delta
```

正如我们所看到的，`var` 关键字可以定义新变量。如果你把它放在脚本顶部，定义的就是类的属性。在函数内部，定义的则是局部变量：只在函数的作用域中存在。

我们定义一个名为 `velocity` 的局部变量，该变量是用于表示方向和速度的 2D 向量。要让节点向前移动，我们可以从 Vector2 类的常量 `Vector2.UP` 入手，这个向量指向上方，调用 Vector2 的 `rotated()` 方法可以将其进行旋转。表达式 `Vector2.UP.rotated(rotation)` 表示的是指向图标前方的向量。用这个方向与我们的 `speed` 属性相乘后，得到的就是用来移动节点的速度。

我们在节点的 `position` 里加上 `velocity * delta` 来实现移动。位置本身是 [Vector2](https://docs.godotengine.org/zh-cn/4.x/classes/class_vector2.html#class-vector2) 类型的，是 Godot 用于表示 2D 向量的内置类型。

运行场景就可以看到 Godot 头像在绕圈圈。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scripting_first_script_rotating_godot.gif" alt=""><figcaption></figcaption></figure>

我们的节点目前是自行移动的。在下一部分 [监听玩家的输入](https://docs.godotengine.org/zh-cn/4.x/getting_started/step_by_step/scripting_player_input.html#doc-scripting-player-input) 中，我们会让玩家的输入来控制它。

### 完整脚本

这是完整的 `sprite_2d.gd` 文件，仅供参考。

```gdscript
extends Sprite2D

var speed = 400
var angular_speed = PI


func _process(delta):
	rotation += angular_speed * delta

	var velocity = Vector2.UP.rotated(rotation) * speed

	position += velocity * delta
```



