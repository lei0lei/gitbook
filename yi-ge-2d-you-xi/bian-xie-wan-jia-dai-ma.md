# 编写玩家代码

在这一课中，我们将添加玩家的动作、动画，并为其设置碰撞检测。

现在我们需要添加一些内置节点所不具备的功能，因此要添加一个脚本。点击 `Player` 节点然后点击“附加脚本”按钮：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/add_script_button.webp" alt=""><figcaption></figcaption></figure>

在脚本设置窗口中，你可以维持默认设置。点击“创建”即可：

{% hint style="info" %}
如果你要创建 C# 脚本或者其他语言的脚本，那就在创建之前&#x5728;_&#x8BED;&#x8A00;_&#x4E0B;拉菜单中选择语言。
{% endhint %}

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/attach_node_window.webp" alt=""><figcaption></figcaption></figure>

首先声明该对象将需要的成员变量：

```gdscript
extends Area2D

@export var speed = 400 # How fast the player will move (pixels/sec).
var screen_size # Size of the game window.
```

在第一个变量 `speed` 前面加上 `export` 关键字，就能让我们在检视器（Inspector）里直接设置它的值。对于那些你希望能像调整节点内置属性一样、随时方便修改的数值来说，这个功能非常好用。点击 `Player` 节点，你会发现这个属性现在出现在了检视器里，并且归类在一个以该脚本名字命名的新板块下。记住哦： 如果你在检视器里修改了这个值，它会覆盖脚本里写的默认值（但不会真的去改动你的脚本代码）。

{% hint style="info" %}
如果你在使用 C# ，想要查看新的导出变量或信号，就需要（重新）构建项目程序集。可以通过点击编辑器右上角的 **构建** 按钮手动触发构建过程。

