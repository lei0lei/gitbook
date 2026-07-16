# 命令模式

《设计模式：可复用面向对象软件的基础》按我的计时，已经快二十岁了。除非你现在正从我肩后看屏幕，否则等你读到这段时，它大概已经“成年”到能合法喝酒了。对软件这样一个飞速变化的行业来说，这简直古老得不可思议。这本书经久不衰的受欢迎程度，也说明了一点：与许多框架和方法论相比，设计本身是多么经得起时间考验。

虽然我认为《设计模式》依然有价值，但过去几十年我们也学到了很多。在本节中，我们会逐一走过“四人帮”记录的一些原始模式。对每个模式，我都希望能说点有用或有趣的东西。

我觉得有些模式被过度使用了（比如 Singleton，单例），而另一些则没有得到足够重视（比如 Command，命令）。还有几个放在这里，是因为我想专门探讨它们与游戏的关联（Flyweight，享元；Observer，观察者）。最后，有时我只是觉得，看看这些模式如何嵌入更广阔的编程领域，本身就很有意思（Prototype，原型；State，状态）。

Command（命令）是我最喜欢的模式之一。我写的大多数大型程序——游戏也好，其他也罢——最终都会在某个地方用到它。当我把它用在合适的位置时，它总能干净利落地解开一些真正棘手的代码。对这样一个出色的模式来说，“四人帮”的描述却一如既往地抽象晦涩：

> 将一个请求封装为对象，从而允许你以不同请求参数化客户，并对请求排队或记录日志，以及支持可撤销操作。

我想我们都同意，这句子写得很糟。首先，它把想建立的隐喻搞得一团糟。在软件这个词语可以指一切的怪诞世界里之外，“client（客户）”指的是人——和你做生意的人。据我所知，人类是没法被“参数化”的。

然后，句子后半段只是一串“你也许可能”会拿这个模式来干的事。除非你的用例刚好落在那份清单里，否则这几乎没什么启发。我对 Command 模式的简短概括是：

命令，就是被具象化的方法调用。

“Reify（具象化）”来自拉丁语 _res_，意为“事物”，加上英语后缀 “-fy”。所以它基本上就是 “thingify（事物化）”——说实话，用这个词会更有趣。

当然，“pithy（简洁）”往往也意味着“密不透风的简短”，所以这可能也算不上多大改进。让我展开一点。“Reify”如果你没听过，意思就是“变成真实存在的东西”。具象化的另一个说法，是把某样东西做成“一等公民（first-class）”。

某些语言里的反射系统，让你能在运行时以命令式方式操作程序中的类型。你可以拿到一个对象，它代表另一个对象的类，然后摆弄它，看看这个类型能做什么。换句话说，反射就是一个被具象化的类型系统。

这两个术语的意思都是：把一个概念变成一块数据——一个对象——你可以把它放进变量、传给函数，等等。所以我说 Command 模式是“被具象化的方法调用”，意思就是：把一个方法调用包进对象里。

这听起来很像“回调（callback）”“一等函数（first-class function）”“函数指针（function pointer）”“闭包（closure）”或“偏函数应用（partially applied function）”——取决于你来自哪种语言——确实，它们都在同一类东西里。“四人帮”后来也写道：

> 命令，是面向对象世界里对回调的替代。

比起他们选的那句，这更适合当这个模式的标语。

但这些都还抽象、模糊。我喜欢每章开头先给点具体的东西，而这一点我搞砸了。为了弥补，从这里开始，我会用一系列例子说明：命令模式在哪些地方特别合适。

### 配置输入（Configuring Input）

每个游戏里总有一块代码，负责读取原始用户输入——按键、键盘事件、鼠标点击，诸如此类。它接收每一次输入，并把它翻译成游戏里的一个有意义动作：

<br>

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

一个极其简单的实现如下：

```cpp
// Some code
void InputHandler::handleInput()
{
  if (isPressed(BUTTON_X)) jump();
  else if (isPressed(BUTTON_Y)) fireGun();
  else if (isPressed(BUTTON_A)) swapWeapon();
  else if (isPressed(BUTTON_B)) lurchIneffectively();
}
```

小提示：别太常按 B 键。

这个函数通常由游戏循环每帧调用一次，我相信你能看懂它在做什么。如果我们愿意把用户输入硬编码到游戏动作上，这段代码完全可用；但许多游戏允许用户自行配置按键映射。

要支持这一点，我们得把直接调用的 `jump()` 和 `fireGun()` 变成可以替换的东西。“替换”听起来很像变量赋值，因此我们需要一个对象来表示游戏动作。命令模式登场。

我们定义一个基类，用于表示可以被触发的游戏命令：

