# State

坦白时间：这一章我有点用力过猛，塞进了太多内容。表面上，它讨论的是状态设计模式（State），但只要把状态模式与游戏放在一起讲，就绕不开更基础的有限状态机（Finite State Machine，简称 FSM）。既然都讲到这里了，我索性又加入了分层状态机和下推自动机。

需要介绍的内容很多，所以为了尽可能缩短篇幅，这里的代码示例省略了一些细节，需要你自行补充。我希望它们仍然足够清晰，能让你理解整体思路。

如果你从未听说过状态机，也不用难过。虽然 AI 和编译器领域的高手对它十分熟悉，但其他编程圈子对它了解得并不多。我认为状态机应该得到更广泛的认识，因此这里会把它用到一种不同的问题上。

这种组合呼应了人工智能发展的早期。在二十世纪五六十年代，AI 研究很大一部分都集中在语言处理上。如今编译器用来解析编程语言的许多技术，最初都是为了分析人类语言而发明的。

### 我们都经历过这种事

我们正在制作一款小型横版平台游戏。我们的任务是实现女主角，也就是玩家在游戏世界中的化身。这意味着要让她响应玩家输入。按下 B 键时，她应该跳起来。很简单：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    yVelocity_ = JUMP_VELOCITY;
    setGraphics(IMAGE_JUMP);
  }
}
```

发现缺陷了吗？

这段代码无法阻止“空中跳跃”。只要在她位于空中时不停按 B，她就能永远飘下去。

简单的修复方式是在 `Heroine` 中添加一个布尔字段 `isJumping_`，用于记录她是否正在跳跃：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    if (!isJumping_)
    {
      isJumping_ = true;
      // 跳跃……
    }
  }
}
```

当然，还应该在女主角接触地面时，将 `isJumping_` 重新设为 `false`。为了保持简洁，这里省略了相关代码。

接下来，我们希望女主角站在地面上时，如果玩家按下方向键下，她就蹲下；松开按键后，她再站起来：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    // 如果没有正在跳跃，则跳跃……
  }
  else if (input == PRESS_DOWN)
  {
    if (!isJumping_)
    {
      setGraphics(IMAGE_DUCK);
    }
  }
  else if (input == RELEASE_DOWN)
  {
    setGraphics(IMAGE_STAND);
  }
}
```

这次发现缺陷了吗？

使用这段代码，玩家可以：

1. 按下方向键下，让女主角蹲下。
2. 按下 B，让她从蹲伏姿势起跳。
3. 在她仍然位于空中时松开方向键下。

女主角会在跳跃途中切换成站立图像。是时候再添加一个标志了……

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    if (!isJumping_ && !isDucking_)
    {
      // 跳跃……
    }
  }
  else if (input == PRESS_DOWN)
  {
    if (!isJumping_)
    {
      isDucking_ = true;
      setGraphics(IMAGE_DUCK);
    }
  }
  else if (input == RELEASE_DOWN)
  {
    if (isDucking_)
    {
      isDucking_ = false;
      setGraphics(IMAGE_STAND);
    }
  }
}
```

