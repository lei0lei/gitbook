# 游戏信息显示

我们的游戏最后还需要用户界面（User Interface，UI），显示分数、“游戏结束”信息、重启按钮。

创建新场景，点击“其他节点”按钮，然后添加一个 [CanvasLayer](https://docs.godotengine.org/zh-cn/4.x/classes/class_canvaslayer.html#class-canvaslayer) 节点并命名为 `HUD`。“HUD”是“heads-up display”（游戏信息显示）的缩写，是覆盖在游戏视图上显示的信息。

[CanvasLayer](https://docs.godotengine.org/zh-cn/4.x/classes/class_canvaslayer.html#class-canvaslayer) 节点可以让我们在游戏的其他部分的上一层绘制 UI 元素，这样它所显示的信息就不会被任何游戏元素（如玩家或敌人）所覆盖。

HUD 中需要显示以下信息：

* 得分，由 `ScoreTimer` 更改。
* 消息，例如“Game Over”或“Get Ready!”
* “Start”按钮来开始游戏。

UI 元素的基本节点是 [Control](https://docs.godotengine.org/zh-cn/4.x/classes/class_control.html#class-control)。要创建 UI，我们需使用 [Control](https://docs.godotengine.org/zh-cn/4.x/classes/class_control.html#class-control) 下的两种节点：[Label](https://docs.godotengine.org/zh-cn/4.x/classes/class_label.html#class-label) 和 [Button](https://docs.godotengine.org/zh-cn/4.x/classes/class_button.html#class-button)。

创建以下节点作为 `HUD` 的子节点：

* 名为分数标签 `ScoreLabel` 的 [Label](https://docs.godotengine.org/zh-cn/4.x/classes/class_label.html#class-label)。
* 名为消息 `Message` 的 [Label](https://docs.godotengine.org/zh-cn/4.x/classes/class_label.html#class-label)。
* 名为开始按钮 `StartButton` 的 [Button](https://docs.godotengine.org/zh-cn/4.x/classes/class_button.html#class-button)。
* 名为信息计数器 `MessageTimer` 的 [Timer](https://docs.godotengine.org/zh-cn/4.x/classes/class_timer.html#class-timer)。

点击 `ScoreLabel` 并在“检查器”的 `Text` 字段中键入一个数字。`Control` 节点的默认字体很小，不能很好地缩放。游戏资产包中有一个叫作“Xolonium-Regular.ttf”的字体文件。 使用此字体需要执行以下操作：

在“Theme Overrides > Fonts”（主题覆盖 > 字体）中选择“加载”，然后选中“Xolonium-Regular.ttf”文件。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/custom_font_load_font.webp" alt=""><figcaption></figcaption></figure>

字体尺寸仍然太小，请在“Theme Overrides > Font Sizes”（主题覆盖 > 字体大小）下将其增加到 `64`。当 `ScoreLabel` 完成此操作后，请重复对 `Message` 和 `StartButton` 节点做同样的修改。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/custom_font_size.webp" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**锚点：**`Control` 节点具有位置和大小，但它也有锚点（Anchor）。锚点定义的是原点——节点边缘的参考点。
{% endhint %}

请将节点如下图排列。拖动节点可以手动放置，也可以使用“锚点预设（Anchor Preset）”进行更精确的定位。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/ui_anchor.webp" alt=""><figcaption></figcaption></figure>

### ScoreLabel

1. 添加文本 `0`。
2. 将“Horizontal Alignment”和“Vertical Alignment”设置为 `Center`。
3. 为“Anchor Preset”选择 `Center Top`。

### Message

1. 添加文本 `Dodge the Creeps!`。
2. 将“Horizontal Alignment”和“Vertical Alignment”设置为 `Center`。
3. 将“Autowrap Mode”设置为 `Word`，否则标签只会有一行。
4. 在“Control - Layout/Transform”中将“Size X”设置为 `480`，使用屏幕的完整宽度。
5. 为“Anchor Preset”选择 `Center`。

### StartButton

1. 添加文本 `Start`。
2. 在“Control - Layout/Transform”中将“Size X”设置为 `200`、“Size Y”设置为 `100`，在边框和文本之间添加间距。
3. 为“Anchor Preset”选择 `Center Bottom`。
4. 在“Control - Layout/Transform”中将“Position Y”设置为 `580`。

在 `MessageTimer` 中，将 `Wait Time` 设置为 `2` 并将 `One Shot` 属性设置为“启用”。

现将这个脚本添加到 `HUD`：

```gdscript
extends CanvasLayer

# Notifies `Main` node that the button has been pressed
signal start_game
```

当想显示一条临时消息时，比如“Get Ready”，就会调用这个函数

```gdscript
func show_message(text):
	$Message.text = text
	$Message.show()
	$MessageTimer.start()
```

我们还需要处理玩家死亡的情况。以下代码会显示 2 秒“Game Over”，然后返回标题屏幕，暂停一会儿之后再显示“Start”按钮。

```gdscript
func show_game_over():
	show_message("Game Over")
	# Wait until the MessageTimer has counted down.
	await $MessageTimer.timeout

	$Message.text = "Dodge the Creeps!"
	$Message.show()
	# Make a one-shot timer and wait for it to finish.
	await get_tree().create_timer(1.0).timeout
	$StartButton.show()
```

{% hint style="info" %}
当你需要暂停片刻时，可以使用场景树的 `create_timer()` 函数替代 Timer 节点。这在需要添加延迟时非常有用，例如在上述代码中，我们希望在显示 “Start” 按钮之前等待一段时间。
{% endhint %}

将以下更新分数代码添加到 `HUD` 中

```gdscript
func update_score(score):
	$ScoreLabel.text = str(score)
```

将 `StartButton` 的 `pressed()` 信号与 `MessageTimer` 的 `timeout()` 信号连接到 `HUD` 节点上，然后在新函数中添加以下代码：

```gdscript
func _on_start_button_pressed():
	$StartButton.hide()
	start_game.emit()

func _on_message_timer_timeout():
	$Message.hide()
```

### 将 HUD 场景连接到 Main 场景

现在我们完成了 `HUD` 场景，保存并返回 `Main` 场景。和 `Player` 场景的做法一样，在 `Main` 场景中实例化 `HUD` 场景。如果你没有错过任何东西，完整的场景树应该像这样：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/completed_main_scene.webp" alt=""><figcaption></figcaption></figure>

现在我们需要将 `HUD` 功能与我们的 `Main` 脚本连接起来。这需要在 `Main` 场景中添加一些内容：

在信号（Signals）标签页中，把 HUD 的 `start_game` 信号连接到 Main 节点的 `new_game()` 函数上。你可以点击 "连接信号（Connect a Signal）" 窗口里的 "拾取（Pick）" 按钮，然后选中 `new_game()` 方法；或者直接在窗口里 "接收方法（Receiver Method）" 的下方输入 "new\_game" 。操作完成后，记得去脚本里确认一下，看看 `func new_game()` 这一行的旁边是不是已经出现了一个绿色的连接小图标。

在 `new_game()` 函数中，更新分数显示并显示“Get Ready”消息：

```gdscript
$HUD.update_score(score)
$HUD.show_message("Get Ready")
```

在 `game_over()` 中我们需要调用相应的 `HUD` 函数：

```gdscript
$HUD.show_game_over()
```

最后，将下面的代码添加到 `_on_score_timer_timeout()` 中，保持不断变化的分数的同步显示：

```gdscript
$HUD.update_score(score)
```

{% hint style="info" %}
如果还没做的话，请不要忘记在 `_ready()` 中移除对 `new_game()` 的调用。否则你的游戏将自动开始。
{% endhint %}

现在你就可以开始游戏了！点击“运行项目”按钮。

### 删除旧的小怪

如果你一直玩到“游戏结束”，然后重新开始新游戏，上局游戏的小怪仍然显示在屏幕上。更好的做法是在新游戏开始时清除它们。我们需要一个同时&#x8BA9;_&#x6240;&#x6709;_&#x5C0F;怪删除它自己的方法，为此可以使用“分组”功能。

在 `Mob` 场景中，选择根节点，然后选择 Signals 选项卡旁边的 Groups 选项卡并点击 “+” 按钮以打开 “创建新组” 对话框。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/group_tab.webp" alt=""><figcaption></figcaption></figure>

将分组命名为 `mobs`，然后点击 "确定" 以添加一个新场景分组。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/add_group_dialog.webp" alt=""><figcaption></figcaption></figure>

现在所有的敌人（mobs）都会在 "mobs" 组中。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/scene_group_mobs.webp" alt=""><figcaption></figcaption></figure>

然后，我们可以在 `Main` 中的 `new_game()` 函数中添加下面这些行：

```gdscript
get_tree().call_group("mobs", "queue_free")
```

`call_group()` 函数调用组中每个节点上的删除函数——让每个怪物删除其自身。

游戏在这一点上大部分已经完成。在下一部分和最后一部分中，我们将通过添加背景，循环音乐和一些键盘快捷键来对其进行一些润色。