```cpp
// Some code
class Command
{
public:
  virtual ~Command() {}
  virtual void execute() = 0;
};
```

当你看到一个只包含单一方法、且该方法没有返回值的接口时，它很可能就是命令模式。

接着，为每种不同的游戏动作创建子类：

```cpp
// Some code
class JumpCommand : public Command
{
public:
  virtual void execute() { jump(); }
};

class FireCommand : public Command
{
public:
  virtual void execute() { fireGun(); }
};

// 其余的你应该能猜到……
```

在输入处理器中，我们为每个按键保存一个指向命令的指针：

```cpp
// Some code
class InputHandler
{
public:
  void handleInput();

  // 用于绑定命令的方法……

private:
  Command* buttonX_;
  Command* buttonY_;
  Command* buttonA_;
  Command* buttonB_;
};
```

现在，输入处理只需将工作委托给这些命令：

```cpp
// Some code
void InputHandler::handleInput()
{
  if (isPressed(BUTTON_X)) buttonX_->execute();
  else if (isPressed(BUTTON_Y)) buttonY_->execute();
  else if (isPressed(BUTTON_A)) buttonA_->execute();
  else if (isPressed(BUTTON_B)) buttonB_->execute();
}
```

注意我们这里没有检查 `NULL`？这是因为它假设每个按键都会绑定某个命令。

如果想支持“什么也不做”的按键，又不必显式检查 `NULL`，可以定义一个命令类，让它的 `execute()` 方法什么也不做。这样，我们不再把按键处理器设为 `NULL`，而是让它指向这个对象。这种模式称为空对象模式（Null Object）。

原先每个输入都会直接调用一个函数；现在，中间多了一层间接性：

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

这就是命令模式的核心。如果你现在已经看出了它的价值，那么本章后面的内容就算额外赠送。

### 给角色下达指令

刚才定义的命令类可以满足前一个例子，但它们限制很多。问题在于，它们假设存在 `jump()`、`fireGun()` 等顶层函数；这些函数会隐式地知道如何找到玩家控制的角色，并让这个提线木偶般的角色做出动作。

这种默认的耦合限制了命令的用途。`JumpCommand` 唯一能让跳起来的对象就是玩家。让我们放宽这个限制：不再调用那些自行寻找被命令对象的函数，而是直接把我们想指挥的对象传进去：

```
// Some code
class Command
{
public:
  virtual ~Command() {}
  virtual void execute(GameActor& actor) = 0;
};
```

这里的 `GameActor` 是我们的“游戏对象”类，表示游戏世界中的一个角色。我们将它传给 `execute()`，这样派生命令就能在我们指定的角色上调用方法，例如：

```
// Some code
class JumpCommand : public Command
{
public:
  virtual void execute(GameActor& actor)
  {
    actor.jump();
  }
};
```

现在，同一个类就能让游戏中的任何角色跳来跳去。我们只缺少连接输入处理器和命令的一环：它接收命令，并在正确的对象上执行命令。首先，修改 `handleInput()`，让它返回命令：

```
// Some code
Command* InputHandler::handleInput()
{
  if (isPressed(BUTTON_X)) return buttonX_;
  if (isPressed(BUTTON_Y)) return buttonY_;
  if (isPressed(BUTTON_A)) return buttonA_;
  if (isPressed(BUTTON_B)) return buttonB_;

  // 没有按键被按下，因此什么也不做。
  return NULL;
}
```

它不能立刻执行命令，因为它不知道该传入哪个角色。这里正好利用命令是“被具象化的调用”这一点——我们可以推迟调用的执行时机。

然后，我们需要一些代码，接收这个命令并让它在代表玩家的角色上运行，例如：

```
// Some code
Command* command = inputHandler.handleInput();
if (command)
{
  command->execute(actor);
}
```

假设 `actor` 是对玩家角色的引用，这段代码就能根据用户输入正确操控他，因此我们回到了第一个例子里的同样行为。

但在命令与实际执行命令的角色之间加入一层间接性，给了我们一个很巧妙的能力：现在只要改变执行命令的 `actor`，我们就能让玩家控制游戏中的任意角色。

实践中，这并不是一个常见功能，但有一种相近的用例却经常出现。到目前为止，我们只考虑玩家操控的角色；那游戏世界里的其他角色呢？它们由游戏 AI 驱动。我们同样可以将这个命令模式用作 AI 引擎与角色之间的接口：AI 代码只需产出 `Command` 对象。

选择命令的 AI 与执行命令的角色代码之间的解耦，带来了很大的灵活性。我们可以让不同角色使用不同的 AI 模块；也可以针对不同类型的行为混搭 AI。想要一个更有进攻性的对手？只需接入一个更具进攻性的 AI，为它生成命令即可。实际上，我们甚至可以把 AI 接到玩家角色上，这对演示模式之类的功能很有用：游戏需要自动驾驶式地运行。

