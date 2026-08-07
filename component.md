# Component

### 意图

让单一实体跨越多个领域，同时不让这些领域彼此耦合。

### 动机

假设我们在做平台跳跃游戏。意大利水管工人群已经有人做了，所以我们的主角是丹麦面包师 Bjørn。合理推断会有一个类代表这位友好的糕点师，并包含他在游戏里做的一切。

> 这种绝妙游戏点子，正是我当程序员而不是设计师的原因。

玩家操控他，就要读手柄输入并转成运动。当然，他还要与关卡交互，于是物理与碰撞也要塞进去。搞定后，他还要出现在屏幕上，于是再扔进动画与渲染。大概还要播声音。

等一下；这失控了。软件架构入门告诉我们：程序中不同领域应彼此隔离。做文字处理器时，负责打印的代码不该受加载保存文档的代码影响。游戏的领域与商业应用不同，但规则一样。

尽可能地，我们不想让 AI、物理、渲染、声音等领域互相知晓，可现在全塞进一个类了。我们知道这条路通向何处：一个 5000 行的倾倒场源文件，大到只有团队里最勇敢的忍者之代码者才敢进去。

对少数驯服得了它的人，这是绝佳铁饭碗；对其余我们来说是地狱。类大到连看似最琐碎的改动都可能影响深远。很快，类收集 _bug_ 的速度会超过收集 _功能_ 的速度。

#### 戈尔迪之结

比单纯规模问题更糟的是耦合。游戏里所有不同系统被系成一团巨大的代码死结，像这样：

```
// Some code
if (collidingWithFloor() && (getRenderState() != INVISIBLE))
{
  playSound(HIT_FLOOR);
}
```

任何想改这类代码的程序员，都得懂一点物理、图形和声音，才能确保自己不弄坏东西。

这种耦合&#x5728;_&#x4EFB;&#x4F55;_&#x6E38;戏里都糟糕，在使用并发的现代游戏中更糟。多核硬件上，关键是让代码在多个线程同时运行。把游戏拆到线程上的常见做法是按领域边界——AI 一核、声音一核、渲染一核，等等。

一旦这么做，这些领域必须保持解耦，以免死锁或其他凶恶并发 bug。单独一个类里既有必须从一线程调用的 `UpdateSounds()`，又有必须从另一线程调用的 `RenderGraphics()`，就是在求这种 bug 上门。

这两个问题互相放大：类碰的领域太多，每个程序员都得碰它；它又大到碰它是噩梦。若糟糕到一定程度， coder 会开始在代码库其它地方打补丁，只为躲开 `Bjorn` 类变成的那团毛球。

#### 斩断死结

我们可以像亚历山大大帝那样——用剑解决。把单体 `Bjorn` 类沿领域边界切开。例如，把处理用户输入的代码全部挪进单独的 `InputComponent` 类，再由 `Bjorn` 拥有该组件的实例。对 `Bjorn` 触及的每个领域重复此过程。

做完后，几乎一切都从 `Bjorn` 搬走了。剩下的只是把组件绑在一起的薄壳。通过拆成多个更小的类，解决了巨型类问题——但成就不止于此。

#### 松开的线头

组件类现在彼此解耦。即便 `Bjorn` 有 `PhysicsComponent` 和 `GraphicsComponent`，两者也不知道对方。于是搞物理的人改自己的组件时不必懂图形，反之亦然。

实践中，组件之间仍&#x9700;_&#x4E00;&#x4E9B;_&#x4EA4;互。例如，AI 组件可能要告诉物理组件 Bjørn 想去哪。但我们可把交互限制&#x5728;_&#x786E;&#x5B9E;_&#x9700;要交谈的组件之间，而不是全扔进同一个游乐场。

#### 重新系上

这设计的另一个特性是：组件成了可复用包。迄今我们聚焦面包师；再想想游戏世界里另外几类对象。_装饰&#x7269;_&#x662F;玩家看得见却不交互的东西：灌木、碎片及其他视觉细节。_道&#x5177;_&#x50CF;装饰物但可碰：箱子、巨石、树。_区&#x57DF;_&#x662F;装饰物的反面——看不见但可交互，适合例如 Bjørn 进入某区触发过场。