接下来，如果玩家在跳跃途中按下方向键下，让女主角发动俯冲攻击，应该会很酷：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    if (!isJumping_ && !isDucking_)
    {
      // 跳跃……
    }
  }
  else if (input == PRESS_DOWN)
  {
    if (!isJumping_)
    {
      isDucking_ = true;
      setGraphics(IMAGE_DUCK);
    }
    else
    {
      isJumping_ = false;
      setGraphics(IMAGE_DIVE);
    }
  }
  else if (input == RELEASE_DOWN)
  {
    if (isDucking_)
    {
      // 站起来……
    }
  }
}
```

又到了寻找缺陷的时间。找到了吗？

我们检查了角色能否在跳跃时进行空中跳跃，却没有检查她是否正在俯冲。看来还得再添加一个字段……

显然，我们的方法出了问题。每次改动这一小段代码，都会破坏其他功能。接下来还需要添加更多动作——甚至连行走都还没有实现——但照这个速度，在完成之前，它就会彻底坍塌成一堆缺陷。

### 有限状态机前来救援

沮丧之下，你把桌上的东西一扫而空，只留下一支笔和一张纸，开始绘制流程图。

你为女主角可能处于的每种状态分别画一个方框：站立、跳跃、蹲伏和俯冲。当她能够在某种状态下响应按键时，就从对应方框画出一条箭头，标注触发转换的按键，再将它连接到她随后进入的状态。

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### 有限状态机的救援

恭喜，你刚刚创建了一台有限状态机。它源自计算机科学中称为自动机理论的一个分支。这个数据结构家族还包括著名的图灵机，而 FSM 是其中最简单的成员。

基本原理如下：

* 机器拥有一组固定的可选状态。在我们的例子中，就是站立、跳跃、蹲伏和俯冲。
* 机器同一时间只能处于一种状态。女主角不能同时处于跳跃和站立状态。事实上，防止这种情况正是我们使用 FSM 的原因之一。
* 一系列输入或事件会被发送给机器。在这个例子中，就是原始的按键按下和松开事件。
* 每个状态都有一组状态转换。每个转换都与某项输入关联，并指向另一个状态。当收到输入时，如果它与当前状态的某项转换匹配，机器就会切换到该转换所指向的状态。

例如，站立时按下方向键下会切换到蹲伏状态；跳跃时按下方向键下则会切换到俯冲状态。如果当前状态没有为某项输入定义转换，就忽略该输入。

在最纯粹的形式下，这就是全部内容：状态、输入和转换。你可以把它画成一张小型流程图。遗憾的是，编译器看不懂我们的涂鸦，那么应该如何实现它呢？

“四人帮”的状态模式是一种方法——稍后会讲到——不过我们先从更简单的方式开始。

我最喜欢用《Zork》这类老式文字冒险游戏来类比 FSM。游戏世界由许多房间组成，房间之间通过出口连接。你输入“向北走”这样的命令来探索这些房间。

它可以直接映射到状态机：

* 每个房间都是一种状态。
* 当前所在的房间就是当前状态。
* 每个房间的出口都是状态转换。
* 导航命令就是输入。

### 枚举与 `switch`

`Heroine` 类的问题之一，是布尔字段的某些组合并不合法。例如，`isJumping_` 和 `isDucking_` 绝不应该同时为 `true`。

如果你有一组标志，而其中同一时间只能有一个为 `true`，这通常说明真正需要的是一个枚举。

在这里，这个枚举正是 FSM 的状态集合：

```cpp
// Some code
enum State
{
  STATE_STANDING,
  STATE_JUMPING,
  STATE_DUCKING,
  STATE_DIVING
};
```

`Heroine` 不再保存一堆标志，而只保存一个 `state_` 字段。

我们还要颠倒条件分支的顺序。在此前的代码中，我们先根据输入分支，然后根据状态分支。这样可以将处理同一个按键的代码集中在一起，却会让处理同一种状态的代码散落在各处。

我们希望把同一种状态的代码放在一起，因此首先根据状态进行分支：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  switch (state_)
  {
    case STATE_STANDING:
      if (input == PRESS_B)
      {
        state_ = STATE_JUMPING;
        yVelocity_ = JUMP_VELOCITY;
        setGraphics(IMAGE_JUMP);
      }
      else if (input == PRESS_DOWN)
      {
        state_ = STATE_DUCKING;
        setGraphics(IMAGE_DUCK);
      }
      break;

    case STATE_JUMPING:
      if (input == PRESS_DOWN)
      {
        state_ = STATE_DIVING;
        setGraphics(IMAGE_DIVE);
      }
      break;

    case STATE_DUCKING:
      if (input == RELEASE_DOWN)
      {
        state_ = STATE_STANDING;
        setGraphics(IMAGE_STAND);
      }
      break;
  }
}
```

