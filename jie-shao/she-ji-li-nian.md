# 设计理念

### 面向对象的设计与组合

Godot凭借其灵活的场景系统和节点层次结构，将面向对象设计作为其核心。 它试图远离严格的编程模式，以提供直观的方式来构建游戏。

首先，Godot 可以让你把场景**组合或聚合**起来。这和嵌套的预制件是类似的：你可以创建 BlinkingLight 场景，并使用 BlinkingLight 创建 BrokenLantern 场景。然后，创建一个充满 BrokenLantern 的城市。更改 BlinkingLight 的颜色、保存，城市中的所有 BrokenLantern 都会立即更新。

更重要的是，你可以从任何场景**继承**。

Godot 场景可以是武器、角色、物品、门、关卡、关卡的一部分……任何你能想象的东西。它就像纯代码中的类一样工作，但也可以使用编辑器，纯代码或同时使用两者来设计场景。

和其他几个 3D 引擎的 Prefab（预制体）不同，场景可以通过继承来扩展。你可以创建一个 Magician（魔术师）来扩展你的 Character（角色）。在编辑器中修改 Character 后 Magician 也会更新。这样的设计可以帮你保持项目结构与设计的一致性。

![image0](https://docs.godotengine.org/zh-cn/4.x/_images/engine_design_01.png)

还要注意，Godot 提供了许多不同类型的对象，称为节点，每种节点都有特定的用途。节点是树的一部分，并且始终从其父类继承直到 Node 类。虽然引擎确实具有一些节点，例如碰撞形状将被其父物理体使用，但大多数节点彼此独立工作。

换句话说，Godot的节点并不像其他一些游戏引擎中的组件那样工作。

![image1](https://docs.godotengine.org/zh-cn/4.x/_images/engine_design_02.png)

Sprite2D 是一种 Node2D、CanvasItem 和 Node。它具有其三个父类的所有属性和功能，例如变换或绘制自定义形状和使用自定义着色器进行渲染的能力。

### 完善的工具集

Godot 尝试提供自己的工具来满足最常见的需求。它具有专用的脚本工作区、动画编辑器、TileMap 编辑器、着色器编辑器、调试器、分析器、在本地和远程设备上热重载等功能。

![image2](https://docs.godotengine.org/zh-cn/4.x/_images/engine_design_03.png)

目标是为游戏开发提供‌完整的解决方案‌，确保‌流畅的用户体验‌。只要 Godot 中有对应的导入插件，你就仍然可以使用外部程序进行编辑。

这也是 Godot 除了提供 C# 之外还会提供自己的编程语言 GDScript 的部分原因。GDScript 是为满足游戏开发人员和游戏设计师的需求而设计的，并且被紧密集成在引擎和编辑器中。

GDScript 让你能够使用基于缩进的语法编写代码，还可以检测类型，提供质量接近静态语言的自动补全。它还针对使用 Vector、Color 等内置类型的游戏代码进行了优化。

请注意，使用 GDExtension，你可以编写出使用类似 C、C++、Rust、D、Haxe、Swift 这类的编译语言编写的高性能代码，并且无需重新编译引擎。

请注意，3D 工作区不像 2D 工作区那样有那么多工具。你将需要使用外部的程序或插件来编辑地形，给复杂的角色模型制作动画等。Godot 提供了完整的 API，可以直接使用编写游戏的代码来扩展编辑器的功能。参见下面的 [Godot 编辑器是一个 Godot 游戏](https://docs.godotengine.org/zh-cn/4.x/getting_started/introduction/godot_design_philosophy.html#the-godot-editor-is-a-godot-game)。

### Godot 编辑器是一款 Godot 游戏

Godot 编辑器在游戏引擎上运行。它使用引擎自己的 UI 系统，可以在你测试项目时热重载代码和场景，也可以在编辑器中运行游戏代码。这意味着你可以为游戏**使用相同的代码**和场景，又可以**构建插件并扩展编辑器。**

这带来了可靠且灵活的 UI 系统，因此它为编辑器本身提供了动力。使用 `@tool` 注解，你可以在编辑器中运行任何游戏代码。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/introduction_rpg_in_a_box.webp" alt="../../_images/introduction_rpg_in_a_box.webp"><figcaption><p>RPG in a Box 是一款基于 Godot 开发的体素 RPG 编辑器，其节点化编程系统及整体界面均采用 Godot 的 UI 工具构建。<a href="https://docs.godotengine.org/zh-cn/4.x/getting_started/introduction/godot_design_philosophy.html#id1"></a></p></figcaption></figure>

将 `@tool` 注解放在任何 GDScript 文件的顶部，文件将在编辑器中运行。这样，你就可以导入、导出插件，创建自定义关卡编辑器之类的插件，或使用与项目中所使用的相同的节点和 API 来创建脚本。



