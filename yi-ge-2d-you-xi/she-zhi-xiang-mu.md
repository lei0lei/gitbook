# 设置项目

在这个简短的第一部分中，我们将设置和组织项目。

启动 Godot 然后新建一个项目。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/new-project-button.webp" alt=""><figcaption></figcaption></figure>

创建新项目时，你只需要选择一个有效的 _项目路径_。你可以保留其他默认设置。

{% hint style="info" %}
下载 [dodge\_the\_creeps\_2d\_assets.zip](https://github.com/godotengine/godot-docs-project-starters/releases/download/latest-4.x/dodge_the_creeps_2d_assets.zip)。归档文件中包含你会在制作游戏过程中使用到的图片和声音。请解压归档文件，将 `art/` 和 `fonts/` 目录移动到你的项目目录。
{% endhint %}

你的项目文件夹应如下所示。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/folder-content.webp" alt=""><figcaption></figcaption></figure>

本游戏设计为竖屏模式，因此我们需要调整游戏窗口的大小。点&#x51FB;_&#x9879;目 -> 项目设&#x7F6E;_&#x6253;开项目设置窗口，在左侧栏中打&#x5F00;_&#x663E;示 -> 窗&#x53E3;_&#x9009;项卡。在那里，将"视口宽度"设置为 `480`，"视口高度"设置为 `720`。你可以在左上角看到"项目"菜单。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/setting-project-width-and-height.webp" alt=""><figcaption></figcaption></figure>

另外，滚动到该小节的底部，在**拉伸**选项中，将**模式**设置为 `canvas_items`，将**比例**设置为 `keep`。这样就可以保证在不同大小的屏幕上，游戏都能够进行一致的比例缩放。

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/setting-stretch-mode.webp" alt=""><figcaption></figcaption></figure>

### 组织项目

在这个项目中，我们将制作 3 个独立的场景：`Player`、`Mob` 以及 `HUD`，我们将把这些场景合并成游戏的 `Main` 场景。

在更大的项目中，为各个场景及对应的脚本创建各自的文件夹会比较好。而这是一个相对小型的游戏，你可以把场景和脚本放在项目的根文件夹里，根文件夹用 `res://` 表示。可以在左下角的“文件系统”面板中查看项目文件夹：

<figure><img src="https://docs.godotengine.org/zh-cn/4.x/_images/filesystem_dock.webp" alt=""><figcaption></figcaption></figure>

项目准备就绪后，我们已准备好在下一课中设计玩家场景。







