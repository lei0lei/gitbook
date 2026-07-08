# 创建敌人

是时候去做一些玩家必须躲避的敌人了。 它们的行为很简单: 怪物将随机生成在屏幕的边缘，沿着随机的方向直线移动。

我们将创建一个 `Mob` 的怪物场景，以便在游戏中独&#x7ACB;_&#x5B9E;例&#x5316;_&#x51FA;任意数量的怪物。

### 节点设置[](https://docs.godotengine.org/zh-cn/4.x/getting_started/first_2d_game/04.creating_the_enemy.html#node-setup)

点击顶部菜单的“场景 -> 新建场景”，然后添加以下节点：

*   [RigidBody2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_rigidbody2d.html#class-rigidbody2d)（名为 `Mob`）

    > * [AnimatedSprite2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_animatedsprite2d.html#class-animatedsprite2d)
    > * [CollisionShape2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_collisionshape2d.html#class-collisionshape2d)
    > * [VisibleOnScreenNotifier2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_visibleonscreennotifier2d.html#class-visibleonscreennotifier2d)

别忘了设置子节点，使其无法被选中，就像你在“Player”场景中所做的那样。具体方法是：在场景树面板中选择父节点（ `RigidBody2D` ），然后点击2D编辑器顶部的 Group 按钮（或在 macOS 上按下 <kbd>Ctrl + G</kbd> / <kbd>Cmd + G</kbd> 快捷键）。

选择 `Mob` 节点，并在检查器的 [RigidBody2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_rigidbody2d.html#class-rigidbody2d) 部分中把它的 `Gravity Scale` 属性设置为 `0`。这样可以防止怪物向下坠落。

此外，在 `Mob` 节点的检查器中，找到 [CollisionObject2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_collisionobject2d.html#class-collisionobject2d) 部分，展开 **Collision** 分组，并取消勾选 `Mask` 属性内的 `1` 。这样可以确保怪物之间不会相互碰撞。

![../../\_images/set\_collision\_mask.webp](https://docs.godotengine.org/zh-cn/4.x/_images/set_collision_mask.webp)

像设置玩家一样设置 [AnimatedSprite2D](https://docs.godotengine.org/zh-cn/4.x/classes/class_animatedsprite2d.html#class-animatedsprite2d)。这一次，我们有 3 个动画：`fly`、`swim`、`walk`，每个动画在 art 文件夹中都有两张图片。

必须为每个单独动画设置 `动画速度` 属性，将三个动画的对应动画速度值都调整为 `3`。

![../../\_images/mob\_animations.webp](https://docs.godotengine.org/zh-cn/4.x/_images/mob_animations.webp)

你可以使用 `动画速度` 输入区域右侧的“播放动画”按钮预览动画。

我们将随机选择其中一个动画，以便小怪有一些变化。

像玩家的图像一样，这些小怪的图像也要缩小。请将 `AnimatedSprite2D` 的 `Scale` 属性设为 `(0.75, 0.75)`。

像在 `Player` 场景中一样，为碰撞添加一个 `CapsuleShape2D`。为了使形状与图像对齐，你需要将 `Rotation` 属性设为 `90`（在“检查器”的“Transform”下）。

保存该场景。

### 敌人的脚本[](https://docs.godotengine.org/zh-cn/4.x/getting_started/first_2d_game/04.creating_the_enemy.html#enemy-script)

像这样将脚本添加到 `Mob` 上：

```gdscript
extends RigidBody2D
```

现在让我们看一下脚本的其余部分。在 `_ready()` 中，我们从三个动画类型中随机选择一个播放：

```gdscript
func _ready():
	var mob_types = Array($AnimatedSprite2D.sprite_frames.get_animation_names())
	$AnimatedSprite2D.animation = mob_types.pick_random()
	$AnimatedSprite2D.play()
```

首先，我们从 AnimatedSprite2D 的 `sprite_frames` 属性中获取动画名称的列表。返回的是一个数组，该数组包含三个动画名称：`["walk", "swim", "fly"]`。

在 GDScript 代码中，我们使用 [Array.pick\_random](https://docs.godotengine.org/zh-cn/4.x/classes/class_array.html#class-array-method-pick-random) 方法从这些动画名称中随机选择一个。 同时，在 C# 代码中，我们会在 `0` 和 `2` 之间随机选择一个数字，从列表中选择其中一个名称（数组索引从 `0` 开始）。 表达式 `GD.Randi() % n` 选择一个介于 `0` 和 `n-1` 之间的随机整数。

最后，我们调用 `play()` 来播放选中的动画。

最后一步是让怪物在超出屏幕时删除自己。将 `VisibleOnScreenNotifier2D` 节点的 `screen_exited()` 信号连接到 `Mob` 上，然后添加如下代码：

```gdscript
func _on_visible_on_screen_notifier_2d_screen_exited():
	queue_free()
```

`queue_free()` 是一个函数，基本上是在帧结束时“释放”或删除节点。

这样就完成了 _Mob_ 场景。

玩家和敌人已经准备就绪，接下来，我们将在一个新的场景中把他们放到一起。我们将使敌人在游戏板上随机生成并前进，我们的项目将变成一个能玩的游戏。