这看起来微不足道，但相比此前的代码确实有了明显改善。我们仍然使用条件分支，但可变状态已经被简化成单个字段。处理每种状态的代码也整齐地集中在了一起。

这是实现状态机最简单的方式，对于某些用途已经足够。

尤其重要的是，女主角现在不可能处于无效状态。使用布尔标志时，有些字段组合虽然可能出现，却没有实际意义。使用枚举后，每个取值都是有效状态。

不过，你的问题可能最终会超出这种方案的能力范围。

假设我们想添加一种新动作：女主角蹲伏一段时间进行蓄力，然后释放特殊攻击。在她蹲伏时，需要记录蓄力时间。

我们在 `Heroine` 中添加 `chargeTime_` 字段，保存攻击已经蓄力了多长时间。假设已经有一个每帧调用的 `update()` 方法，可以在其中添加：

```cpp
// Some code
void Heroine::update()
{
  if (state_ == STATE_DUCKING)
  {
    chargeTime_++;
    if (chargeTime_ > MAX_CHARGE)
    {
      superBomb();
    }
  }
}
```

如果你猜出这是更新方法模式（Update Method），就可以领一份奖品！

女主角开始蹲伏时，需要重置计时器，因此还要修改 `handleInput()`：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  switch (state_)
  {
    case STATE_STANDING:
      if (input == PRESS_DOWN)
      {
        state_ = STATE_DUCKING;
        chargeTime_ = 0;
        setGraphics(IMAGE_DUCK);
      }
      // 处理其他输入……
      break;

      // 其他状态……
  }
}
```

总而言之，为了添加这项蓄力攻击，我们不得不修改两个方法，并在 `Heroine` 中添加 `chargeTime_` 字段，尽管这个字段只有在蹲伏状态下才有意义。

我们真正希望的是，将所有相关代码和数据整齐地封装在同一个地方。“四人帮”已经为我们准备好了解决方案。

### 状态模式

对于深受面向对象思维影响的人来说，每个条件分支似乎都是使用动态分派的机会——在 C++ 中，也就是一次虚方法调用。

我认为沿着这条路钻得太深也会走火入魔。有时，一个 `if` 就足够了。

这种倾向有其历史背景。许多早期的面向对象传道者，例如《设计模式》的“四人帮”和《重构》的 Martin Fowler，都来自 Smalltalk 世界。在 Smalltalk 中，`ifThen:` 只是一个在条件对象上调用的方法，而 `true` 和 `false` 对象分别提供不同的实现。

不过，在我们的例子中，情况已经越过临界点，面向对象的解决方案确实更加合适。这就引出了状态模式。用“四人帮”的话来说：

> 允许一个对象在其内部状态发生变化时改变自身行为。这个对象看起来就像改变了自己的类。

这句话没有告诉我们多少东西。见鬼，我们的 `switch` 同样做到了这一点。将他们描述的具体模式应用到女主角身上，会是下面这样。

#### 状态接口

首先，为状态定义一个接口。所有依赖状态的行为——也就是此前所有出现 `switch` 的地方——都变成该接口中的虚方法。

在我们的例子中，就是 `handleInput()` 和 `update()`：

```cpp
// Some code
class HeroineState
{
public:
  virtual ~HeroineState() {}
  virtual void handleInput(Heroine& heroine, Input input) {}
  virtual void update(Heroine& heroine) {}
};
```

#### 为每种状态创建类

我们为每种状态定义一个实现该接口的类。类中的方法决定女主角处于该状态时的行为。

换句话说，把此前 `switch` 语句中的每个 `case` 取出来，移入对应的状态类。例如：

```cpp
// Some code
class DuckingState : public HeroineState
{
public:
  DuckingState()
  : chargeTime_(0)
  {}

  virtual void handleInput(Heroine& heroine, Input input)
  {
    if (input == RELEASE_DOWN)
    {
      // 切换到站立状态……
      heroine.setGraphics(IMAGE_STAND);
    }
  }

