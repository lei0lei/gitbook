# 使用信号

在本课中，我们将介绍信号。它们是节点在发生特定事件时发出的消息，例如按下按钮。其他节点可以连接到该信号，并在事件发生时调用函数。

信号是 Godot 内置的委派机制，允许一个游戏对象对另一个游戏对象的变化做出反应，而无需相互引用。使用信号可以限制[耦合](https://zh.wikipedia.org/zh-cn/%E8%80%A6%E5%90%88%E6%80%A7_\(%E8%A8%88%E7%AE%97%E6%A9%9F%E7%A7%91%E5%AD%B8\))，并保持代码的灵活性。

例如，你可能在屏幕上有一个代表玩家生命值的生命条。当玩家受到伤害或使用治疗药水时，你希望生命条反映变化。要做到这一点，在 Godot 中，你会使用到信号。

从 Godot 4.0 开始，信号和方法（[Callable](https://docs.godotengine.org/zh-cn/4.x/classes/class_callable.html#class-callable)）一样，都成为了一等类型。这意味着你可以直接把信号当作方法的参数使用，无需以字符串的形式传参，这样能够更好地实现自动补全、更不容易出错。使用 Signal 类型能够直接实现的功能见 [Signal](https://docs.godotengine.org/zh-cn/4.x/classes/class_signal.html#class-signal) 类参考手册。

现在，我们将使用信号来使上一节课（[监听玩家的输入](https://docs.godotengine.org/zh-cn/4.x/getting_started/step_by_step/scripting_player_input.html#doc-scripting-player-input)）中的 Godot 图标移动，并通过按下按钮来停止。

{% hint style="info" %}


对于此项目，我们将遵循 Godot 的命名约定。

* **GDScript**：类（节点）使用 PascalCase（大驼峰命名法），变量和函数使用 snake\_case（蛇形命名法），常量使用 ALL\_CAPS（全大写）（请参阅 [GDScript 编写风格指南](https://docs.godotengine.org/zh-cn/4.x/tutorials/scripting/gdscript/gdscript_styleguide.html#doc-gdscript-styleguide)）。
* **C#**：类、导出变量和方法使用 PascalCase（大驼峰命名法），私有字段使用 \_camelCase（前缀下划线的小驼峰命名法），局部变量和参数使用 camelCase（小驼峰命名法）（请参阅 [C# 风格指南](https://docs.godotengine.org/zh-cn/4.x/tutorials/scripting/c_sharp/c_sharp_style_guide.html#doc-c-sharp-styleguide)）。连接信号时，请务必准确键入方法名称。
{% endhint %}

### 场景设置

要为我们的游戏添加按钮，我们需要新建一个场景，包含一个[按钮](https://docs.godotengine.org/zh-cn/4.x/classes/class_button.html#class-button)以及之前课程 [创建第一个脚本](https://docs.godotengine.org/zh-cn/4.x/getting_started/step_by_step/scripting_first_script.html#doc-scripting-first-script) 编写的 `sprite_2d.tscn` 场景。

主菜单选择 Scene > New Scene，创建新场景。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_01_new_scene.webp" alt=""><figcaption></figcaption></figure>

在场景面板中，单击 2D Scene 按钮，即可添加一个 [Node2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_node2d.html#class-node2d) 作为场景根节点。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_02_2d_scene.webp" alt=""><figcaption></figcaption></figure>

在文件系统面板中，单击之前保存的 `sprite_2d.tscn` 文件并将其拖动到 Node2D 上，对其进行实例化。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_03_dragging_scene.webp" alt=""><figcaption></figcaption></figure>

我们希望添加另一个节点作为 Sprite2D 的同级节点。为此，请右键点击 Node2D，然后选择 Add Child Node。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_04_add_child_node.webp" alt=""><figcaption></figcaption></figure>

寻找并添加 [Button](https://docs.godotengine.org/zh-cn/4.x/classes/class_button.html#class-button) 节点。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_05_add_button.webp" alt=""><figcaption></figcaption></figure>

该节点默认比较小。在视口中，点击并拖拽该按钮右下角的手柄来调整大小。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_06_drag_button.png" alt=""><figcaption></figcaption></figure>

如果看不到手柄，请确保工具栏中的选择工具处于活动状态。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_07_select_tool.webp" alt=""><figcaption></figcaption></figure>

点击并拖拽按钮使其更接近精灵。

你也可以在 Inspector 中编辑 Button 的 Text 属性来显示文字。请输入 `Toggle motion`。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_08_toggle_motion_text.webp" alt=""><figcaption></figcaption></figure>

你的场景树和视口应该是类似这样的。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_09_scene_setup.webp" alt=""><figcaption></figcaption></figure>

如果你还没保存场景的话，保存新建的场景为 `node_2d.tscn`。然后你就可以使用 <kbd>F6</kbd> （macOS 则为 <kbd>Cmd + R</kbd> ）来运行。此时，你可以看到按钮，但是按下之后不会有任何反应。

### 在编辑器中连接信号

然后，我们希望将按钮的“pressed”信号连接到我们的 Sprite2D，并且我们想要调用一个新函数来打开和关闭其运动。我们需要像我们在上一课中所做的操作一样，将一个脚本附加到 Sprite2D 节点。

您可以在 Signals 面板中连接信号。选中 Button 节点，然后在编辑器右侧点击紧邻 Inspector 旁边的 Signals 。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_10_node_dock.webp" alt=""><figcaption></figcaption></figure>

停靠栏显示所选节点上可用的信号列表。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_11_pressed_signals.webp" alt=""><figcaption></figcaption></figure>

双击“pressed”信号，打开节点连接窗口。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_12_node_connection.webp" alt=""><figcaption></figcaption></figure>

然后，你可以将信号连接到 Sprite2D 节点。该节点需要一个用于接收按钮信号的函数，当按钮发出信号时，Godot 将调用该函数。编辑器会为你生成一个。按照规范，我们将这些回调方法命名为"\_on\_node\_name\_signal\_name"。在这里，它被命名为"\_on\_button\_pressed"。

{% hint style="info" %}
在通过编辑器的“信号”工具栏连接信号时，您可以采用两种模式。其中一种较为简单，仅允许您将信号连接到已附加脚本的节点上，并在这些节点上创建一个新的回调函数。

<p align="center"><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_advanced_connection_window.webp" alt="../../_images/signals_advanced_connection_window.webp" data-size="original"></p>

高级视图允许您连接到任意节点和内置函数，为回调添加参数并设置选项。您可以通过点击窗口右下角的 Advanced 按钮来切换此模式。
{% endhint %}

{% hint style="info" %}
如果你在使用一个外部代码编辑器（例如VS Code），可能会没有自动代码生成。在这种情况下，你需要按照下一部分阐述的方法使用信号连接代码。
{% endhint %}

点击 Connect 按钮以完成信号连接，并跳转至 Script 工作区。您应在左侧边栏看到带有连接图标的新增方法。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_13_signals_connection_icon.webp" alt=""><figcaption></figcaption></figure>

如果单击该图标，将弹出一个窗口并显示有关连接的信息。此功能仅在编辑器中连接节点时可用。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_14_signals_connection_info.webp" alt=""><figcaption></figcaption></figure>

让我们用代码替换带有 `pass` 关键字的一行，以切换节点的运动。

我们的 Sprite2D 由于 `_process()` 函数中的代码而移动。Godot 提供了一种打开和关闭处理的方法：[Node.set\_process()](https://docs.godotengine.org/zh-cn/4.x/classes/class_node.html#class-node-method-set-process) 。Node 的另一个方法 `is_processing()` ，如果空闲处理处于活动状态，则返回 `true`。我们可以使用 `not` 关键字来反转该值。

```gdscript
func _on_button_pressed():
	set_process(not is_processing())
```

此函数将切换处理，进而切换按下按钮时图标的移动。

在尝试游戏之前，我们需要简化 `_process()` 函数，以自动移动节点，而不是等待用户输入。将其替换为以下代码，这是我们在两课前看到的代码：

```gdscript
func _process(delta):
	rotation += angular_speed * delta
	var velocity = Vector2.UP.rotated(rotation) * speed
	position += velocity * delta
```

你的完整的 `Sprite_2d.gd` 代码应该是类似下面这样的。

```gdscript
extends Sprite2D

var speed = 400
var angular_speed = PI


func _process(delta):
	rotation += angular_speed * delta
	var velocity = Vector2.UP.rotated(rotation) * speed
	position += velocity * delta


func _on_button_pressed():
	set_process(not is_processing())
```

按下 <kbd>F6</kbd> 键运行当前场景（macOS为 <kbd>Cmd + R</kbd> ），然后点击按钮，就可以看到精灵开始或停止运动。

### 用代码连接信号

你可以通过代码连接信号，而不是使用编辑器。这在脚本中创建节点或实例化场景时是必需的。

让我们在这里使用一个不同的节点。Godot 有一个 [Timer](https://docs.godotengine.org/zh-cn/4.x/classes/class_timer.html#class-timer) 节点，可用于实现技能冷却时间、武器重装等。

回到 2D 工作区。你可以点击窗口顶部的“2D”字样，或者按 <kbd>Ctrl + F1</kbd>（macOS 上则是 <kbd>Ctrl + Cmd + 1</kbd>）。

在“场景”面板中，右键点击 Sprite2D 节点并添加新的子节点。搜索 Timer 并添加对应节点。你的场景现在应该类似这样。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_15_scene_tree.webp" alt=""><figcaption></figcaption></figure>

选中 Timer 节点后，前往 Inspector，并启用 Autostart 属性。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_18_timer_autostart.webp" alt=""><figcaption></figcaption></figure>

点击 Sprite2D 旁边的脚本图标，即可跳转回脚本工作区。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_16_click_script.webp" alt=""><figcaption></figcaption></figure>

我们需要执行两个操作，通过代码将节点连接起来：

1. 从 Sprite2D 获取对 Timer 的引用。
2. 在 Timer 的 "timeout" 信号上调用 `connect()` 方法。

{% hint style="info" %}
要通过代码连接信号，需要调用您希望监听的信号的 `connect()` 方法。在此例中，我们希望监听 Timer 的 "timeout" 信号。
{% endhint %}

我们想要在场景实例化时连接信号，我们可以使用 [Node.\_ready()](https://docs.godotengine.org/zh-cn/4.x/classes/class_node.html#class-node-private-method-ready) 内置函数来实现这一点，当节点完全实例化时，引擎会自动调用该函数。

为了获取相对于当前节点的引用，我们使用方法 [Node.get\_node()](https://docs.godotengine.org/zh-cn/4.x/classes/class_node.html#class-node-method-get-node)。我们可以将引用存储在变量中。

```gdscript
func _ready():
	var timer = get_node("Timer")
```

`get_node()` 函数会查看 Sprite2D 的子节点，并按节点的名称获取节点。例如，如果在编辑器中将 Timer 节点重命名为“BlinkingTimer”，则必须将调用更改为 `get_node("BlinkingTimer")`。

现在，我们可以在 `_ready()` 函数中将Timer连接到Sprite2D。

```gdscript
func _ready():
	var timer = get_node("Timer")
	timer.timeout.connect(_on_timer_timeout)
```

该行读起来是这样的：我们将计时器的“timeout”信号连接到脚本附加到的节点上。当计时器发出 `timeout` 时，去调用我们需要定义的函数 `_on_timer_timeout()`。让我们将其定义添加到脚本的底部，并使用它来切换精灵的可见性。

```gdscript
func _on_timer_timeout():
	visible = not visible
```

`visible` 属性是一个布尔值，用于控制节点的可见性。`visible = not visible` 行切换该值。如果 `visible` 是 `true`，它就会变成 `false`，反之亦然。

如果你现在运行 Node2D 场景，就会看到精灵在闪啊闪的，间隔为一秒。

### 完整脚本

这就是我们小小的 Godot 图标移动闪烁演示了！这是完整的 `sprite_2d.gd` 文件，仅供参考。

```gdscript
extends Sprite2D

var speed = 400
var angular_speed = PI


func _ready():
	var timer = get_node("Timer")
	timer.timeout.connect(_on_timer_timeout)


func _process(delta):
	rotation += angular_speed * delta
	var velocity = Vector2.UP.rotated(rotation) * speed
	position += velocity * delta


func _on_button_pressed():
	set_process(not is_processing())


func _on_timer_timeout():
	visible = not visible
```

### 自定义信号

您可以在脚本中定义自定义信号。例如，假设您希望在玩家的生命值为零时通过屏幕显示游戏结束。为此，当他们的生命值达到 0 时，您可以定义一个名为“died”或“health\_depleted”的信号。

```gdscript
extends Node2D

signal health_depleted

var health = 10
```

您的自定义信号与内置信号的工作方式相同：它们会显示在 Signals 标签页中，并且可以像其他信号一样进行连接。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/signals_17_custom_signal.webp" alt=""><figcaption></figcaption></figure>

要通过代码发出信号，请调用信号的 `emit()` 方法。

```gdscript
func take_damage(amount):
	health -= amount
	if health <= 0:
		health_depleted.emit()
```

信号还可以选择声明一个或多个参数。在括号之间指定参数的名称：

```gdscript
extends Node2D

signal health_changed(old_value, new_value)

var health = 10
```

信号参数会出现在编辑器的“信号”工具栏中，Godot 可以利用这些参数为您生成回调函数。不过，在发出信号时，您仍可以传递任意数量的参数。所以，您需要自行确定要传递的正确值。

要在发出信号的同时传值，请将它们添加为 `emit()` 函数的额外参数：

```gdscript
func take_damage(amount):
	var old_health = health
	health -= amount
	health_changed.emit(old_health, health)
```

### 总结

Godot 中的任何节点都会在发生特定事件时发出信号，例如按下按钮。其他节点可以连接到单个信号并对所选事件做出反应。

信号有很多用途。有了它们，你可以对进入或退出游戏世界的节点、碰撞、角色进入或离开某个区域、界面元素的大小变化等等做出反应。

例如，代表金币的 [Area2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_area2d.html#class-area2d) 会在玩家的物理实体进入其碰撞形状时发出 `body_entered` 信号，让你知道玩家收集到了金币。