面向对象初次走红时，继承是工具箱里最闪亮的锤子。它被视为代码复用的终极神器，人们经常抡它。后来我们惨痛地学到，这锤确实很重。继承有用处，但对简单代码复用往往太笨重。

软件设计里上升的趋势是：尽量用组合而非继承。不通过两个&#x7C7B;_&#x7EE7;&#x627F;_&#x81EA;同一类来共享代码，而是让它们&#x90FD;_&#x62E5;&#x6709;_&#x540C;一类的实例。

若不使用组件，这些类的继承层次初稿可能是：`Zone` 从 `GameObject` 继承并加碰撞；`Decoration` 继承并加渲染；`Prop` 继承自 `Zone` 以复用碰撞，却无法再继承 `Decoration` 复用渲染而不撞上致命钻石。

> “致命钻石”出现在有多重继承、通向同一基类有两条路径的类层次里。痛苦超出本书范围，但要知道他们叫它“致命”是有原因的。

可把东西反过来让 `Prop` 继承 `Decoration`，但那又得复&#x5236;_&#x78B0;&#x649E;_&#x4EE3;码。无论怎样，没有干净方式在需要的类之间复用碰撞与渲染，又不求助于多重继承。唯一其它选项是全推到 `GameObject`，那 `Zone` 就会在它不需要的渲染数据上浪费内存，`Decoration` 在物理上同理。

再用组件试试。子类完全消失。只有一个 `GameObject` 类，加两个组件类：`PhysicsComponent` 与 `GraphicsComponent`。装饰物是只有 `GraphicsComponent`、没有 `PhysicsComponent` 的 `GameObject`。区域相反，道具两个都有。无代码重复，无多重继承，四个类变成三个。

> 餐厅菜单是好类比。若每个实体是单体类，就像只能点套餐——每种功&#x80FD;_&#x7EC4;&#x5408;_&#x90FD;要一个单独的类。要伺候每位顾客，需要几十种套餐。\
> 组件是单点菜单——每位顾客可选自己想要的菜，菜单列出他们能选的菜品。

组件基本上是对象的即插即用。通过把不同可复用组件插进实体上的插槽，就能构建行为丰富的复杂实体。想想软件版伏尔甘（Voltron）。

### 模式

单一实体跨越多个领域。为保持领域隔离，各领域代码放进各自的组件类。实体缩减为简单的组件容器。

> “Component”像“Object”一样，在编程中什么都指、又什么都不指。因此被用来描述过几种概念。企业软件里有一种“Component”设计模式，描述经 Web 通信的解耦服务。\
> 我试过给游戏里这个无关模式另起名，但“Component”似乎是最常见术语。设计模式关乎记录既有实践，我没有发明新词的余裕。于是追随 XNA、Delta3D 等，就叫“Component”。

### 何时使用

组件最常见于定义游戏实体的核心类，但其它地方也可能有用。下列任一成立时，这模式就用得好：

* 有一个类触及多个你想保持解耦的领域。
* 类变得巨大难搞。
* 你想定义多种共享不同能力的对象，但继承无法足够精确地挑出你想复用的部分。

### 需要注意

相对简单建类并往里塞代码，组件模式多了不少复杂度。每个概念“对象”变成一簇必须实例化、初始化并正确接线的对象。组件间通信更难，控制它们如何占用内存也更复杂。

对大型代码库，这复杂度或许值得，因换来了解耦与代码复用；但应用前要确保自己不是在给不存在的问题过度工程“解法”。

另一后果是：做事常要多跳一层间接。有了容器对象，先要拿到想要的组件，_&#x518D;_&#x80FD;做需要做的事。在性能关键内环里，跟着指针走可能导致性能变差。

硬币也有另一面。组件模式往往&#x80FD;_&#x63D0;&#x5347;_&#x6027;能与缓存亲和性。组件让更容易用 [Data Locality](https://gameprogrammingpatterns.com/data-locality.html) 按 CPU 想要的顺序组织数据。

### 示例代码

写这本书最大挑战之一是弄清如何把每个模式隔离。许多设计模式存在是为了容纳本身并非模式一部分的代码。为把模式蒸馏到本质，我尽量砍掉那些，但某点就像解释如何整理衣柜却不能展示任何衣服。