  virtual void update(Heroine& heroine)
  {
    chargeTime_++;
    if (chargeTime_ > MAX_CHARGE)
    {
      heroine.superBomb();
    }
  }

private:
  int chargeTime_;
};
```

注意，我们还将 `chargeTime_` 从 `Heroine` 移入了 `DuckingState` 类。这非常好——这项数据只有处于该状态时才有意义，现在对象模型明确体现了这一事实。

#### 委托给状态

接下来，让 `Heroine` 保存一个指向当前状态的指针，移除那些庞大的 `switch`，改为将行为委托给状态对象：

```cpp
// Some code
class Heroine
{
public:
  virtual void handleInput(Input input)
  {
    state_->handleInput(*this, input);
  }

  virtual void update()
  {
    state_->update(*this);
  }

  // 其他方法……
private:
  HeroineState* state_;
};
```

为了“改变状态”，只需让 `state_` 指向另一个 `HeroineState` 对象。

这就是状态模式的全部内容。

它看起来很像策略模式和类型对象模式。这三个模式都包含一个主对象，由它把工作委托给另一个从属对象。区别在于设计意图：

* 策略模式： 将主类与其部分行为解耦。
* 类型对象模式： 让多个对象通过共享同一个类型对象的引用，表现出相似的行为。
* 状态模式： 让主对象通过更换接受委托的对象来改变自身行为。

### 状态对象从哪里来？

这里有个细节被我轻描淡写地略过了。

改变状态时，需要让 `state_` 指向新状态，但新状态对象从哪里来？使用枚举实现时，这根本不是问题，因为枚举值和数字一样属于基础值。但现在状态是类，因此必须获得一个真实实例供指针指向。

通常有两种解决方案。

#### 静态状态

如果状态对象没有任何字段，那么它保存的唯一数据就是一个指向内部虚方法表的指针，以便调用它的方法。

这种情况下，完全没有必要创建多个实例，因为每个实例都一模一样。

如果状态没有字段，而且只包含一个虚方法，还可以进一步简化该模式。用一个普通的顶层状态函数取代每个状态类，主类中的 `state_` 字段也就变成了简单的函数指针。

状态类没有字段时，可以为它创建一个静态实例。即使同时有很多 FSM 处于同一种状态，它们也可以全部指向同一个实例，因为其中不包含任何与特定状态机有关的数据。

这就是享元模式（Flyweight）。

静态实例放在哪里由你决定，只需找个合理的位置即可。没有什么特别的理由，我们就把它们放在状态基类中：

```cpp
// Some code
class HeroineState
{
public:
  static StandingState standing;
  static DuckingState ducking;
  static JumpingState jumping;
  static DivingState diving;

  // 其他代码……
};
```

这些静态字段分别是游戏中每种状态的唯一实例。为了让女主角跳跃，站立状态可以这样做：

```cpp
// Some code
if (input == PRESS_B)
{
  heroine.state_ = &HeroineState::jumping;
  heroine.setGraphics(IMAGE_JUMP);
}
```

#### 实例化状态

不过，有时静态状态行不通。蹲伏状态就不能使用静态实例，因为它包含 `chargeTime_` 字段，而该字段属于当前恰好正在蹲伏的女主角。

如果游戏中只有一位女主角，这种做法碰巧可能正常工作。但如果添加双人合作模式，让两位女主角同时出现在屏幕上，就会出现问题。

这种情况下，必须在转换到某个状态时创建相应的状态对象。这样，每台 FSM 都能拥有自己的状态实例。

当然，如果分配了新状态，就需要释放当前状态。这里必须谨慎，因为触发状态转换的代码正位于当前状态的方法中。我们不希望在方法仍在运行时把 `this` 从脚下删除。

因此，我们允许 `HeroineState` 中的 `handleInput()` 选择性地返回一个新状态。返回新状态时，`Heroine` 再删除旧状态并换上新状态：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  HeroineState* state = state_->handleInput(*this, input);
  if (state != NULL)
  {
    delete state_;
    state_ = state;
  }
}
```

