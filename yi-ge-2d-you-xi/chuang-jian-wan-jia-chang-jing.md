# 创建玩家场景

项目设置到位后，我们可以开始处理玩家控制的角色。

第一个场景，我们会定义 `Player` 对象。 单独创建Player场景的好处之一是，在游戏的其他部分做出来之前，我们就可以对其进行单独测试。

### 节点结构

首先，我们需要为玩家对象选择一个根节点。一般来说，场景的根节点应该反映对象的预期功能——即对&#x8C61;_&#x662F;什么_。在左上角的“场景”选项卡中，点击“其他节点”按钮，向场景中添加一个 [Area2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_area2d.html#class-area2d) 节点。

{% hint style="info" %}
Godot 还提供了专为 2D 角色设计的 [CharacterBody2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_characterbody2d.html#class-characterbody2d) 节点，该节点内置了对本教程中所述部分流程的支持。在许多实际项目中，CharacterBody2D 是玩家和敌人的更佳选择。然而，本教程侧重于讲解适用于更广泛节点和用例的核心概念。
{% endhint %}

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/add_node.webp" alt=""><figcaption></figcaption></figure>

添加 `Area2D` 节点后，Godot 会在场景树中该节点的旁边显示**警告图标**：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/no_shape_warning.webp" alt=""><figcaption></figcaption></figure>

这个警告告诉我们 `Area2D` 节点需要一个形状来检测碰撞或重叠。我们可以 **暂时忽略这个警告**，因为我们将首先设置玩家的视觉效果（使用动画精灵）。当视觉效果准备好后，我们将添加一个碰撞形状作为子节点。这样，我们可以根据精灵的外观准确地调整和定位形状的大小。

使用 `Area2D` 可以检测到与玩家重叠或进入玩家内的物体。 通过双击节点名称将其名称更改为 `Player`。 我们已经设置好了场景的根节点，现在可以向该角色中添加其他节点来增加功能。

在将任何子节点添加到 `Player` 节点之前，我们要确保不会通过点击它们来意外移动它们或调整它们的大小。选择该节点并单击锁右侧的图标。其工具提示显示“将所选节点与其子节点组合。这样在 2D 和 3D 视图中点击子节点就会选中父节点。”

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/lock_children.webp" alt=""><figcaption></figcaption></figure>

将场景保存为 `player.tscn`。点击**场景 > 保存**，或者在 Windows/Linux 上按 <kbd>Ctrl + S</kbd>，在 macOS 上按 <kbd>Cmd + S</kbd>。

{% hint style="info" %}


对于此项目，我们将遵循 Godot 的命名约定。

* **GDScript**：类（节点）使用 PascalCase（大驼峰命名法），变量和函数使用 snake\_case（蛇形命名法），常量使用 ALL\_CAPS（全大写）（请参阅 [GDScript 编写风格指南](https://docs.godotengine.org/zh-cn/4.x/tutorials/scripting/gdscript/gdscript_styleguide.html#doc-gdscript-styleguide)）。
* **C#**：类、导出变量和方法使用 PascalCase（大驼峰命名法），私有字段使用 \_camelCase（前缀下划线的小驼峰命名法），局部变量和参数使用 camelCase（小驼峰命名法）（请参阅 [C# 风格指南](https://docs.godotengine.org/zh-cn/4.x/tutorials/scripting/c_sharp/c_sharp_style_guide.html#doc-c-sharp-styleguide)）。连接信号时，请务必准确键入方法名称。
{% endhint %}

### 精灵动画

点击 `Player` 节点并添加（Windows/Linux 下按 <kbd>Ctrl + A</kbd>， macOS 下按 <kbd>Cmd + A</kbd>）一个子节点 [AnimatedSprite2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_animatedsprite2d.html#class-animatedsprite2d)。`AnimatedSprite2D` 将为我们的玩家处理外观和动画。 请注意，节点旁边有一个警告符号。 一个 `AnimatedSprite2D` 需要一个 [SpriteFrames](https://docs.godotengine.org/zh-cn/4.x/classes/class_spriteframes.html#class-spriteframes) 资源，这是它可以显示的动画的列表。 确保已选中 `AnimatedSprite2D`，然后在检查器的 `Animation` 部分找到 `Sprite Frames` 属性，并单击 “\[empty]” -> “新建 SpriteFrames”：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/new_spriteframes.webp" alt=""><figcaption></figcaption></figure>

点击你刚刚创建的 `SpriteFrames`，以打开 "SpriteFrames" 面板：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/spriteframes_panel.webp" alt=""><figcaption></figcaption></figure>

左侧是动画列表。点击 `default` 并将其重命名为 `walk`。然后点击**添加动画**按钮，再创建一个名为 `up` 的动画。

在“文件系统”面板中找到玩家图像——它们就在你之前解压缩的 `art` 文件夹中。 将每个动画的两张图片拖到相应动画面板的 **动画帧** 侧：

* `playerGrey_walk1` 和 `playerGrey_walk2` 用于 `walk` 动画
* `playerGrey_up1` 和 `playerGrey_up2` 用于 `up` 动画

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/spriteframes_panel2.webp" alt=""><figcaption></figcaption></figure>

玩家图像对于游戏窗口来说有点过大，需要缩小它们。点击 `AnimatedSprite2D` 节点，可以在检查器 `Node2D` 标签中，将 `Scale` 属性设置为 `(0.5, 0.5)`。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/player_scale.webp" alt=""><figcaption></figcaption></figure>

最后，在 `Player` 下添加一个 [CollisionShape2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_collisionshape2d.html#class-collisionshape2d) 作为子节点，以确定玩家的“攻击框”，或者说碰撞范围。`CapsuleShape2D` 节点最适合这个角色，那么就在检查器中“Shape”的旁边点击“\[空]”->“新建 CapsuleShape2D”添加形状，使用两个控制柄，调整形状大小以覆盖精灵：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/player_coll_shape1.webp" alt=""><figcaption></figcaption></figure>

完成后，你的 `Player` 场景看起来应该像这样:

<div align="center"><figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/player_scene_nodes.webp" alt=""><figcaption></figcaption></figure></div>

完成后，`Area2D` 节点上的警告将消失，因为它现在已经分配了形状，并且可以与其他对象进行交互。

修改完成后请确保再次保存场景。

在下一部分中，我们将向玩家节点添加一个脚本，以移动它并为其添加动画效果。然后，我们将设置碰撞检测，以了解玩家何时被某些东西击中。