组件模式特别难。看不到它解耦的各领域一些代码，就很难真正感受它，所以我得多画一点 Bjørn 的代码，多过我本想的。模式其实只&#x662F;_&#x7EC4;件&#x7C7B;_&#x672C;身，但其中代码应能澄清类是干什么的。是假代码——调了这里未呈现的其它类——但应能让你明白我们的目标。

#### 单体类

为更清楚看模式如何应用，先展&#x793A;_&#x4E0D;_&#x7528;这模式、却做完一切所需的单体 `Bjorn`：

> 应指出：在代码库里用角色真名通常是坏主意。市场部有烦人习惯，发售前几天就要求改名。“焦点小组显示 11–15 岁男性对‘Bjørn’负反馈。改用‘Sven’。”\
> 这也是许多软件项目用仅内部代号的原因。当然还有：告诉人你在搞“Big Electric Cat”，比说“下一版 Photoshop”好玩。

```
// Some code
class Bjorn
{
public:
  Bjorn()
  : velocity_(0),
    x_(0), y_(0)
  {}

  void update(World& world, Graphics& graphics);

private:
  static const int WALK_ACCELERATION = 1;

  int velocity_;
  int x_, y_;

  Volume volume_;

  Sprite spriteStand_;
  Sprite spriteWalkLeft_;
  Sprite spriteWalkRight_;
};
```

`Bjorn` 有每帧由游戏调用的 `update()`：

```
// Some code
void Bjorn::update(World& world, Graphics& graphics)
{
  // 把用户输入应用到英雄速度
  switch (Controller::getJoystickDirection())
  {
    case DIR_LEFT:
      velocity_ -= WALK_ACCELERATION;
      break;

    case DIR_RIGHT:
      velocity_ += WALK_ACCELERATION;
      break;
  }

  // 用速度修改位置
  x_ += velocity_;
  world.resolveCollision(volume_, x_, y_, velocity_);

  // 画合适的精灵
  Sprite* sprite = &spriteStand_;
  if (velocity_ < 0)
  {
    sprite = &spriteWalkLeft_;
  }
  else if (velocity_ > 0)
  {
    sprite = &spriteWalkRight_;
  }

  graphics.draw(*sprite, x_, y_);
}
```

它读摇杆决定如何加速面包师，再用物理引擎解析新位置，最后把 Bjørn 画到屏幕上。

这里的示例实现极简：无重力、无动画，也没有让角色好玩的其它几十处细节。即便如此，也能看出单个函数里团队好几个 coder 大概都得花时间，而且开始变乱。想象放大到千行，就能体会有多痛苦。

#### 拆出一个领域

从一个领域开始，从 `Bjorn` 拉出一块推进单独组件类。从最先处理的领域——输入——开始。`Bjorn` 第一件事就是读用户输入并据此调速度。把那逻辑挪到单独类：

```
// Some code
class InputComponent
{
public:
  void update(Bjorn& bjorn)
  {
    switch (Controller::getJoystickDirection())
    {
      case DIR_LEFT:
        bjorn.velocity -= WALK_ACCELERATION;
        break;

      case DIR_RIGHT:
        bjorn.velocity += WALK_ACCELERATION;
        break;
    }
  }

private:
  static const int WALK_ACCELERATION = 1;
};
```

相当简单：拿了 `Bjorn::update()` 的第一段放进这个类。`Bjorn` 的改动也直白：

```
// Some code
class Bjorn
{
public:
  int velocity;
  int x, y;

  void update(World& world, Graphics& graphics)
  {
    input_.update(*this);

    // 用速度修改位置
    x += velocity;
    world.resolveCollision(volume_, x, y, velocity);

    // 画合适的精灵
    Sprite* sprite = &spriteStand_;
    if (velocity < 0)
    {
      sprite = &spriteWalkLeft_;
    }
    else if (velocity > 0)
    {
      sprite = &spriteWalkRight_;
    }

    graphics.draw(*sprite, x, y);
  }

private:
  InputComponent input_;

  Volume volume_;

  Sprite spriteStand_;
  Sprite spriteWalkLeft_;
  Sprite spriteWalkRight_;
};
```