通过把控制角色的命令变成一等对象，我们消除了直接方法调用带来的紧耦合。你可以把它理解成一个命令队列或命令流：

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

一些代码（输入处理器或 AI）负责生成命令，并把它们放入这个流中；另一些代码（调度器或角色本身）负责消费命令并调用它们。通过把队列放在中间，我们让一端的生产者与另一端的消费者解耦了。

如果我们让这些命令可序列化，就可以把命令流通过网络发送出去。我们可以取得玩家输入，将其传到另一台机器，再在那里重放。这是实现联网多人游戏的一个重要组成部分。

### 撤销与重做

最后一个例子是这个模式最广为人知的用途。如果命令对象能执行操作，那么让它能够撤销操作也只是顺理成章的一步。撤销功能用于某些策略游戏，玩家可以回退自己不满意的操作。在供人们制作游戏的工具中，它更是必备功能。想让游戏设计师讨厌你，最可靠的办法就是给他们一个无法撤销误操作的关卡编辑器。

这里我或许是在现身说法。

不使用命令模式，实现撤销会出乎意料地困难；使用了它，则轻而易举。假设我们正在制作一款单人回合制游戏，并希望允许用户撤销行动，让他们能更专注于策略，而不是猜测。

我们前面已经用命令抽象了输入处理，所以玩家的每次行动本来就已封装在命令对象里。例如，移动一个单位可以这样写：

```
// Some code
class MoveUnitCommand : public Command
{
public:
  MoveUnitCommand(Unit* unit, int x, int y)
  : unit_(unit),
    x_(x),
    y_(y)
  {}

  virtual void execute()
  {
    unit_->moveTo(x_, y_);
  }

private:
  Unit* unit_;
  int x_, y_;
};
```

注意，这和我们之前的命令稍有不同。上一个例子中，我们想把命令与被修改的角色解耦；这里，我们特意要把命令绑定到被移动的单位上。这个命令的一个实例不是可在多种上下文中使用的通用“移动某个东西”操作，而是游戏回合序列里一次具体、确定的移动。

这凸显了命令模式实现方式上的一种变化。在某些情况下，例如前几个例子，命令是一个可复用对象，表示某件能做的事。之前的输入处理器保存一个命令对象，并在对应按键被按下时调用它的 `execute()` 方法。

这里的命令则更加具体。它们表示在某个特定时间点执行的一件事。这意味着每次玩家选择一次移动时，输入处理代码都要创建一个新的命令实例。例如：

```
// Some code
Command* handleInput()
{
  Unit* unit = getSelectedUnit();

  if (isPressed(BUTTON_UP)) {
    // 将单位向上移动一格。
    int destY = unit->y() - 1;
    return new MoveUnitCommand(unit, unit->x(), destY);
  }

  if (isPressed(BUTTON_DOWN)) {
    // 将单位向下移动一格。
    int destY = unit->y() + 1;
    return new MoveUnitCommand(unit, unit->x(), destY);
  }

  // 其他移动……

  return NULL;
}
```

当然，在 C++ 这类没有垃圾回收的语言中，这意味着执行命令的代码也要负责释放它们的内存。

命令是一次性对象这个事实，马上会成为我们的优势。为了让命令可撤销，我们为每个命令类定义另一个需要实现的操作：

```
// Some code
class Command
{
public:
  virtual ~Command() {}
  virtual void execute() = 0;
  virtual void undo() = 0;
};
```

`undo()` 方法会逆转对应 `execute()` 方法对游戏状态造成的改变。下面是加入撤销支持后的移动命令：

```
// Some code
class MoveUnitCommand : public Command
{
public:
  MoveUnitCommand(Unit* unit, int x, int y)
  : unit_(unit),
    xBefore_(0),
    yBefore_(0),
    x_(x),
    y_(y)
  {}

  virtual void execute()
  {
    // 记录移动前单位的位置，
    // 以便之后恢复。
    xBefore_ = unit_->x();
    yBefore_ = unit_->y();

    unit_->moveTo(x_, y_);
  }

  virtual void undo()
  {
    unit_->moveTo(xBefore_, yBefore_);
  }

private:
  Unit* unit_;
  int xBefore_, yBefore_;
  int x_, y_;
};
```

注意，我们在类中加入了一些额外状态。单位移动后，会忘记它原先的位置。如果想撤销移动，就必须自行记住单位的旧位置；`xBefore_` 与 `yBefore_` 正是为此而设。

