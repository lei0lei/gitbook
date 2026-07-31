# Update Method

### 意图

通过让每个对象一次处理一帧行为，来模拟一组彼此独立的对象。

### 动机

玩家操控的女武神正踏上征途，要从已故法师王的尸骨上夺走璀璨宝石。她小心翼翼走近华丽陵墓入口，却遭到……什么攻击都没有。没有朝她放闪电的诅咒雕像，没有在门口巡逻的亡灵战士。她径直走进去，把战利品拿走。游戏结束，你赢了。

这可不行。

陵墓需要守卫——让勇敢女主角能与之搏斗的敌人。首先，我们想要一具复生的骷髅战士在门前来回巡逻。若抛开你对游戏编程已有的一切知识，让骷髅来回晃的最简单代码大概是这样：

> 若法师王想要更聪明的行为，当初就该复生一个还有脑组织的家伙。

```
// Some code
while (true)
{
  // 向右巡逻
  for (double x = 0; x < 100; x++)
  {
    skeleton.setX(x);
  }

  // 向左巡逻
  for (double x = 100; x > 0; x--)
  {
    skeleton.setX(x);
  }
}
```

问题当然是：骷髅确实在来回动，但玩家永远看不到。程序锁死在无限循环里，这可不是好玩的体验。我们真正想要的是：骷髅每帧只走一步。

于是必须去掉那些内层循环，改由外层游戏循环负责迭代。这样守卫巡逻时，游戏仍能响应输入并渲染。例如：