这样，直到旧状态的方法返回之后，我们才会删除它。

现在，站立状态可以通过创建新实例来转换到蹲伏状态：

```cpp
// Some code
HeroineState* StandingState::handleInput(Heroine& heroine,
                                         Input input)
{
  if (input == PRESS_DOWN)
  {
    // 其他代码……
    return new DuckingState();
  }

  // 保持当前状态。
  return NULL;
}
```

只要可以，我更喜欢静态状态，因为每次状态变化时，它都不会浪费内存和 CPU 周期来分配对象。不过，对于那些更加“有状态”的状态，就应该使用实例化方式。

动态分配状态时，可能需要担心内存碎片。“对象池”模式可以帮助解决这个问题。

### 进入和退出动作

状态模式的目标，是把一种状态的所有行为和数据都封装到同一个类中。我们已经完成了一部分，但仍有一些零散内容没有处理。

女主角改变状态时，还会切换她的精灵图。目前，这段代码由她正在离开的状态负责。例如，从蹲伏切换到站立时，由蹲伏状态设置她的图像：

```cpp
// Some code
HeroineState* DuckingState::handleInput(Heroine& heroine,
                                        Input input)
{
  if (input == RELEASE_DOWN)
  {
    heroine.setGraphics(IMAGE_STAND);
    return new StandingState();
  }

  // 其他代码……
}
```

我们真正希望的是，让每种状态控制自己的图像。可以通过为状态添加进入动作来实现：

```cpp
// Some code
class StandingState : public HeroineState
{
public:
  virtual void enter(Heroine& heroine)
  {
    heroine.setGraphics(IMAGE_STAND);
  }

  // 其他代码……
};
```