![../../\_images/build\_dotnet1.webp](https://docs.godotengine.org/zh-cn/4.x/_images/build_dotnet1.webp)
{% endhint %}

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/export_variable.webp" alt=""><figcaption></figcaption></figure>

你的 `player.gd` 脚本应该已经包含一个 `_ready()` 和一个 `_process()` 函数。如果你没有选择上面展示的默认模板，请在学习本课程的同时创建这些函数。

当节点进入场景树时，`_ready()` 函数被调用，这是查看游戏窗口大小的好时机：

```gdscript
func _ready():
	screen_size = get_viewport_rect().size
```

现在我们可以使用 `_process()` 函数定义玩家将执行的操作。`_process()` 在每一帧都被调用，因此我们将使用它来更新我们希望会经常变化的游戏元素。对于玩家而言，我们需要执行以下操作：

* 检查输入。
* 沿给定方向移动。
* 播放合适的动画。

首先，我们需要检查输入——玩家是否正在按键？对于这个游戏，我们有 4 个方向的输入要检查。输入动作在项目设置中的“输入映射”下定义。在这里，你可以定义自定义事件，并为其分配不同的按键、鼠标事件、或者其他输入。对于此游戏，我们将把方向键映射给四个方向。

点&#x51FB;_&#x9879;目 -> 项目设&#x7F6E;_&#x6253;开项目设置窗口，然后单击顶部&#x7684;_&#x8F93;入映&#x5C04;_&#x9009;项卡。在顶部栏中键入“move\_right”，然后单击“添加”按钮以添加该 `move_right` 动作。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/input-mapping-add-action.webp" alt=""><figcaption></figcaption></figure>

我们需要为这个操作分配一个按键。单击右侧的“+”图标，打开事件管理器窗口。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/input-mapping-add-key.webp" alt=""><figcaption></figcaption></figure>

会自动选中“正在监听输入...”区域。按下键盘上的“右方向”键，菜单应该像这样。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/input-mapping-event-configuration.webp" alt=""><figcaption></figcaption></figure>

选择“确定”按钮。现在“右方向”键与 `move_right` 动作关联了。

重复这些步骤以再添加三个映射：

1. `move_left` 映射到左箭头键。
2. `move_up` 映射到向上箭头键。
3. `move_down` 映射到向下箭头键。

按键映射选项卡应该看起来类似这样：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/input-mapping-completed.webp" alt=""><figcaption></figcaption></figure>

单击“关闭”按钮关闭项目设置。

{% hint style="info" %}
我们只将一个键映射到每个输入动作，但你可以将多个键、操纵杆按钮或鼠标按钮映射到同一个输入动作。
{% endhint %}

你可以使用 `Input.is_action_pressed()` 来检测是否按下了某个键，如果按下会返回 `true`，否则返回 `false`。

```gdscript
func _process(delta):
	var velocity = Vector2.ZERO # The player's movement vector.
	if Input.is_action_pressed("move_right"):
		velocity.x += 1
	if Input.is_action_pressed("move_left"):
		velocity.x -= 1
	if Input.is_action_pressed("move_down"):
		velocity.y += 1
	if Input.is_action_pressed("move_up"):
		velocity.y -= 1

	if velocity.length() > 0:
		velocity = velocity.normalized() * speed
		$AnimatedSprite2D.play()
	else:
		$AnimatedSprite2D.stop()
```

我们首先将 `velocity` 设置为 `(0, 0)`——默认情况下玩家不应该移动。然后我们检查每个输入并从 `velocity` 中进行加/减以获得总方向。例如，如果你同时按住 `右` 和 `下`，则生成的 `velocity` 向量将为 `(1, 1)`。此时，由于我们同时向水平和垂直两个方向进行移动，玩家斜向移动的速度将会比水平移动&#x8981;_&#x66F4;快_。

只要对速度进&#x884C;_&#x5F52;一&#x5316;_&#x5C31;可以防止这种情况，也就是将速度&#x7684;_&#x957F;&#x5EA6;_&#x8BBE;置为 `1`，然后乘以想要的速度。这样就不会有过快的斜向运动了。

我们还会检查玩家是否正在移动，以便在 AnimatedSprite2D 上调用 `play()` 或 `stop()`。

{% hint style="info" %}
`$` 是 `get_node()` 的简写。因此在上面的代码中，`$AnimatedSprite2D.play()` 与 `get_node("AnimatedSprite2D").play()` 相同。

在 GDScript 中，`$` 返回从当前节点开始的相对路径上的节点，如果找不到该节点，则返回 `null`。当前 AnimatedSprite2D 是该节点子节点，因而可以使用 `$AnimatedSprite2D` 以获取。
{% endhint %}

现在我们有了一个运动方向，我们可以更新玩家的位置了。我们也可以使用 `clamp()` 来防止它离开屏幕。 _clamp_ 一个值意味着将其限制在给定范围内。将以下内容添加到 `_process` 函数的底部：

```gdscript
position += velocity * delta
position = position.clamp(Vector2.ZERO, screen_size)
```

{% hint style="info" %}
_\_process()_ 函数的 _delta_ 参数是 _帧长度_ ——完成上一帧所花费的时间。 使用这个值的话，可以保证你的移动不会被帧率的变化所影响。
{% endhint %}

点击“运行当前场景”（<kbd>F6</kbd>，macOS 上为 <kbd>Cmd + R</kbd>）并确认你能够在屏幕中沿任一方向移动玩家。

{% hint style="info" %}
如果在“调试器”面板中出现错误

`Attempt to call function 'play' in base 'null instance' on a null instance`（尝试调用空实例在基类“空实例”上的“play”函数）

这可能意味着你拼错了 AnimatedSprite2D 节点的名称。节点名称区分大小写，并且 `$NodeName` 必须与你在场景树中看到的名称匹配。
{% endhint %}

### 选择动画

现在玩家可以移动了，我们需要根据方向更改 AnimatedSprite2D 所播放的动画。我们的“walk”动画显示的是玩家向右走。向左移动时就应该使用 `flip_h` 属性将这个动画进行水平翻转。我们还有向上的“up”动画，向下移动时就应该使用 `flip_v` 将其进行垂直翻转。让我们把这段代码放在 `_process()` 函数的末尾：

```gdscript
if velocity.x != 0:
	$AnimatedSprite2D.animation = "walk"
	$AnimatedSprite2D.flip_v = false
	# See the note below about the following boolean assignment.
	$AnimatedSprite2D.flip_h = velocity.x < 0
elif velocity.y != 0:
	$AnimatedSprite2D.animation = "up"
	$AnimatedSprite2D.flip_v = velocity.y > 0
```

上面代码中的布尔赋值是程序员常用的缩写。 在做布尔比较同时，同时可 _赋_ 一个布尔值。 参考这段代码与上面的单行布尔赋值:

```gdscript
if velocity.x < 0:
	$AnimatedSprite2D.flip_h = true
else:
	$AnimatedSprite2D.flip_h = false
```

再次播放场景并检查每个方向上的动画是否正确。

这里一个常见错误是打错了动画的名字。“动画帧”面板中的动画名称必须与在代码中键入的内容匹配。如果你将动画命名成了 `"Walk"`，就必须在代码中也使用大写的“W”。

当你确定移动正常工作时， 请将此行添加到 `_ready()` 中，在游戏开始时隐藏玩家：

```gdscript
hide()
```

### 准备碰撞

我们希望 `Player` 能够检测到何时被敌人击中，但是我们还没有任何敌人！没关系，因为我们将使用Godot的 _信号_ 功能来使其正常工作。

在脚本顶部添加以下内容。如果你使用的是 GDScript，请将其添加到 `extends Area2D` 之后。如果你使用 C#，请将其添加到 `public partial class Player : Area2D` 之后：

```gdscript
signal hit
```

这定义了一个叫作“hit”的自定义信号，当玩家与敌人碰撞时，我们会让他发出这个信号。我们将使用 `Area2D` 来检测碰撞。选中 `Player` 节点，然后点击“检查器”选项卡旁边的 Signals 选项卡，就可以查看玩家可以发出的信号列表：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/player_signals.webp" alt=""><figcaption></figcaption></figure>

请注意自定义的“hit”信号也在其中！由于敌人将是 `RigidBody2D` 节点，所以需要 `body_entered(body: Node2D)` 信号。当物体接触到玩家时就会发出这个信号。点击“连接...”就会出现“连接信号”窗口。

Godot 将直接在脚本中为你创建一个具有确切名称的函数。现在你不需要更改默认设置。

{% hint style="info" %}
如果你使用外部文本编辑器（例如 Visual Studio Code），当前有一个错误会阻止 Godot 执行此操作。你将被送到外部编辑器那边，但在那里并不会有新函数。

在这种情况下，你需要自己将该函数写入玩家的脚本文件中。
{% endhint %}

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/player_signal_connection.webp" alt=""><figcaption></figcaption></figure>

注意，绿色图标表示信号已连接到此函数；但这并不意味着该函数存在，只是信号将尝试连接到具有该名称的函数。因此请仔细检查该函数的拼写是否能完全匹配上！

接下来，将此代码添加到函数中：

```gdscript
func _on_body_entered(_body):
	hide() # Player disappears after being hit.
	hit.emit()
	# Must be deferred as we can't change physics properties on a physics callback.
	$CollisionShape2D.set_deferred("disabled", true)
```

敌人每次击中 玩家时都会发出一个信号。我们需要禁用玩家的碰撞检测，确保我们不会多次触发 `hit` 信号。

如果在引擎的碰撞处理过程中禁用区域的碰撞形状可能会导致错误。使用 `set_deferred()` 告诉 Godot 等待可以安全地禁用形状时再这样做。

最后再为玩家添加一个函数，用于在开始新游戏时调用来重置玩家。

```gdscript
func start(pos):
	position = pos
	show()
	$CollisionShape2D.disabled = false
```