> [Game Loop](https://gameprogrammingpatterns.com/game-loop.html) 也是本书中的模式。

```
// Some code
Entity skeleton;
bool patrollingLeft = false;
double x = 0;

// 主游戏循环：
while (true)
{
  if (patrollingLeft)
  {
    x--;
    if (x == 0) patrollingLeft = false;
  }
  else
  {
    x++;
    if (x == 100) patrollingLeft = true;
  }

  skeleton.setX(x);

  // 处理用户输入并渲染游戏……
}
```

我做前后对比，是为了让你看到代码如何变复杂。原先左右巡逻是两个简单的 `for` 循环，方向由“当前执行哪段循环”隐含表示。现在每帧都要把控制权交回外层游戏循环，再从断点继续，就必须用 `patrollingLeft` 显式记录方向。

这大致能用，于是我们继续。一袋没脑子的骨头对北欧少女构不成太大挑战，于是再加几尊附魔雕像，偶尔朝她放闪电，让她保持警惕。

继续用“最简单怎么写”的思路，结果变成：

```
// Some code
// 骷髅相关变量……
Entity leftStatue;
Entity rightStatue;
int leftStatueFrames = 0;
int rightStatueFrames = 0;

// 主游戏循环：
while (true)
{
  // 骷髅代码……

  if (++leftStatueFrames == 90)
  {
    leftStatueFrames = 0;
    leftStatue.shootLightning();
  }

  if (++rightStatueFrames == 80)
  {
    rightStatueFrames = 0;
    rightStatue.shootLightning();
  }

  // 处理用户输入并渲染游戏……
}
```

看得出，这正走向我们不愿维护的代码：变量和命令式逻辑越堆越多，全塞进游戏循环，每个实体各写一摊。为让它们同时运行，只好把代码揉成一团。

只要“揉成一团”能准确形容你的架构，你多半就有问题了。

用来修复的模式简单到你大概已经想到了：游戏中每个实体应封装自己的行为。这样游戏循环保持干净，增删实体也容易。

为此需要一层抽象：定义抽象的 `update()` 方法。游戏循环维护一组对象，但不知道它们的具体类型，只知道它们都能被更新。这样每个对象的行为既与游戏循环解耦，也与其他对象解耦。

每帧，游戏循环遍历集合并对每个对象调用 `update()`，让每个对象执行一帧行为。对所有对象每帧都调用一次，它们就“同时”活动起来。

> 较真的人会指出：它们并非真正并发。一个对象在更新时，其他对象并不在更新。稍后会细说。

游戏循环用的是动态集合，增删关卡中的对象只需改集合。不再硬编码，甚至可用数据文件填充关卡——正是关卡设计师想要的。

### 模式

游戏世界维护一组对象。每个对象实现一个更新方法，用于模拟该对象一帧的行为。每一帧，游戏更新集合中的每一个对象。

### 何时使用

若说 [Game Loop](https://gameprogrammingpatterns.com/game-loop.html) 是切片面包，那 Update Method 就是抹在上面的黄油。大量包含玩家可交互“活实体”的游戏，都以某种形式使用这一模式。若游戏里有星际陆战队员、龙、火星人、幽灵或运动员，多半就用了它。

但若游戏更抽象，活动单位更像棋盘上的棋子而非活的角色，这个模式往往不太合适。如下棋：你不必并发模拟所有棋子，也大概不必每帧告诉兵“自己更新自己”。

即便棋类游戏不必每帧更&#x65B0;_&#x884C;为_，你仍可能每帧更&#x65B0;_&#x52A8;画_。这个模式对此也有帮助。

Update Method 适合以下情况：

* 游戏中有多个对象或系统需要同时运行
* 各对象行为大体相互独立
* 对象需要随时间被模拟

### 需要注意

模式很简单，暗坑不多，但每行代码都有后果。

#### 把代码切成单帧切片会更复杂

对比前两段代码，第二段明显更复杂。两者都只是让骷髅来回走，但第二段必须每帧把控制权交回游戏循环。

为处理输入、渲染及游戏循环负责的其他事，这一改动几乎总是必要的，所以第一个例子并不实用。但要记住：把行为代码切成这样，会有一笔不小的前期复杂度成本。

> 我说“几乎”，是因为有时可以两全其美：对象行为写成不返回的直线代码，同时多个对象并发运行并与游戏循环协调。

你需要能同时推进多条“执行线程”的系统。若对象代码能在中途暂停再恢复，而不是必须完全返回，就能用更命令式的写法。

真正的线程通常太重；若语言支持生成器、协程、纤程等轻量并发，或许能用。

[Bytecode](https://gameprogrammingpatterns.com/bytecode.html) 模式也可在应用层创建执行线程。

#### 每帧都要保存状态，才能从断点继续

第一段示例里，没有变量表示守卫向左还是向右——方向隐含在“当前执行哪段代码”里。

改成一次一帧后，就必须用 `patrollingLeft` 记录。从代码返回后，执行位置丢失，因此要显式存足够信息，好在下一帧恢复。

[State](https://gameprogrammingpatterns.com/state.html) 模式往往能帮忙。状态机在游戏里常见，部分原因正是（如其名）它们保存了“从断点继续”所需的状态。

#### 对象每帧都模拟，但并非真正并发

本模式中，游戏遍历对象集合并逐个更新。在 `update()` 里，多数对象能触达游戏世界其余部分，包括正在被更新的其他对象。因此更新顺序很重要。

若列表中 A 在 B 之前：A 更新时看到的是 B 的旧状态；B 更新时看到的是 A &#x7684;_&#x65B0;_&#x72B6;态（A 本帧已更新）。尽管在玩家看来一切同时移动，游戏核心仍是回合制——只是一整“回合”只有一帧长。

若&#x4F60;_&#x4E0D;&#x60F3;_&#x8FD9;种顺序语义，可用 [Double Buffer](https://gameprogrammingpatterns.com/double-buffer.html)：A、B 更新时都看到上一帧状态，顺序不再要紧。

对游戏逻辑而言，顺序更新多半是好事。并行更新会带来难缠的语义。想象黑白双方同时下棋，都想把棋子下到同一空格——如何裁决？

顺序更新解决了这点——每次更新把世界从一种合法状态渐进到下一种，中间没有模糊、需要事后调和的时段。

这也有利于联机：你有一串可序列化的操作，可经网络发送。

#### 更新时修改对象列表要小心

使用本模式时，大量游戏行为落在这些 `update` 里，其中常包括向游戏增删可更新对象。

例如，骷髅守卫被杀时掉落物品。新对象通常可加到列表末尾，问题不大：你会继续遍历，最终更新到新对象。

但这意味着新生对象在生成当帧就有机会行动，玩家甚至还没看见它。若不想如此，一个简单做法是：在更新循环开始时缓存对象数量，只更新那么多个就停：

```
// Some code
int numObjectsThisTurn = numObjects_;
for (int i = 0; i < numObjectsThisTurn; i++)
{
  objects_[i]->update();
}
```

这里 `objects_` 是可更新对象数组，`numObjects_` 是长度。添加新对象时长度会增加。我们在循环开头把长度缓存在 `numObjectsThisTurn`，这样迭代会在碰到本帧新增对象之前停下。

更棘手的是迭代&#x4E2D;_&#x5220;&#x9664;_&#x5BF9;象。你消灭了某邪兽，要从列表里扯掉它。若它排在当前正在更新的对象之前，可能意外跳过一个对象：

```
// Some code
for (int i = 0; i < numObjects_; i++)
{
  objects_[i]->update();
}
```

这个简单循环每轮递增索引。假设正在更新女主角时 `i` 为 1，她杀掉上方的邪兽，邪兽被移除，女主角移到 0，倒霉农民移到 1。女主角更新完后 `i` 增为 2——农民被跳过，本帧永不更新。

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

廉价做法是反向遍历更新：删除只移动已经更新过的项。

也可在删除时小心调整迭代变量；或把删除推迟到遍历结束后：先标为“死亡”，更新时跳过死亡对象，结束后再扫一遍清尸体。

若多线程处理更新列表中的项，更应推迟修改，以免更新期间做昂贵的线程同步。

### 示例代码

模式直白到示例几乎显得啰嗦。但这不说明它没用——正因其简单：干净解决问题，少有花活。

为保持具体，走一遍基本实现。先从表示骷髅与雕像的 `Entity` 类开始：

```
// Some code
class Entity
{
public:
  Entity()
  : x_(0), y_(0)
  {}

  virtual ~Entity() {}

  virtual void update() = 0;

  double x() const { return x_; }
  double y() const { return y_; }

  void setX(double x) { x_ = x; }
  void setY(double y) { y_ = y; }

private:
  double x_;
  double y_;
};
```

只放了后面需要的最少内容。真实代码大概还有图形、物理等。对本模式，关键是抽象的 `update()`。

游戏维护这些实体的集合。示例中放在表示游戏世界的类里：

```
// Some code
class World
{
public:
  World()
  : numEntities_(0)
  {}

  void gameLoop();

private:
  Entity* entities_[MAX_ENTITIES];
  int numEntities_;
};
```

真实项目多用集合类；这里用普通数组保持简单。

布置好后，游戏通过每帧更新每个实体实现该模式：

```
// Some code
void World::gameLoop()
{
  while (true)
  {
    // 处理用户输入……

    // 更新每个实体
    for (int i = 0; i < numEntities_; i++)
    {
      entities_[i]->update();
    }

    // 物理与渲染……
  }
}
```

方法名也表明，这是 [Game Loop](https://gameprogrammingpatterns.com/game-loop.html) 的例子。

#### 给实体做子类？！

有些读者此刻浑身发毛：我用继承主 `Entity` 类来定义不同行为。若你还不觉得有问题，我补充一点背景。

游戏业从 6502 汇编与 VBLANK 的原始海洋爬上面向对象语言的海岸后，开发者陷入架构潮流狂热。最大的潮流之一就是滥用继承——建起遮天蔽日的拜占庭式类层次。

事实证明那是糟糕主意，没人能维护巨型类层次而不让它崩塌。即便 Gang of Four 在 1994 年就写过：

> 优先使用对象组合，而非类继承。

私下说，我觉得钟摆已摆得离子类化太远。我通常避免继承，但教条&#x5730;_&#x4E0D;&#x7528;_&#x7EE7;承，和教条&#x5730;_&#x6EE5;&#x7528;_&#x7EE7;承一样糟。适度使用即可。

当这一认识在业界渗透后，出现的方案是 [Component](https://gameprogrammingpatterns.com/component.html) 模式。那时 `update()` 会在实体&#x7684;_&#x7EC4;&#x4EF6;_&#x4E0A;，而不在 `Entity` 本身。这样不必用复杂实体类层次来定义与复用行为，只需混搭组件。

若做真实游戏，我大概也会那样。但本章讲的是 `update()`，用最少活动部件展示它的最简单方式，就是把方法放在 `Entity` 上并做几个子类。

#### 定义实体

回到正题。最初动机是定义巡逻骷髅守卫，以及释放闪电的魔法雕像。先从骨头朋友开始：实现合适的 `update()`：

```
// Some code
class Skeleton : public Entity
{
public:
  Skeleton()
  : patrollingLeft_(false)
  {}

  virtual void update()
  {
    if (patrollingLeft_)
    {
      setX(x() - 1);
      if (x() == 0) patrollingLeft_ = false;
    }
    else
    {
      setX(x() + 1);
      if (x() == 100) patrollingLeft_ = true;
    }
  }

private:
  bool patrollingLeft_;
};
```

几乎就是把本章前面游戏循环里那块代码剪贴进 `Skeleton::update()`。小差别是 `patrollingLeft_` 变成字段而非局部变量，以便在多次 `update()` 调用间保持值。

雕像同理：

```
// Some code
class Statue : public Entity
{
public:
  Statue(int delay)
  : frames_(0),
    delay_(delay)
  {}

  virtual void update()
  {
    if (++frames_ == delay_)
    {
      shootLightning();

      // 重置计时器
      frames_ = 0;
    }
  }

private:
  int frames_;
  int delay_;

  void shootLightning()
  {
    // 发射闪电……
  }
};
```

大多仍是把代码从游戏循环挪进类并改名。但这次代码库真的更简单了：原先恶心的命令式代码里，每尊雕像各有帧计数与射速局部变量。

现在它们进了 `Statue` 类，想创建多少就能创建多少，每实例自带小计时器。这正是模式的动机——往游戏世界加新实体容易得多，因为每个实体自带照顾自己所需的一切。

该模式让我们&#x628A;_&#x586B;&#x5145;_&#x6E38;戏世界&#x4E0E;_&#x5B9E;&#x73B0;_&#x6E38;戏世界分开，从而可用独立数据文件或关卡编辑器来填充世界。

<figure><img src=".gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

#### 传递时间

核心模式如上；再提一个常见细化。目前我们假定每次 `update()` 以相同固定时间单位推进世界。

我个人更偏爱固定步长，但许多游戏&#x7528;_&#x53EF;变时间步长_：游戏循环每圈模拟的时间片，取决于上一帧处理与渲染花了多久。

[Game Loop](https://gameprogrammingpatterns.com/game-loop.html) 一章有更多固定/可变时间步长的利弊讨论。

这意味着每次 `update()` 需要知道虚拟时钟走了多远，因此常会传入经过时间。例如让巡逻骷髅支持可变时间步长：

```
// Some code
void Skeleton::update(double elapsed)
{
  if (patrollingLeft_)
  {
    x -= elapsed;
    if (x <= 0)
    {
      patrollingLeft_ = false;
      x = -x;
    }
  }
  else
  {
    x += elapsed;
    if (x >= 100)
    {
      patrollingLeft_ = true;
      x = 100 - (x - 100);
    }
  }
}
```

现在经过时间越大，骷髅移动距离越大。也能看到可变时间步长的额外复杂度：大时间片可能越过巡逻边界，必须小心处理。

### 设计决策

模式虽简单，变化不多，但仍有几个旋钮可调。

#### `update` 方法放在哪个类上？

最明显也最重要的决定：把 `update()` 放在哪。

* 实体类：\
  若已有实体类，这是最简单选项，不引入额外类。实体种类不多时或许可行，但业界整体在远离此做法。\
  每要一种新行为就子类化 `Entity`，在种类很多时脆弱痛苦。最终你会想以无法优雅映射到单继承层次的方式复用代码，然后就卡住了。
* 组件类：\
  若已在用 [Component](https://gameprogrammingpatterns.com/component.html)，这是无脑选择。每个组件可独立更新。正如 Update Method 让游戏世界中的实体彼此解耦，这也&#x8BA9;_&#x5355;个实体的各部&#x5206;_&#x5F7C;此解耦。渲染、物理、AI 可各自照顾自己。
* 委托类：\
  还有其他把部分行为委托给另一对象的模式。[State](https://gameprogrammingpatterns.com/state.html) 通过换委托对象改变行为；[Type Object](https://gameprogrammingpatterns.com/type-object.html) 让同“种”实体共享行为。\
  若用了这些，很自然把 `update()` 放在委托类上。主类上可能仍有非虚的 `update()`，只是转发给委托对象，例如：

```
// Some code
void Entity::update()
{
  // 转发给状态对象
  state_->update();
}
```

这样可通过更换委托对象定义新行为。与组件类似，无需全新子类即可改变行为。

#### 休眠对象如何处理？

世界中常有一批对象因故暂时不必更新：禁用、在屏外、尚未解锁等。若大量对象处于此状态，每帧走一遍却什么都不做，会浪费 CPU。

一种替代是单独维护只需更新的“活”对象集合。禁用时从中移除，重新启用时加回。这样只迭代真正有活干的项。

* 若用单个集合（含非活动对象）：
  * _浪费时间。_ 对非活动对象，要么检查“是否启用”标志，要么调用空方法。\
    除浪费检查与跳过的周期外，无意义遍历还会打爆数据缓存。CPU 通过把 RAM 加载到更快的片上缓存来优化读取，并推测你会读紧挨刚读位置的内存。跳过对象可能越过缓存边界，迫使再慢速拉取另一块主存。
* 若用仅含活动对象的单独集合：
  * _多用内存维护第二集合。_ 通常还有一份含全部实体的主集合；此时该集合技术上冗余。当速度比内存更紧（经常如此）时，仍可能值得。\
    也可两集合，另一份只&#x542B;_&#x975E;活&#x52A8;_&#x5B9E;体，而非全部。
  * _必须保持集合同步。_ 对象创建或彻底销毁（不只是临时休眠）时，要记得同时改主集合与活动集合。

指导原则是：非活动对象通常有多少。越多，就越值得用单独集合，避免在核心游戏循环里碰到它们。

### 另见

* 本模式与 [Game Loop](https://gameprogrammingpatterns.com/game-loop.html)、[Component](https://gameprogrammingpatterns.com/component.html) 常构成游戏引擎的核心三件套。
* 当开始关心每帧循环更新大量实体/组件的缓存性能时，[Data Locality](https://gameprogrammingpatterns.com/data-locality.html) 可帮忙加速。
* [Unity](http://unity3d.com/) 在多个类中使用此模式，包括 [`MonoBehaviour`](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)。
* Microsoft [XNA](http://creators.xna.com/en-US/) 在 [`Game`](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.game.update.aspx) 与 [`GameComponent`](http://msdn.microsoft.com/en-us/library/microsoft.xna.framework.gamecomponent.update.aspx) 中使用。
* [Quintus](http://html5quintus.com/) JavaScript 引擎在主 [`Sprite`](http://html5quintus.com/guide/sprites.md) 类上使用此模式。