`Bjorn` 现在拥有一个 `InputComponent`。先前在 `update()` 里直接处理用户输入，现在委托给组件：

```
// Some code
input_.update(*this);
```

才刚开始，已去掉一些耦合——主 `Bjorn` 类不再引用 `Controller`。后面会派上用场。

#### 拆出其余

继续对物理与图形代码做同样的剪切粘贴。新的 `PhysicsComponent`：

```
// Some code
class PhysicsComponent
{
public:
  void update(Bjorn& bjorn, World& world)
  {
    bjorn.x += bjorn.velocity;
    world.resolveCollision(volume_,
        bjorn.x, bjorn.y, bjorn.velocity);
  }

private:
  Volume volume_;
};
```

除了把物&#x7406;_&#x884C;&#x4E3A;_&#x632A;出主 `Bjorn` 类，你也看&#x5230;_&#x6570;&#x636E;_&#x642C;走了：`Volume` 对象现在由组件拥有。

最后，渲染代码现在住在：

```
// Some code
class GraphicsComponent
{
public:
  void update(Bjorn& bjorn, Graphics& graphics)
  {
    Sprite* sprite = &spriteStand_;
    if (bjorn.velocity < 0)
    {
      sprite = &spriteWalkLeft_;
    }
    else if (bjorn.velocity > 0)
    {
      sprite = &spriteWalkRight_;
    }

    graphics.draw(*sprite, bjorn.x, bjorn.y);
  }

private:
  Sprite spriteStand_;
  Sprite spriteWalkLeft_;
  Sprite spriteWalkRight_;
};
```

几乎全抽走了，我们谦逊的糕点师还剩什么？不多：

```
// Some code
class Bjorn
{
public:
  int velocity;
  int x, y;

  void update(World& world, Graphics& graphics)
  {
    input_.update(*this);
    physics_.update(*this, world);
    graphics_.update(*this, graphics);
  }

private:
  InputComponent input_;
  PhysicsComponent physics_;
  GraphicsComponent graphics_;
};
```

`Bjorn` 类现在基本上干两件事：持有真正定义它的那组组件，以及跨多个领域共享的状态。位置与速度仍在核心 `Bjorn` 类里，原因有二。第一，它们是“跨领域”状态——几乎每个组件都会用，若想往下推，也不清&#x695A;_&#x54EA;&#x4E2A;_&#x7EC4;件该拥有。

第二、更重要的是：它给组件在不彼此耦合的情况下通信的便捷途径。看看能否加以利用。

#### 机器 Bjørn

迄今我们把行为推到单独组件类，但还没把行&#x4E3A;_&#x62BD;&#x8C61;_&#x51FA;来。`Bjorn` 仍知道行为定义在确切哪些具体类。改一下。

把处理用户输入的组件藏在接口后。把 `InputComponent` 变成抽象基类：

```
// Some code
class InputComponent
{
public:
  virtual ~InputComponent() {}
  virtual void update(Bjorn& bjorn) = 0;
};
```

再把现有用户输入处理代码压进实现该接口的类：

```
// Some code
class PlayerInputComponent : public InputComponent
{
public:
  virtual void update(Bjorn& bjorn)
  {
    switch (Controller::getJoystickDirection())
    {
      case DIR_LEFT:
        bjorn.velocity -= WALK_ACCELERATION;
        break;

      case DIR_RIGHT:
        bjorn.velocity += WALK_ACCELERATION;
        break;
    }
  }

private:
  static const int WALK_ACCELERATION = 1;
};
```

改 `Bjorn` 持有输入组件指针，而非内联实例：

```
// Some code
class Bjorn
{
public:
  int velocity;
  int x, y;

  Bjorn(InputComponent* input)
  : input_(input)
  {}

  void update(World& world, Graphics& graphics)
  {
    input_->update(*this);
    physics_.update(*this, world);
    graphics_.update(*this, graphics);
  }

private:
  InputComponent* input_;
  PhysicsComponent physics_;
  GraphicsComponent graphics_;
};
```

实例化 `Bjorn` 时传入要用的输入组件：

```
// Some code
Bjorn* bjorn = new Bjorn(new PlayerInputComponent());
```

