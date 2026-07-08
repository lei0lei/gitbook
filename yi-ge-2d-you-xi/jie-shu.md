# 结束

现在，我们已经完成了游戏的所有功能。以下是一些剩余的步骤，为游戏加点“料”，改善游戏体验。

随意用你自己的想法扩展游戏玩法。

### 背景

默认的灰色背景不是很吸引人，那么我们就来改一下颜色。一种方法是使用 [ColorRect](https://docs.godotengine.org/zh-cn/4.x/classes/class_colorrect.html#class-colorrect) 节点。将其设为 `Main` 下的第一个节点，这样这个节点就会绘制在其他节点之后。`ColorRect` 只有一个属性：`Color`（颜色）。选择一个你喜欢的颜色，然后在视口顶部的工具栏或者检查器中选择“布局”->“锚点预设”->“整个矩形”（Layout -> Anchors Preset -> Full Rect），使其覆盖屏幕。

如果你有背景图片，你也可以通过使用 [TextureRect](https://docs.godotengine.org/zh-cn/4.x/classes/class_texturerect.html#class-texturerect) 节点来添加它。

### 音效

声音和音乐可能是增强游戏吸引力的最有效方法。在游戏 **art** 文件夹中，有两个声音文件：“House in a Forest Loop.ogg”用于背景音乐，而“gameover.wav”用于当玩家失败时。

添加两个 [AudioStreamPlayer](https://docs.godotengine.org/zh-cn/4.x/classes/class_audiostreamplayer.html#class-audiostreamplayer) 节点作为 `Main` 的子节点。将其中一个命名为 `Music`，将另一个命名为 `DeathSound`。 在每个节点选项上，点击 `Stream` 属性，选择 “加载”，然后选择相应的音频文件。

自动导入音频时 `Loop` 设置是禁用的。如果希望音乐无缝循环，请单击流文件的下拉箭头，选择 `唯一化`，然后再单击流文件并选中 `Loop` 框。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/unique_resource_music.webp" alt=""><figcaption></figcaption></figure>

要播放音乐，请在 `new_game()` 函数中添加 `$Music.play()`，在 `game_over()` 函数中添加 `$Music.stop()`。

最后，在 `game_over()` 函数中添加 `$DeathSound.play()`。

```gdscript
func game_over():
	...
	$Music.stop()
	$DeathSound.play()

func new_game():
	...
	$Music.play()
```

### 键盘快捷键

当游戏使用键盘控制，可以方便地按键盘上的键来启动游戏。一种方法是使用 `Button` 节点的 “Shortcut”（快捷键）属性。

在上一课中，我们创建了四个输入动作来移动角色。我们将创建一个类似的输入动作来映射到开始按钮。

选择“项目 -> 项目设置”，然后单击“输入映射”选项卡。与创建移动输入动作的方式相同，创建一个名为 `start_game` 的新输入操作，并为 <kbd>Enter</kbd> 添加按键映射。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/input-mapping-start_game.webp" alt=""><figcaption></figcaption></figure>

如果你有一个手柄，现在可以添加一个手柄支持。连接上你的手柄，然后在每一个你想添加手柄支持的输入动作下，点击 "+" 按钮然后按下该输入动作对应的按钮，方向键或者摇杆。

在 `HUD` 场景中，选择 `StartButton` 并在检查器中找到它的 **Shortcut（快捷方式）**&#x5C5E;性。通过在框中单击来创建一个新的 [快捷键](https://docs.godotengine.org/zh-cn/4.x/classes/class_shortcut.html#class-shortcut) 资源，打开 **Events（事件）** 数组并通过单击 **Array\[InputEvent] (size 0)** 向其添加一个新的数组元素。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/start_button_shortcut.webp" alt=""><figcaption></figcaption></figure>

创建一个新的 [InputEventAction](https://docs.godotengine.org/zh-cn/4.x/classes/class_inputeventaction.html#class-inputeventaction)并选择 `start_game`动作。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/start_button_shortcut2.webp" alt=""><figcaption></figcaption></figure>

这样，开始按钮出现后，你就可以点击它或按 <kbd>Enter</kbd> 来启动游戏。

就这样，你在 Godot 中完成了你的第一个 2D 游戏。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/dodge_preview.gif" alt=""><figcaption></figcaption></figure>

你已经能够制作由玩家控制的角色、在游戏区域内随机产生的敌人、计算分数、实现游戏结束和重玩、用户界面、声音，以及更多内容。祝贺！

还有很多东西需要学习，但你可以花点时间来欣赏你所取得的成就。

当你准备好了，你可以继续学习 [你的第一个 3D 游戏](https://docs.godotengine.org/zh-cn/4.x/getting_started/first_3d_game/index.html#doc-your-first-3d-game)，学习在 Godot 中从头开始创建一个完整的 3D 游戏。

### 与他人分享完成的游戏

如果你希望别人无需安装 Godot 就能试玩你的游戏，你需要将项目导出为目标操作系统可玩的格式。有关详细说明，请参见 [导出项目](https://docs.godotengine.org/zh-cn/4.x/tutorials/export/exporting_projects.html#doc-exporting-projects)。

导出项目后，将导出的可执行文件和 PCK 文件（不是原始项目文件）压缩为 ZIP 文件，然后将此 ZIP 文件上传到文件共享网站。
