# Subclass Sandbox

### 意图

用基类提供的一组操作，在子类中定义行为。

### 动机

每个小孩都梦过当超级英雄，可惜地球上宇宙射线紧缺。能让你假装超级英雄的游戏是最接近的替代。因为我们的游戏设计师从没学会说“不”，_我们&#x7684;_&#x8D85;级英雄游戏打算塞进几十乃至上百种可选超能力。

计划是做个 `Superpower` 基类，再为每种超能力做一个派生类。把设计文档分给一队程序员然后开写。做完就有上百个超能力类。

> 当你像本例这样&#x6709;_&#x5F88;&#x591A;_&#x5B50;类时，往往说明数据驱动更合适。别用大&#x91CF;_&#x4EE3;&#x7801;_&#x5B9A;义不同能力，试着改&#x7528;_&#x6570;&#x636E;_&#x5B9A;义行为。\
> [Type Object](https://gameprogrammingpatterns.com/type-object.html)、[Bytecode](https://gameprogrammingpatterns.com/bytecode.html)、[Interpreter](http://en.wikipedia.org/wiki/Interpreter_pattern) 等都有帮助。

我们想让玩家沉浸在充满变化的世界里。他们小时候梦见的任何能力，都希望进游戏。于是这些超能力子类几乎什么都能做：播声音、生成视觉效果、与 AI 交互、创建销毁游戏实体、搅和物理——代码库没有它们够不着的角落。

假设我们放手让团队写超能力类，会发生什么？

* _大量冗余代码。_ 不同能力再千差万别，仍会有不少重叠。许多会以相同方式生成视觉效果和播声音。冷冻射线、加热射线和第戎芥末射线，细究起来都相当类似。若实现的人互不协调，就会有大量重复代码和重复劳动。
* _引擎每个部分都会和这些类耦合。_ 不知情时，人们会写出直接调用本不该与超能力类绑死的子系统的代码。就算渲染器用几层整洁结构组织，只有一层本打算给图形引擎外的代码用，我们也敢打赌超能力代码最终会戳进每一层。
* _外部系统要改时，很可能某段随机的超能力代码就挂了。_ 一旦不同超能力类各自耦合到引擎的乱七八糟部分，那些系统的改动必然冲击能力类。这不好玩——图形、音频、UI 程序员大概也不想顺带当玩法程序员。
* _很难定义所有超能力都遵守的不变量。_ 比方说我们想确保能力播放的所有音频都正确排队并按优先级处理。若上百个类各自直接调用声音引擎，就很难做到。

我们想给每个实现超能力的玩法程序员一套可把玩的原语。想播声音？这里有 `playSound()`。想要粒子？这里有 `spawnParticles()`。我们确保这些操作覆盖你需要做的一切，这样你就不必 `#include` 随机头文件、把鼻子伸进其余代码库。

做法是：把这些操作做成 `Superpower` 基类的 _protected 方法_。放在基类里，让每个能力子类都能直接、方便地访问。标成 protected（往往还是非虚）表明：它们存在的目的就是被子&#x7C7B;_&#x8C03;用_。

有了这些玩具，还需要使用它们的地方。为此定义一&#x4E2A;_&#x6C99;箱方法_：子类必须实现的抽象 protected 方法。有了这些，实现一种新能力只需：

1. 新建一个继承自 `Superpower` 的类。
2. 重写沙箱方法 `activate()`。
3. 在方法体里调用 `Superpower` 提供的 protected 方法。

冗余代码问题可以这样修：把提供的操作做得尽量高层。看到许多子类之间重复的代码，总能把它卷进 `Superpower`，成为大家都能用的新操作。

耦合问题也解决了：耦合被约束在一处。`Superpower` 本身会和不同游戏系统耦合，但上百个派生类不会——它&#x4EEC;_&#x53EA;_&#x548C;基类耦合。某个游戏系统变了，可能要改 `Superpower`，但几十个子类不必动。

这模式带来浅而宽的类层次：继承链&#x4E0D;_&#x6DF1;_，但从 `Superpower` 挂下去的&#x7C7B;_&#x5F88;多_。有一个带大量直接子类的类，就在代码库里有了一个杠杆点。我们花在 `Superpower` 上的时间和心血，能惠及游戏中一大批类。

> 近来很多人批评面向对象语言里的继承。继承确实有问题——代码库里没有比基类与子类之间更深的耦合了——但我发&#x73B0;_&#x5BBD;_&#x7EE7;承树&#x6BD4;_&#x6DF1;_&#x7684;更好对付。

### 模式

基类定义抽象的沙箱方法以及若干提供的操作。标成 protected 表明它们供派生类使用。每个派生的沙箱子类用这些提供的操作实现沙箱方法。

### 何时使用

Subclass Sandbox 非常简单、常见，潜伏在许多代码库里，甚至游戏之外也有。若你有非虚的 protected 方法躺着，大概已经在用类似东西。适合以下情况：

* 有一个基类和若干派生类。
* 基类能提供派生类可能需要执行的全部操作。
* 子类之间有行为重叠，你想更容易共享代码。
* 你想尽量降低这些派生类与程序其余部分的耦合。

### 需要注意

如今“继承”在很多编程圈子是脏词，一个原因是基类倾向于越积越多代码。本模式对此特别敏感。

子类要经基类才能到达游戏其余部分，于是基类会&#x548C;_&#x4EFB;&#x610F;_&#x6D3E;生类需要交谈的每个系统耦合。当然，子类也与基类紧密相连。这张耦合蛛网让改基类而不弄坏什么变得很难——你碰上了[脆弱基类问题](http://en.wikipedia.org/wiki/Fragile_base_class)。

硬币另一面是：多数耦合被推到基类后，派生类与外部世界分离得更干净。理想情况下，多数行为都在那些子类里，于是代码库很大一部分被隔离、更易维护。

不过，若发现这模式把基类变成一大碗代码炖菜，考虑把一些提供的操作抽到单独类，由基类分派职责。[Component](https://gameprogrammingpatterns.com/component.html) 模式在此能帮忙。

### 示例代码

模式如此简单，示例代码不多。这不说明它没用——模式关&#x4E4E;_&#x610F;图_，而非实现复杂度。

从 `Superpower` 基类开始：

```
// Some code
class Superpower
{
public:
  virtual ~Superpower() {}

protected:
  virtual void activate() = 0;

  void move(double x, double y, double z)
  {
    // 代码……
  }

  void playSound(SoundId sound, double volume)
  {
    // 代码……
  }

  void spawnParticles(ParticleType type, int count)
  {
    // 代码……
  }
};
```

`activate()` 是沙箱方法。它是虚且抽象的，子&#x7C7B;_&#x5FC5;&#x987B;_&#x91CD;写。这让创建能力子类的人清楚工作该放哪。

其余 protected 方法 `move()`、`playSound()`、`spawnParticles()` 是提供的操作——子类在实现 `activate()` 时调用它们。

本例没实现这些操作，真实游戏里会有真正代码。正是在这些方法里，`Superpower` 与其他游戏系统耦合——`move()` 可能调物理，`playSound()` 会跟音频引擎说话，等等。这一切都在基&#x7C7B;_&#x5B9E;&#x73B0;_&#x91CC;，耦合被封装在 `Superpower` 自身内。

好，放出放射性蜘蛛，造一种能力：

```
// Some code
class SkyLaunch : public Superpower
{
protected:
  virtual void activate()
  {
    // 弹跳上天
    playSound(SOUND_SPROING, 1.0f);
    spawnParticles(PARTICLE_DUST, 10);
    move(0, 0, 20);
  }
};
```

> 好吧，&#x80FD;_&#x8DF3;_&#x6216;许算不上&#x591A;_&#x8D85;_，我只是想保持简单。

这能力把英雄弹到空中，播合适声音，扬起一小团尘土。若所有超能力都这么简单——声音、粒子、运动的组合——我们根本不需要这模式。只需在 `Superpower` 里内置 `activate()`，访问声音 ID、粒子类型、运动等字段即可。但这只在每种能力本质相同、只差数据时才行。我们再展开一点：

```
// Some code
class Superpower
{
protected:
  double getHeroX()
  {
    // 代码……
  }

  double getHeroY()
  {
    // 代码……
  }

  double getHeroZ()
  {
    // 代码……
  }

  // 已有内容……
};
```

加了几个获取英雄位置的方法。`SkyLaunch` 子类现在能用：

```
// Some code
class SkyLaunch : public Superpower
{
protected:
  virtual void activate()
  {
    if (getHeroZ() == 0)
    {
      // 在地面，弹跳上天
      playSound(SOUND_SPROING, 1.0f);
      spawnParticles(PARTICLE_DUST, 10);
      move(0, 0, 20);
    }
    else if (getHeroZ() < 10.0f)
    {
      // 离地不远，二段跳
      playSound(SOUND_SWOOP, 1.0f);
      move(0, 0, getHeroZ() + 20);
    }
    else
    {
      // 高高在天，俯冲攻击
      playSound(SOUND_DIVE, 0.7f);
      spawnParticles(PARTICLE_SPARKLES, 1);
      move(0, 0, -getHeroZ());
    }
  }
};
```

有了状态访问，沙箱方法就能做真正、有趣的控制流。这里仍是几个简单 `if`，但你可以做任何事。沙箱方法本身是可含任意代码的完整方法，天空才是极限。

> 前文我建议对能力用数据驱动。这就是你可&#x80FD;_&#x4E0D;_&#x90A3;么做的一个原因：行为复杂且命令式时，用数据定义更难。

### 设计决策

可见 Subclass Sandbox 是相当“软”的模式：描述一个基本想法，却没有很多细密机制。因此每次应用都会做一些有趣选择。可考虑这些问题：

#### 应提供哪些操作？

这是最大的问题，深刻影响模式的感觉和效果。光谱最低端：基类不提&#x4F9B;_&#x4EFB;&#x4F55;_&#x64CD;作，只有沙箱方法。要实现它，你得调用基类之外的系统。若走那条路，说你在用这模式大概都不公平。

光谱另一端：基类提供子类可能需要&#x7684;_&#x6BCF;&#x4E00;_&#x4E2A;操作。子&#x7C7B;_&#x53EA;_&#x4E0E;基类耦合，完全不调用任何外部系统。

具体来说：每个子类源文件只需一个 `#include`——即其基类的那个。

两点之间是广阔中间地带：部分操作由基类提供，其余直接访问定义它的外部系统。提供越多操作，子类与外部系统耦合越少，但基类耦&#x5408;_&#x8D8A;多_。它从派生类卸下耦合，却把耦合推到基类自己身上。

若有一堆派生类都耦合到某个外部系统，这是赢：把耦合收进一个提供的操作，就集中到一处——基类。但做得越多，那个类越大、越难维护。

线画在哪？几条经验法则：

* 若某提供的操作只有一两个子类用，性价比不高。你给基类加复杂度，影响所有人，却只有少数类受益。\
  为了与其他提供的操作保持一致，或许值得；或许让这些特例子类直接调外部系统更简单干净。
*   调用游戏其他角落的方法时，若该方法不修改状态，侵入性较小。仍会有耦合，但是“安全”的耦合，因为它破坏不了游戏。

    > “安全”加引号是因为：即便只访问数据也可能出问题。多线程游戏可能在读取同时被修改，不小心就会得到假数据。\
    > 另一糟糕情况是：若游戏状态严格确定性（许多联机游戏为保持玩家同步都是），访问同步状态集合之外的东西会造成极难查的非确定 bug。\
    > 另一方面，会修改状态的调用更深层地把你绑在那些代码上，必须更警醒。因此它们是收进更可见的基类、做成提供的操作的好候选。
* 若提供的操作实现只是转发到某外部系统，增值不多。那时直接调外部方法可能更简单。\
  不过即便简单转发仍可能有用——那些方法常访问基类不想直接暴露给子类的状态。例如 `Superpower` 提供：

```
// Some code
void playSound(SoundId sound, double volume)
{
  soundEngine_.play(sound, volume);
}
```

只是转发到 `Superpower` 的 `soundEngine_` 字段。优势是：字段仍封装在 `Superpower` 里，子类戳不到。

#### 方法应直接提供，还是通过包含它们的对象？

本模式的挑战是：基类可能塞进多到痛苦的方法。可把某些方法挪到其他类，基类提供的操作只返回那些对象之一，以缓解。

例如，为让能力播声音，可直接加到 `Superpower`：

```
// Some code
class Superpower
{
protected:
  void playSound(SoundId sound, double volume)
  {
    // 代码……
  }

  void stopSound(SoundId sound)
  {
    // 代码……
  }

  void setVolume(SoundId sound)
  {
    // 代码……
  }

  // 沙箱方法与其他操作……
};
```

但若 `Superpower` 已过大难用，或许想避免。改为创建暴露该功能的 `SoundPlayer`：

```
// Some code
class SoundPlayer
{
  void playSound(SoundId sound, double volume)
  {
    // 代码……
  }

  void stopSound(SoundId sound)
  {
    // 代码……
  }

  void setVolume(SoundId sound)
  {
    // 代码……
  }
};
```

然后 `Superpower` 提供对它的访问：

```
// Some code
class Superpower
{
protected:
  SoundPlayer& getSoundPlayer()
  {
    return soundPlayer_;
  }

  // 沙箱方法与其他操作……

private:
  SoundPlayer soundPlayer_;
};
```

把提供的操作分流到辅助类，可为你做几件事：

* _减少基类方法数量。_ 本例从三个方法变成一个 getter。
* _辅助类里的代码通常更易维护。_ 像 `Superpower` 这样的核心基类，尽管我们好意，往往很难改，因为太多东西依赖它。功能挪到耦合更低的次级类，就更容易拨弄而不弄坏东西。
* _降低基类与其他系统的耦合。_ `playSound()` 直接挂在 `Superpower` 上时，基类直接绑死 `SoundId` 及实现所调用的音频代码。挪到 `SoundPlayer` 后，`Superpower` 只耦合到单一 `SoundPlayer` 类，再由它封装其余依赖。

#### 基类如何获得所需状态？

基类常需要一些想封装、不想让子类看见的数据。前例中 `Superpower` 提供 `spawnParticles()`。若实现需要某个粒子系统对象，如何拿到？

* 传给基类构造函数：\
  最简单做法是基类用构造参数接收：

```
// Some code
class Superpower
{
public:
  Superpower(ParticleSystem* particles)
  : particles_(particles)
  {}

  // 沙箱方法与其他操作……

private:
  ParticleSystem* particles_;
};
```

这能安全确保每个超能力在构造完成时都有粒子系统。但看派生类：

```
// Some code
class SkyLaunch : public Superpower
{
public:
  SkyLaunch(ParticleSystem* particles)
  : Superpower(particles)
  {}
};
```

问题来了：每个派生类都要有调用基类并把参数传下去的构造函数。这样就把我们不想让它们知道的状态暴露给每个派生类。\
这也是维护噩梦：以后再给基类加一块状态，每个派生类的构造函数都得改来转发。

* 两阶段初始化：\
  为避免一切经构造函数传递，可把初始化拆成两步。构造函数无参，只创建对象；再调基类上的单独方法传入其余所需数据：

```
// Some code
Superpower* power = new SkyLaunch();
power->init(particles);
```

注意：既然没向 `SkyLaunch` 构造函数传任何东西，它就不会耦合到我们想在 `Superpower` 里保持私有的东西。麻烦在于：必须总记得调 `init()`。一忘，就会得到半生不熟、不能工作的能力。\
可用一个函数封装整个过程来修：

```
// Some code
Superpower* createSkyLaunch(ParticleSystem* particles)
{
  Superpower* power = new SkyLaunch();
  power->init(particles);
  return power;
}
```

再靠私有构造函数、友元类等小技巧，可确保 `createSkyLaunch()` &#x662F;_&#x552F;_&#x4E00;能真正创建能力的函数。这样就不会漏掉任何初始化阶段。

* 让状态成为静态的：\
  前例中我们为每个 `Superpower` _实&#x4F8B;_&#x521D;始化粒子系统。每个能力需要自己独有状态时说得通。但若粒子系统是[单例](https://gameprogrammingpatterns.com/singleton.html)，每个能力共享同一状态呢？\
  那时可让状态对基类私有&#x4E14;_&#x9759;态_。游戏仍要确保初始化状态，但只需为整个游戏初始化一次 `Superpower` _类_，而非每个实例。\
  记住这仍有许多单例问题：状态在大量对象（所有 `Superpower` 实例）间共享。粒子系统被封装，全&#x5C40;_&#x4E0D;可见_，这很好，但推理能力仍更难，因为它们都能戳同一对象。

```
// Some code
class Superpower
{
public:
  static void init(ParticleSystem* particles)
  {
    particles_ = particles;
  }

  // 沙箱方法与其他操作……

private:
  static ParticleSystem* particles_;
};
```

注意 `init()` 和 `particles_` 都是静态的。只要游戏早早调用一次 `Superpower::init()`，每个能力都能访问粒子系统。同时可通过调用正确派生类的构造函数自由创建 `Superpower` 实例。\
更好的是：`particles_` &#x662F;_&#x9759;&#x6001;_&#x53D8;量，不必为每个 `Superpower` 实例存储，类更省内存。

* 使用服务定位器：\
  前一选项要求外部代码记得在需要前把状态推给基类，初始化负担放在周围代码上。另一选项是让基类自己拉取所需状态。做法之一是用 [Service Locator](https://gameprogrammingpatterns.com/service-locator.html) 模式：

```
// Some code
class Superpower
{
protected:
  void spawnParticles(ParticleType type, int count)
  {
    ParticleSystem& particles = Locator::getParticles();
    particles.spawn(type, count);
  }

  // 沙箱方法与其他操作……
};
```

这里 `spawnParticles()` 需要粒子系统。不是由外部代&#x7801;_&#x7ED9;_&#x4E00;个，而是从服务定位器自己取。

### 另见

* 应用 [Update Method](https://gameprogrammingpatterns.com/update-method.html) 时，你的 update 方法往往也是沙箱方法。
* 本模式是 [Template Method](http://en.wikipedia.org/wiki/Template_method_pattern) 的角色对调。两种模式都用一组原语操作实现一个方法。Subclass Sandbox 中，方法在派生类，原语操作在基类；Template Method 中，方法&#x5728;_&#x57FA;_&#x7C7B;，原语操作&#x7531;_&#x6D3E;&#x751F;_&#x7C7B;实现。
* 也可视为 [Facade](http://en.wikipedia.org/wiki/Facade_Pattern) 的变体。门面模式把多个不同系统藏在单一简化 API 后。Subclass Sandbox 中，基类充当门面，对子类隐藏整个游戏引擎。

***

一句话概括：基类提供沙箱入口（如 `activate()`）和一组 protected 操作；子类只在沙箱里调这些操作，从而与引擎其余部分解耦。

需要的话可以继续下一章（本书 Behavioral Patterns 里再下一篇是 [Type Object](https://gameprogrammingpatterns.com/type-object.html)）。