该实例可以是实现抽象 `InputComponent` 接口的任何具体类型。我们付出代价——`update()` 现在是虚调用，稍慢一点。回报是什么？

> 多数主机要求游戏支持“演示模式”。玩家在主菜单坐着不动，游戏会自动开玩，由电脑代打。这防止主菜单烧屏进电视，也让店里展柜上的游戏更好看。

把输入组件类藏在接口后，就能做到。已有正常玩游戏时用的具体 `PlayerInputComponent`。再做一个：

```
// Some code
class DemoInputComponent : public InputComponent
{
public:
  virtual void update(Bjorn& bjorn)
  {
    // 自动控制 Bjorn 的 AI……
  }
};
```

游戏进入演示模式时，不再像先前那样构造 Bjørn，而是接上新组件：

```
// Some code
Bjorn* bjorn = new Bjorn(new DemoInputComponent());
```

只需换一个组件，就有完整可用的电脑代打玩家。能复用 Bjørn 的其它全部代码——物理与图形甚至不知道有差别。或许我有点怪，但这种事让我早上起床。还有咖啡。香甜滚热的咖啡。

#### 完全没有 Bjørn？

看现在的 `Bjorn` 类，会发现没什么真“Bjørn”的东西——它只是组件袋。实际上，它很像可用于游戏&#x4E2D;_&#x6BCF;&#x4E2A;_&#x5BF9;象的基“游戏对象”类。只需传&#x5165;_&#x5168;&#x90E8;_&#x7EC4;件，就能像科学怪人一样挑选零件构建任意对象。

把剩下两个具体组件——物理与图形——也像输入一样藏在接口后：

```
// Some code
class PhysicsComponent
{
public:
  virtual ~PhysicsComponent() {}
  virtual void update(GameObject& obj, World& world) = 0;
};

class GraphicsComponent
{
public:
  virtual ~GraphicsComponent() {}
  virtual void update(GameObject& obj, Graphics& graphics) = 0;
};
```

再把 `Bjorn` 更名为使用这些接口的通用 `GameObject`：

```
// Some code
class GameObject
{
public:
  int velocity;
  int x, y;

  GameObject(InputComponent* input,
             PhysicsComponent* physics,
             GraphicsComponent* graphics)
  : input_(input),
    physics_(physics),
    graphics_(graphics)
  {}

  void update(World& world, Graphics& graphics)
  {
    input_->update(*this);
    physics_->update(*this, world);
    graphics_->update(*this, graphics);
  }

private:
  InputComponent* input_;
  PhysicsComponent* physics_;
  GraphicsComponent* graphics_;
};
```