这看似是备忘录模式（Memento）的用武之地，但我发现它并不太好用。命令往往只修改对象状态的一小部分，若把其余数据也做快照，会浪费内存。手动只保存被改变的部分更便宜。

持久化数据结构是另一种选择。使用它时，每次修改对象都会返回一个新对象，旧对象保持不变。通过巧妙的实现，新对象会与旧对象共享数据，因此成本远低于克隆整个对象。

使用持久化数据结构时，每个命令都会保存一份“命令执行前对象”的引用；撤销就只是切回旧对象。

为了让玩家撤销一次移动，我们保留他们上次执行的命令。当他们按下 `Ctrl+Z` 时，调用该命令的 `undo()` 方法即可。（如果他们已经撤销过，则这次变成“重做”，我们再次执行该命令。）

支持多级撤销也不会困难太多。我们不再只记住最后一条命令，而是保存一个命令列表，并维护一个指向“当前”命令的引用。玩家执行命令时，将它追加到列表中，并让“当前”指向它。

当玩家选择“撤销”时，我们撤销当前命令，并把当前指针向前移。当他们选择“重做”时，我们先推进指针，再执行该命令。如果他们在撤销一些命令之后又执行了新命令，那么列表中位于当前命令之后的所有内容都会被丢弃。

我第一次在关卡编辑器中实现这个功能时，感觉自己像个天才。我惊讶于它竟然这么直接，而且效果这么好。你需要保持纪律，确保每一次数据修改都经由命令完成；一旦做到这一点，剩下的就简单了。

重做在游戏中未必常见，但回放很常见。一种天真的实现方式是记录每一帧的完整游戏状态，以便重放；但那会占用太多内存。

相反，许多游戏会记录每个实体在每一帧执行的命令集合。要回放游戏时，引擎只需正常运行游戏模拟，并执行预先记录的命令。

### 有类但不太好用？

前面我说过，命令类似于一等函数或闭包，但这里展示的每个例子都使用了类定义。如果你熟悉函数式编程，可能会想：函数去哪了？

我以这种方式写示例，是因为 C++ 对一等函数的支持相当有限。函数指针没有状态；函数对象很奇怪，而且仍然需要定义一个类；C++11 的 lambda 又因为手动内存管理而不太好处理。

这并不意味着在其他语言中，你不该使用函数来实现命令模式。如果你有一门真正支持闭包的语言，尽管使用它们！从某种意义上说，命令模式就是在不支持闭包的语言中模拟闭包的方法。

我说“从某种意义上”，是因为即使在有闭包的语言中，为命令构建实际的类或结构体依然有用。如果命令包含多个操作（例如可撤销命令），将它映射为单个函数会显得别扭。

定义一个带字段的真实类，也能帮助读者轻松看出命令包含哪些数据。闭包是一种非常简洁的自动状态封装方式，但它也可能自动得过头，让人难以看清它实际捕获并保存了什么状态。

例如，如果我们用 JavaScript 制作游戏，可以像这样创建一个“移动单位”命令：

```
// Some code
function makeMoveUnitCommand(unit, x, y) {
  // 这里返回的函数就是命令对象：
  return function() {
    unit.moveTo(x, y);
  }
}
```

也可以用一对闭包加入撤销支持：

```
// Some code
function makeMoveUnitCommand(unit, x, y) {
  var xBefore, yBefore;
  return {
    execute: function() {
      xBefore = unit.x();
      yBefore = unit.y();
      unit.moveTo(x, y);
    },
    undo: function() {
      unit.moveTo(xBefore, yBefore);
    }
  };
}
```

如果你习惯函数式风格，这种做法很自然。如果还不习惯，希望这一章至少能帮助你逐步理解。对我而言，命令模式的实用性恰好体现了函数式范式对许多问题有多么有效。

### 另见（See Also）

你最终可能会有许多不同的命令类。为了更容易实现它们，通常可以定义一个具体基类，提供一组方便的高层方法；派生命令可以组合这些方法来定义自己的行为。这样，命令主要的 `execute()` 方法就成了子类沙盒模式（Subclass Sandbox）。

在示例中，我们显式选择了哪个角色处理命令。但在某些情况下，特别是对象模型具有层级结构时，事情未必这么明确。一个对象可能自己响应命令，也可能决定把它转交给某个下属对象。这样做时，你就得到了责任链模式（Chain of Responsibility）。

有些命令是无状态的纯行为片段，例如第一个例子里的 `JumpCommand`。在这种情况下，持有多个该类实例会浪费内存，因为所有实例都等价。\*\*享元模式（Flyweight）\*\*专门处理这个问题。

你当然也可以把它做成单例，但朋友不会让朋友创建单例。