回到 `Heroine` 中，修改状态转换代码，让它调用新状态的进入动作：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  HeroineState* state = state_->handleInput(*this, input);
  if (state != NULL)
  {
    delete state_;
    state_ = state;

    // 调用新状态的进入动作。
    state_->enter(*this);
  }
}
```

这样就能简化蹲伏状态的代码：

```cpp
// Some code
HeroineState* DuckingState::handleInput(Heroine& heroine,
                                        Input input)
{
  if (input == RELEASE_DOWN)
  {
    return new StandingState();
  }

  // 其他代码……
}
```

进入动作有一个特别好的地方：无论从哪个状态转换而来，只要进入该状态，它都会执行。

现实中的状态图通常会有多条转换指向同一个状态。例如，女主角在跳跃或俯冲落地之后也会进入站立状态。这意味着，如果没有进入动作，我们就必须在每个转换位置重复相同代码。进入动作为集中这些代码提供了合适的位置。

当然，我们也可以扩展这套机制来支持退出动作。它只是在切换到新状态之前，对即将离开的状态调用的一个方法。

### 代价是什么？

我花了这么多时间向你推销 FSM，现在却要突然抽走你脚下的地毯。

到目前为止，我所说的都是真的，FSM 确实非常适合某些问题。但它最大的优点，同时也是它最大的缺点。

状态机通过强制代码遵循一种高度受限的结构，帮助你理清纠缠不清的逻辑。你拥有的只有：

* 一组固定的状态；
* 一个当前状态；
* 一些硬编码的状态转换。

有限状态机甚至都不是图灵完备的。自动机理论使用一系列抽象模型描述计算，每种模型都比前一种更加复杂。图灵机是其中表达能力最强的模型之一。

“图灵完备”意味着一个系统——通常是一门编程语言——拥有足够的能力，可以在其中实现一台图灵机。从某种意义上说，所有图灵完备语言都具有相同的表达能力。FSM 的灵活性还不足以加入这个俱乐部。

如果尝试用状态机处理游戏 AI 这类更加复杂的问题，就会一头撞上这个模型的种种限制。

幸运的是，我们的前辈已经找到了绕开其中一些障碍的方法。本章最后会介绍其中几种。

### 并发状态机

我们决定让女主角能够携带枪支。持枪时，她仍然可以做此前的一切：奔跑、跳跃、蹲伏等等。但她还需要能够在执行这些动作的同时开火。

如果坚持把所有内容都塞进一台 FSM，就必须把状态数量翻倍。对于现有的每种状态，都需要再创建一个持枪时执行相同动作的状态：

* 站立；
* 持枪站立；
* 跳跃；
* 持枪跳跃；
* 诸如此类。

再添加几种武器，状态数量就会组合式爆炸。不仅状态数量庞大，还会产生大量重复：持枪和未持枪的状态几乎完全相同，区别仅仅是那一点处理开火的代码。

问题在于，我们将两类状态——她正在做什么，以及她携带着什么——硬塞进了同一台状态机。为了表示所有可能的组合，每一对状态都需要一个独立状态。

解决方案显而易见：使用两台独立的状态机。

如果把表示行为的 `n` 种状态和表示装备的 `m` 种状态塞进一台机器，就需要 `n × m` 种状态。使用两台机器，则只需要 `n + m` 种。

我们保留原来的行为状态机，不对其进行改动。然后，再为女主角携带的装备定义一台独立状态机。`Heroine` 将拥有两个“状态”引用：

```cpp
// Some code
class Heroine
{
  // 其他代码……

private:
  HeroineState* state_;
  HeroineState* equipment_;
};
```

为了便于说明，我们对她的装备也使用完整的状态模式。在实际项目中，因为装备只有两种状态，使用一个布尔标志也可以。

女主角将输入委托给状态时，会把它同时交给两台状态机：

```cpp
// Some code
void Heroine::handleInput(Input input)
{
  state_->handleInput(*this, input);
  equipment_->handleInput(*this, input);
}
```

更加完善的系统可能会允许一台状态机“消费”某项输入，使另一台状态机不再收到它。这样可以避免两台机器错误地同时响应同一项输入。

每台状态机都能独立响应输入、执行行为并改变自身状态。当两组状态大体互不相关时，这种方式非常有效。

实际使用中，仍会遇到少数两组状态相互影响的情况。例如，她也许不能在跳跃时开火，或者持枪时不能进行俯冲攻击。

为了处理这些情况，你可能只能在一台状态机的状态代码中使用几个简单粗暴的 `if`，检查另一台状态机的状态并协调两者。这不是最优雅的方案，但能完成工作。

### 分层状态机

进一步完善女主角的行为之后，她很可能会拥有许多相似的状态。例如：

* 站立；
* 行走；
* 奔跑；
* 滑行。

在这些状态中的任何一个状态下，按 B 都会跳跃，按方向键下都会蹲伏。

使用简单状态机时，必须在每种状态中复制这段代码。如果能只实现一次，然后在所有这些状态间复用，就更好了。

如果这是普通的面向对象代码，而不是状态机，那么在这些状态间共享代码的一种方式就是继承。可以定义一个“位于地面上”的状态类，由它处理跳跃和蹲伏。随后，让站立、行走、奔跑和滑行状态继承该类，并添加各自特有的行为。

这既有优点，也有缺点。继承是一种强大的代码复用方式，但也会在两段代码之间建立非常牢固的耦合。它是一把大锤，挥动时务必谨慎。

这种常见结构称为分层状态机（Hierarchical State Machine）。

一个状态可以拥有一个父状态，而自身则成为子状态。收到事件时，如果子状态没有处理它，事件就会沿着父状态链向上传递。换句话说，它的工作方式与重写继承方法很相似。

如果使用状态模式实现 FSM，就可以借助类继承实现这种层次结构。首先为父状态定义一个基类：

```cpp
// Some code
class OnGroundState : public HeroineState
{
public:
  virtual void handleInput(Heroine& heroine, Input input)
  {
    if (input == PRESS_B)
    {
      // 跳跃……
    }
    else if (input == PRESS_DOWN)
    {
      // 蹲伏……
    }
  }
};
```

然后让每种子状态继承它：

```cpp
// Some code
class DuckingState : public OnGroundState
{
public:
  virtual void handleInput(Heroine& heroine, Input input)
  {
    if (input == RELEASE_DOWN)
    {
      // 站起来……
    }
    else
    {
      // 没有处理该输入，因此沿层次结构向上查找。
      OnGroundState::handleInput(heroine, input);
    }
  }
};
```

当然，这并不是实现层次结构的唯一方法。如果没有使用“四人帮”的状态模式，这种方案就无法工作。

作为替代，可以在主类中使用一个状态栈，而不是单个状态，从而显式表示当前状态的父状态链。

当前状态位于栈顶，其下方是它的直接父状态，再往下是父状态的父状态，以此类推。需要执行某项与状态相关的行为时，从栈顶开始向下遍历，直到某个状态处理它。如果没有任何状态进行处理，就忽略它。

### 下推自动机

有限状态机还有另一种常见扩展，同样使用状态栈。令人困惑的是，这个栈代表的东西完全不同，解决的也是另一个问题。

问题在于，有限状态机没有“历史”这个概念。你知道自己当前处于什么状态，却不记得此前处于什么状态，因此没有简单的方法返回上一个状态。

来看一个例子。前面，我们让无所畏惧的女主角武装到了牙齿。她开枪时，需要一个新状态来播放射击动画、生成子弹和各种视觉效果。

因此，我们快速拼出一个 `FiringState`，并让所有能够开火的状态在玩家按下开火键时转换到这个状态。

由于这项行为会在多个状态中重复，它可能也适合使用分层状态机来复用代码。

棘手之处在于：射击结束后，她应该转换到什么状态？

她可以在站立、奔跑、跳跃和蹲伏时开火。射击序列结束后，她应该回到开火前正在做的事情。

如果坚持使用普通 FSM，那么我们早已忘记她此前处于什么状态。为了追踪它，只能定义一大堆几乎完全相同的状态：

* 站立时开火；
* 奔跑时开火；
* 跳跃时开火；
* 蹲伏时开火；
* 诸如此类。

这样做仅仅是为了让每种状态都能硬编码一条转换，在结束时回到正确的状态。

我们真正需要的是一种能够保存开火前状态，并在稍后恢复它的方法。自动机理论再次前来帮忙。相应的数据结构称为下推自动机（Pushdown Automaton）。

有限状态机只有一个指向状态的指针，而下推自动机拥有一个状态栈。在 FSM 中，转换到新状态会替换此前的状态。下推自动机也允许这样做，但还提供了两项额外操作：

* 压入状态： 可以将新状态压入栈中。栈顶状态始终是“当前”状态，因此这会转换到新状态。但原来的状态不会被丢弃，而是保留在新状态正下方。
* 弹出状态： 可以弹出并丢弃栈顶状态。它下方的状态随即成为新的当前状态。

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

这正是开火行为所需要的机制。我们只需创建一个射击状态。当角色处于其他任意状态并按下开火键时，就将射击状态压入栈中。射击动画播放完毕后，再将其弹出。下推自动机会自动让角色回到开火前所处的状态。

### 那么，它们究竟有多大用处？

即使加入这些常见扩展，状态机仍然存在很大的局限性。如今游戏 AI 的发展趋势更倾向于行为树和规划系统等令人兴奋的技术。如果你感兴趣的是复杂 AI，那么本章所做的仅仅是勾起你的兴趣。要真正满足求知欲，还需要阅读其他书籍。

但这并不意味着有限状态机、下推自动机以及其他简单系统没有用处。对于某些类型的问题，它们是很好的建模工具。有限状态机适用于以下情况：

* 某个实体的行为会根据其内部状态发生变化。
* 该状态可以严格划分为数量相对较少的几种明确选项之一。
* 该实体会随时间推移，对一系列输入或事件作出响应。

在游戏领域，状态机最广为人知的用途是实现 AI。不过，它也经常用于处理用户输入、菜单界面导航、文本解析、网络协议以及其他异步行为。