> 有些组件系统走得更远。不是 `GameObject` 包含其组件，游戏实体只是一个 ID、一个数字。然后维护单独的组件集合，每个组件知道它所附实体的 ID。\
> 这些 [实体组件系统](http://en.wikipedia.org/wiki/Entity_component_system) 把组件解耦推到极致，可在实体甚至不知道的情况下给实体加新组件。[Data Locality](https://gameprogrammingpatterns.com/data-locality.html) 一章有更多细节。

现有具体类会改名并实现那些接口：

```
// Some code
class BjornPhysicsComponent : public PhysicsComponent
{
public:
  virtual void update(GameObject& obj, World& world)
  {
    // 物理代码……
  }
};

class BjornGraphicsComponent : public GraphicsComponent
{
public:
  virtual void update(GameObject& obj, Graphics& graphics)
  {
    // 图形代码……
  }
};
```

现在可构建拥有 Bjørn 全部原行为的对象，而不必真为他建类：

```
// Some code
GameObject* createBjorn()
{
  return new GameObject(new PlayerInputComponent(),
                        new BjornPhysicsComponent(),
                        new BjornGraphicsComponent());
}
```

> 这 `createBjorn()` 当然是经典 Gang of Four [Factory Method](http://c2.com/cgi/wiki?FactoryMethod) 模式的例子。

定义其它用不同组件实例化 `GameObject` 的函数，就能创建游戏需要的所有不同对象种类。

### 设计决策

本模式最重要的设计问题是：“我需要哪套组件？”答案取决于游戏需求与类型。引擎越大越复杂，你大概越想把组件切得更细。

除此之外，还有几个更具体选项要考虑：

#### 对象如何获得其组件？

把单体对象拆成几个独立组件部分后，要决定谁把零件装回去。

* 若对象自己创建组件：
  * _确保对象总有所需组件。_ 不必担心有人忘了接对组件而弄坏游戏。容器对象自己照料。
  * _更难重新配置对象。_ 本模式的强大特性之一是仅靠重组组件就能构建新种类对象。若对象总用同一套硬编码组件连接自己，就没用上那份灵活。
* 若由外部代码提供组件：
  * _对象更灵活。_ 给它不同组件，就能彻底改变行为。推到极致，对象变成可反复复用的通用组件容器。
  * _对象可与具体组件类型解耦。_ 若允许外部传入组件，多半也允许传&#x5165;_&#x6D3E;&#x751F;_&#x7EC4;件类型。此时对象只知道组&#x4EF6;_&#x63A5;口_，不知具体类型。可形成封装良好的架构。

#### 组件之间如何通信？

完美解耦、孤立运行的组件是美好理想，实践中并不真能。这些组件属&#x4E8E;_&#x540C;&#x4E00;_&#x5BF9;象，意味着它们是更大整体的一部分，需要协调。于是就要通信。

组件如何交谈？有几个选项；与本书多数设计“备选”不同，这些并不互斥——设计中很可能同时支持多于一种。

* 通过修改容器对象的状态：
  * _保持组件解耦。_ `InputComponent` 设了 Bjørn 速度、`PhysicsComponent` 之后使用它时，两个组件完全不知道对方存在。对它们来说，速度可能是黑魔法改的。
  * _要求组件需共享的任何信息都推到容器对象上。_ 常有状态其实只被组件子集需要。例如，动画与渲染组件可能共享图形特有信息。把它推&#x5230;_&#x6BCF;&#x4E2A;_&#x7EC4;件都能取到的容器对象，会弄脏对象类。\
    更糟的是，若同一容器类用不同组件配置，可能&#x5728;_&#x4EFB;&#x4F55;_&#x7EC4;件都不需要的状态上浪费内存。把某些渲染特有数据推进容器，任何不可见对象都会白白占内存。
  * _让通信变得隐式，并依赖组件处理顺序。_ 示例中，原先单体 `update()` 有非常精心安排的操作顺序：用户输入改速度，物理用它改位置，渲染用它把 Bjørn 画对位置。拆成组件时，我们小心保留了那顺序。\
    若不保留，就会引入微妙、难追踪的 bug。例如，&#x82E5;_&#x5148;_&#x66F4;新图形组件，就会错误地&#x7528;_&#x4E0A;&#x4E00;_&#x5E27;而非本帧的位置渲染 Bjørn。想象更多组件和更多代码，就能体会这种 bug 有多难避免。\
    共享可变状态——大量代码读写同一数据——出了名地难做对。这也是学者花时间研究 Haskell 等根本没有可变状态的纯函数式语言的一个重要原因。
* 通过彼此直接引用：\
  想法是：需要交谈的组件直接持有对彼此的引用，完全不必经容器对象。\
  假设想让 Bjørn 跳跃。图形代码需知道该不该用跳跃精灵画。可通过问物理引擎他是否在地面上来判定。简单做法是让图形组件直接知道物理组件：

```
// Some code
class BjornGraphicsComponent
{
public:
  BjornGraphicsComponent(BjornPhysicsComponent* physics)
  : physics_(physics)
  {}

  void Update(GameObject& obj, Graphics& graphics)
  {
    Sprite* sprite;
    if (!physics_->isOnGround())
    {
      sprite = &spriteJump_;
    }
    else
    {
      // 已有图形代码……
    }

    graphics.draw(*sprite, obj.x, obj.y);
  }

private:
  BjornPhysicsComponent* physics_;

  Sprite spriteStand_;
  Sprite spriteWalkLeft_;
  Sprite spriteWalkRight_;
  Sprite spriteJump_;
};
```

构造 Bjørn 的 `GraphicsComponent` 时，给它对应 `PhysicsComponent` 的引用。

* _简单且快。_ 通信是对象间直接方法调用。组件可调它持有引用的组件支持的任何方法。自由混战。
* _两个组件紧耦合。_ 自由混战的坏处。我们基本退回单体类一步。又不像原先单个类那么糟，因为至少把耦合限制在需要交互的那些组件对。
* 通过发消息：
  * 这是最复杂的备选。可在容器对象里建个小消息系统，让组件彼此广播信息。\
    一种可能实现：先定义所有组件要实现的基 `Component` 接口：

```
// Some code
class Component
{
public:
  virtual ~Component() {}
  virtual void receive(int message) = 0;
};
```

只有一个 `receive()`，组件类实现它以监听入站消息。这里只用 `int` 标识消息；更完整实现可为消息附额外数据。\
再给容器对象加发消息方法：

```
// Some code
class ContainerObject
{
public:
  void send(int message)
  {
    for (int i = 0; i < MAX_COMPONENTS; i++)
    {
      if (components_[i] != NULL)
      {
        components_[i]->receive(message);
      }
    }
  }

private:
  static const int MAX_COMPONENTS = 10;
  Component* components_[MAX_COMPONENTS];
};
```

若组件能访问其容器，就可向容器发消息，容器再重播给所有所含组件。（包括原发送组件；小心别掉进反馈环！）这带来一些后果：

> 若真想花哨，甚至可让消息系&#x7EDF;_&#x6392;&#x961F;_&#x7A0D;后投递。更多见 [Event Queue](https://gameprogrammingpatterns.com/event-queue.html)。

* _兄弟组件解耦。_ 与共享状态备选一样，经父容器对象，确保组件仍彼此解耦。在这系统里，它们唯有的耦合是消息值本身。\
  Gang of Four 称之为 [Mediator](http://c2.com/cgi-bin/wiki?MediatorPattern) 模式——两个或多个对象经中间对象间接通信。此处容器对象本身就是中介。
* _容器对象简单。_ 不同于共享状态里容器自己拥有并知晓组件所用数据，这里它只盲目转发消息。这在让两个组件传递非常领域特有信息而不渗进容器时很有用。

毫不意外，这里没有唯一最佳答案。你多半会三种都用一点。共享状态适合可默认每个对象都有的真正基础东西——位置、尺寸等。

有些领域独立却仍紧密相关。想想动画与渲染、用户输入与 AI、或物理与碰撞。若这些对半各有单独组件，或许最容易让它们直接知道另一半。

消息适合“次要”通信。其发后即忘的性质，很适合物理组件发“撞到东西”消息时、音频组件播声音这类事。

一如既往：建议从简单开始，需要时再加额外通信路径。

### 另见

* [Unity](http://unity3d.com/) 框架的核心 [`GameObject`](http://docs.unity3d.com/Documentation/Manual/GameObjects.html) 类完全围绕 [组件](http://docs.unity3d.com/Manual/UsingComponents.html) 设计。
* 开源 [Delta3D](http://www.delta3d.org/) 引擎有实现本模式的基 `GameActor` 类，配套基类恰当地叫 `ActorComponent`。
* Microsoft [XNA](http://creators.xna.com/en-US/) 游戏框架带有核心 `Game` 类，拥有 `GameComponent` 对象集合。我们的示例在单个游戏实体层用组件；XNA 在主游戏对象层实现此模式，但目的相同。
* 本模式与 Gang of Four 的 [Strategy](http://c2.com/cgi/wiki?StrategyPattern) 有相似之处。两者都把对象行为的一部分委托给单独的从属对象。差别是：Strategy 的单独“策略”对象通常无状态——封装算法，但不含数据。它定义对&#x8C61;_&#x5982;&#x4F55;_&#x884C;为，而&#x975E;_&#x662F;_&#x4EC0;么。\
  组件更有自我存在感。它们常持有描述对象、帮助定义其真实身份的状态。不过界线可能模糊。可能有些组件不需要任何局部状态。那时可对多个容器对象复用同一组&#x4EF6;_&#x5B9E;例_。此时行为更接近策略。

***

核心一句话：把跨 AI/物理/渲染等领域的单体实体拆成可插拔组件；实体变成组件容器，领域彼此解耦，还能用组合拼出道具、装饰、区域等不同对象。

