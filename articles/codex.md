### 一句话总结

B-Human 仓库可以先理解成：`Src` 是机器人程序本体，`Config` 是运行参数，`Make` 是构建/生成/部署脚本，`Util` 是仿真器和第三方工具，`Build` 是生成产物，`Install` 是安装/部署相关辅助内容。

### 架构图

```text
BHumanCodeRelease
├── Src          # C++ 主代码：App / Framework / Threads / Modules / Representations / Tools
├── Config       # 机器人、场景、模块参数、SimRobot 场景配置
├── Make         # CMake、平台构建脚本、部署脚本、检查工具入口
├── Util         # SimRobot、GameController、gtest、onnxruntime、第三方库
├── Build        # 构建生成目录，先不要作为源码入口分析
└── Install      # 安装、部署、root image 等流程相关，先不要深入扫描
```

### 数据流

```text
Config/*.cfg
   ↓
ConfigurationDataProvider / Framework
   ↓
Representations 作为数据结构
   ↓
Modules 读取/写入 Representations
   ↓
Threads 调度模块图
   ↓
MotionRequest
   ↓
MotionEngine / WalkingEngine / WalkGenerator
   ↓
JointRequest / ModifiedJointRequest
   ↓
机器人关节执行 / SimRobot 显示
```

### 关键文件表格

| 文件路径 | 作用 | 我需要看什么 |
|---|---|---|
| [Src](/home/fishros/BHumanCodeRelease/Src) | 主要源码目录 | 先看 `Apps`、`Threads`、`Libs/Framework`、`Modules/MotionControl`、`Representations` |
| [Src/Apps/Nao/Main.cpp](/home/fishros/BHumanCodeRelease/Src/Apps/Nao/Main.cpp) | 真机 Nao 程序入口 | 启动流程从哪里进入 Framework |
| [Src/Threads](/home/fishros/BHumanCodeRelease/Src/Threads) | 线程定义 | `Motion`、`Cognition`、`Perception`、`Audio` 怎么分工 |
| [Src/Libs/Framework](/home/fishros/BHumanCodeRelease/Src/Libs/Framework) | Module/Representation 框架核心 | `Module.h`、`Blackboard.h`、`ModuleGraphRunner.h` |
| [Src/Modules](/home/fishros/BHumanCodeRelease/Src/Modules) | 功能模块实现 | 重点看 `MotionControl`、`Sensing`、`Infrastructure` |
| [Src/Representations](/home/fishros/BHumanCodeRelease/Src/Representations) | 模块之间交换的数据结构 | `MotionRequest`、`JointRequest`、`WalkingEngineOutput` |
| [Config](/home/fishros/BHumanCodeRelease/Config) | 参数配置目录 | 机器人参数、场景参数、步态 cfg |
| [Config/Robots/Default/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngine.cfg) | 默认步态参数 | 后面做竞走/稳定步态时重点看 |
| [Config/Robots/Default/walkingEngineCommon.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngineCommon.cfg) | 通用步态参数 | 可能影响步频、支撑、摆腿、稳定控制 |
| [Config/Robots/Default/motionEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/motionEngine.cfg) | MotionEngine 配置 | 看运动选择和生成器配置 |
| [Config/Scenarios/CompetitionWalk/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg) | 竞走相关场景步态配置 | 很适合作为你目标任务的入口 |
| [Make](/home/fishros/BHumanCodeRelease/Make) | 构建、生成、部署脚本 | 先看 Linux 下的 `generate`、`compile`，不要急着全量编译 |
| [Make/CMake](/home/fishros/BHumanCodeRelease/Make/CMake) | CMake 项目描述 | 理解 target 如何对应 Src 里的 App/Lib |
| [Util](/home/fishros/BHumanCodeRelease/Util) | 工具和第三方依赖 | 重点看 `SimRobot`、`GameController`、`TestRunner` |
| [Util/SimRobot](/home/fishros/BHumanCodeRelease/Util/SimRobot) | 仿真器相关 | 后面调试步态、看 plots/scene 会用到 |
| `Build` | 构建产物目录 | 不是源码入口，除非排查编译结果，否则先跳过 |
| `Install` | 安装/部署/root image 相关 | 和部署到机器人有关，当前结构分析阶段先不深入 |

### 关键类/模块

- Framework 核心：  
  [Src/Libs/Framework/Module.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Module.h)、[Src/Libs/Framework/Blackboard.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Blackboard.h)、[Src/Libs/Framework/ModuleGraphRunner.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/ModuleGraphRunner.h)

- 线程入口：  
  [Src/Threads/Motion.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Motion.cpp)、[Src/Threads/Cognition.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Cognition.cpp)、[Src/Threads/Perception.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Perception.cpp)

- Motion 主链路：  
  [Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp)、[Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp)

- 步态生成器：  
  [Src/Modules/MotionControl/WalkingEngine/WalkGenerators](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkGenerators)、[Src/Representations/MotionControl/WalkGenerator.h](/home/fishros/BHumanCodeRelease/Src/Representations/MotionControl/WalkGenerator.h)

- 运动请求与关节输出：  
  [Src/Representations/MotionControl/MotionRequest.h](/home/fishros/BHumanCodeRelease/Src/Representations/MotionControl/MotionRequest.h)、[Src/Representations/Infrastructure/JointRequest.h](/home/fishros/BHumanCodeRelease/Src/Representations/Infrastructure/JointRequest.h)

### 对竞走任务的意义

你的目标是“竞走/无腾空/稳定步态”，所以最重要的结构不是行为层，而是：

```text
MotionRequest
 → MotionEngine
 → WalkingEngine / WalkGenerator
 → WalkingEngineOutput
 → JointRequest
```

其中 `Config/Robots/*/walkingEngine*.cfg` 决定大量步态参数，`WalkingEngine.cpp` 和 `WalkLibs` 决定相位、支撑、步长、髋部移动、稳定调节等核心逻辑。后面真正改代码时，应该先从配置和仿真调参入手，再动 WalkingEngine 源码。

### 下一步建议

1. 先分析启动流程：  
   [Src/Apps/Nao/Main.cpp](/home/fishros/BHumanCodeRelease/Src/Apps/Nao/Main.cpp) → [Src/Libs/Framework/Robot.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Robot.h) → [Src/Threads](/home/fishros/BHumanCodeRelease/Src/Threads)

2. 再分析 Module/Representation 机制：  
   [Src/Libs/Framework/Module.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Module.h)、[Src/Libs/Framework/Blackboard.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Blackboard.h)

3. 然后进入 Motion 线程：  
   [Src/Threads/Motion.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Motion.cpp)

4. 最后集中看步态链路：  
   [Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp)  
   [Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp)  
   [Config/Scenarios/CompetitionWalk/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg)  
   [Config/Robots/Default/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngine.cfg)

   ### 一句话总结

B-Human 真机启动流程是：`main()` 读取机器人身份和 `settings.cfg`，创建 `Robot`，`Robot` 读取 `threads.cfg`，按线程配置创建多个 `ModuleContainer`，每个线程在 `ThreadFrame::threadMain()` 里循环执行自己的模块图。

### 架构图

```text
Src/Apps/Nao/Main.cpp
  ↓
main()
  ↓
Settings(...)
  ↓
Robot(settings, "")
  ↓
Robot 读取 threads.cfg
  ↓
创建 Debug 线程
  ↓
为 threads.cfg 中每个线程创建 ModuleContainer
  ↓
ModuleContainer 根据 executionUnit 创建 Motion / Cognition / Perception ...
  ↓
ThreadList::start()
  ↓
ThreadFrame::threadMain()
  ↓
executionUnit->beforeFrame()
  ↓
moduleGraphRunner.execute()
  ↓
executionUnit->afterModules()
  ↓
executionUnit->afterFrame()
```

### 数据流

```text
Config/settings.cfg
  ↓ 决定 scenario/location/player 等
Settings::updateSearchPath()
  ↓ 生成 Config 搜索路径
threads.cfg
  ↓ 决定线程、优先级、ExecutionUnit、Representation Provider
Robot::Robot()
  ↓ 创建 ModuleContainer
ModuleContainer::main()
  ↓ 接收跨线程 ModulePacket
ModuleGraphRunner::execute()
  ↓ 执行本线程模块
Sender/Receiver
  ↓ 把 Representation 传给其他线程
```

### 关键文件表格

| 文件路径 | 作用 | 我需要看什么 |
|---|---|---|
| [Src/Apps/Nao/Main.cpp](/home/fishros/BHumanCodeRelease/Src/Apps/Nao/Main.cpp) | 真机主入口 | 参数解析、watchdog、LoLA 等待、创建 `Settings` 和 `Robot` |
| [Src/Libs/Framework/Settings.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Settings.cpp) | 配置搜索路径生成 | `updateSearchPath()` 决定 cfg 覆盖顺序 |
| [Config/settings.cfg](/home/fishros/BHumanCodeRelease/Config/settings.cfg) | 默认运行设置 | 当前 `scenario = Default`，所以默认读 `Config/Scenarios/Default` |
| [Config/Robots/robots.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/robots.cfg) | bodyId/headId 到机器人名字映射 | 真机启动时用 bodyId 找 bodyName |
| [Src/Libs/Framework/Robot.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Robot.cpp) | 创建线程列表 | 读取 `threads.cfg`，创建 `Debug` 和所有 `ModuleContainer` |
| [Config/Scenarios/Default/threads.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/Default/threads.cfg) | 默认线程/模块图配置 | `Upper`、`Lower`、`Cognition`、`Motion`、`Audio`、`Referee` |
| [Src/Libs/Framework/ThreadFrame.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/ThreadFrame.cpp) | 线程主循环 | `threadMain()` 如何 init、收 debug、调用 main、等待 |
| [Src/Libs/Framework/ModuleContainer.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/ModuleContainer.cpp) | 每个线程的模块容器 | 如何创建 ExecutionUnit、执行模块图、跨线程通信 |
| [Src/Libs/Framework/FrameExecutionUnit.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/FrameExecutionUnit.h) | 线程执行单元接口 | `beforeFrame/beforeModules/afterModules/afterFrame` |
| [Src/Threads/Motion.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Motion.cpp) | Motion 线程执行单元 | 等待关节/传感器帧，执行后 `finishFrame()` |
| [Src/Threads/Cognition.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Cognition.cpp) | Cognition 线程执行单元 | 如何从 Upper/Lower 感知帧中选择一帧 |
| [Src/Threads/Perception.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Perception.cpp) | 感知线程执行单元 | 等待 camera frame，执行图像处理模块 |

### 关键类/模块

`Main.cpp` 里最关键的是：

```cpp
Settings settings(...);
bhumanStart(settings);
```

`bhumanStart()` 会执行：

```cpp
robot = new Robot(settings, std::string());
robot->start();
```

`Robot::Robot()` 里真正搭框架：

```cpp
InMapFile stream("threads.cfg");
stream >> config;

push_back(new Debug(settings, name, config));

for(...)
  push_back(new ModuleContainer(...));
```

所以，**启动时不是硬编码创建 Motion/Cognition/Perception 线程，而是由 `threads.cfg` 配置驱动**。

`ModuleContainer` 再根据 `threads.cfg` 中的字段创建具体 ExecutionUnit：

```cfg
name = Motion;
priority = 20;
executionUnit = Motion;
representationProviders = [
  {representation = JointRequest; provider = MotionEngine;},
  {representation = WalkGenerator; provider = WalkingEngine;},
  ...
];
```

对应源码在：

- [Src/Libs/Framework/ModuleContainer.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/ModuleContainer.cpp)
- [Src/Threads/Motion.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Motion.cpp)

### 调用链

```text
main()
  -> FunctionList::execute()
  -> Settings(...)
     -> Settings::updateSearchPath()
  -> bhumanStart(settings)
     -> new Robot(settings, "")
        -> File::setSearchPath(settings.searchPath)
        -> read "threads.cfg"
        -> new Debug(...)
        -> new ModuleContainer(...) for each configured thread
        -> connectWithSender(...)
        -> connectWithDebug(...)
     -> robot->start()
        -> ThreadList::start()
        -> ThreadFrame::start()
        -> ThreadFrame::threadMain()
           -> setGlobals()
           -> init()
           -> while(isRunning())
              -> debugReceiver->receivePacket()
              -> handleAllMessages()
              -> ModuleContainer::main()
                 -> receive inter-thread packets
                 -> executionUnit->beforeFrame()
                 -> executionUnit->beforeModules()
                 -> moduleGraphRunner.execute()
                 -> executionUnit->afterModules()
                 -> send inter-thread packets
                 -> logger/debug output
                 -> executionUnit->afterFrame()
```

### 对竞走任务的意义

对“竞走/无腾空/稳定步态”来说，启动流程里最重要的点是：**Motion 线程是否启动、Motion 线程里哪些 Representation 由哪些 Provider 提供，完全由 `threads.cfg` 决定。**

默认配置中 Motion 线程的关键片段在 [Config/Scenarios/Default/threads.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/Default/threads.cfg)：

```cfg
name = Motion;
priority = 20;
executionUnit = Motion;
...
{representation = JointRequest; provider = MotionEngine;},
{representation = MotionInfo; provider = MotionEngine;},
{representation = OdometryData; provider = MotionEngine;},
{representation = StandGenerator; provider = WalkingEngine;},
{representation = WalkGenerator; provider = WalkingEngine;},
{representation = WalkingEngineOutput; provider = WalkingEngine;},
{representation = WalkStepData; provider = WalkingEngine;},
```

这说明步态主链路在 Motion 线程内：

```text
MotionRequest
  -> MotionEngine
  -> WalkingEngine / WalkGenerator
  -> WalkingEngineOutput
  -> JointRequest
```

其中 `Motion` 线程优先级是 `20`，明显高于默认的 `Cognition = 1`、`Upper/Lower = 0`，这符合运动控制要稳定周期执行的需求。

### 下一步建议

1. 下一步先分析 `threads.cfg` 的 Motion 段：  
   [Config/Scenarios/Default/threads.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/Default/threads.cfg)

2. 然后分析 `Module / Representation` 机制：  
   [Src/Libs/Framework/Module.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Module.h)  
   [Src/Libs/Framework/ModuleGraphRunner.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/ModuleGraphRunner.cpp)

3. 再进入 Motion 线程数据流：  
   [Src/Threads/Motion.cpp](/home/fishros/BHumanCodeRelease/Src/Threads/Motion.cpp)  
   [Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp)

4. 最后看步态核心：  
   [Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp)  
   [Config/Scenarios/CompetitionWalk/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg)

   ### 一句话总结

继续往下看，B-Human 的核心运行机制是：`threads.cfg` 指定每个线程中“哪个 Representation 由哪个 Module 提供”，`ModuleGraphCreator` 根据模块依赖排序，`ModuleGraphRunner` 每帧按顺序调用模块的 `update()`。

### 架构图

```text
Config/Scenarios/*/threads.cfg
  ↓
Configuration
  ↓
Debug 线程中的 ModuleGraphCreator
  ↓
计算：
  - 每个线程执行哪些 Module
  - Module 执行顺序
  - 哪些 Representation 跨线程传输
  ↓
通过 idModuleRequest 发给各 ModuleContainer
  ↓
每个线程里的 ModuleGraphRunner
  ↓
每帧执行：
  provider.update(representation)
```

### 数据流

以 Motion 线程为例：

```text
Cognition 线程
  SkillBehaviorControl
    ↓ PROVIDES(MotionRequest)
Motion 线程接收 MotionRequest
    ↓
WalkingEngine
    ↓ PROVIDES(WalkGenerator / WalkingEngineOutput / WalkStepData)
MotionEngine
    ↓ REQUIRES(WalkGenerator)
    ↓ PROVIDES(JointRequest)
NaoProvider / BoosterProvider
    ↓
发送给机器人硬件或仿真
```

### 关键文件表格

| 文件路径 | 作用 | 我需要看什么 |
|---|---|---|
| [Src/Libs/Framework/Module.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Module.h) | Module 机制宏定义 | `MODULE`、`REQUIRES`、`PROVIDES`、`LOADS_PARAMETERS`、`MAKE_MODULE` |
| [Src/Libs/Framework/Module.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Module.cpp) | 模块参数加载 | `loadModuleParameters()` 如何把模块名映射到 cfg 文件 |
| [Src/Libs/Framework/Configuration.h](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Configuration.h) | `threads.cfg` 数据结构 | `Thread`、`RepresentationProvider` |
| [Src/Libs/Framework/ModuleGraphCreator.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/ModuleGraphCreator.cpp) | 从配置生成模块执行图 | 依赖检查、跨线程数据计算、拓扑排序 |
| [Src/Libs/Framework/ModuleGraphRunner.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/ModuleGraphRunner.cpp) | 每个线程实际执行模块 | `execute()` 里逐个调用 provider 的 `update()` |
| [Src/Libs/Framework/Debug.cpp](/home/fishros/BHumanCodeRelease/Src/Libs/Framework/Debug.cpp) | 模块图下发与调试入口 | `idModuleRequest`、`moduleGraph:moduleOrder`、`moduleGraph:dataExchanged` |
| [Src/Modules/MotionControl/MotionEngine/MotionEngine.h](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.h) | MotionEngine 模块声明 | 它 `REQUIRES(MotionRequest/WalkGenerator)`，`PROVIDES(JointRequest)` |
| [Src/Modules/MotionControl/WalkingEngine/WalkingEngine.h](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.h) | WalkingEngine 模块声明 | 它读传感器/步态参数，提供 `WalkGenerator` 等 |
| [Src/Representations/MotionControl/MotionRequest.h](/home/fishros/BHumanCodeRelease/Src/Representations/MotionControl/MotionRequest.h) | 行为层发给运动层的请求 | `motion` 枚举和 walk 相关字段 |
| [Src/Representations/Infrastructure/JointRequest.h](/home/fishros/BHumanCodeRelease/Src/Representations/Infrastructure/JointRequest.h) | 最终关节请求 | 关节角 + stiffness |

### 关键类/模块

`MODULE(...)` 是每个模块的“契约声明”。例如 [MotionEngine.h](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.h) 中：

```cpp
MODULE(MotionEngine,
{,
  REQUIRES(MotionRequest),
  REQUIRES(WalkGenerator),
  PROVIDES(JointRequest),
  PROVIDES(MotionInfo),
  PROVIDES(OdometryData),
  LOADS_PARAMETERS(...)
});
```

含义是：

```text
MotionEngine 需要 MotionRequest、WalkGenerator 等输入
MotionEngine 输出 JointRequest、MotionInfo、OdometryData
MotionEngine 参数从 motionEngine.cfg 加载
```

`MAKE_MODULE(MotionEngine)` 在 [MotionEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp) 中注册模块，让框架能按名字创建它。

### 调用链

```text
Debug::init()
  -> new ModuleGraphCreator(config)
  -> moduleGraphCreator->update(config stream)
  -> getExecutionValues(threadIndex)
  -> sender.bin(idModuleRequest) 发给各线程

ModuleContainer::handleMessage()
  -> 收到 idModuleRequest
  -> moduleGraphRunner.update(stream)

ModuleContainer::main()
  -> moduleGraphRunner.execute()

ModuleGraphRunner::execute()
  -> for Provider in providers:
       如果模块实例不存在，createNew()
       调用 update(representation)
```

### 对竞走任务的意义

要做“竞走/无腾空/稳定步态”，你不能只看 `WalkingEngine.cpp`，还要知道它什么时候被调用、输入从哪里来、输出交给谁。

核心链路是：

```text
MotionRequest
  provider = SkillBehaviorControl
  thread = Cognition

↓ 跨线程传给 Motion

WalkingEngine
  REQUIRES(MotionRequest)
  PROVIDES(WalkGenerator)
  PROVIDES(WalkingEngineOutput)
  PROVIDES(WalkStepData)

↓ 同线程内被 MotionEngine 使用

MotionEngine
  REQUIRES(WalkGenerator)
  PROVIDES(JointRequest)
```

也就是说：

- 行为层决定“要不要走、往哪走”：`MotionRequest`
- 步态层决定“脚怎么走、身体怎么平衡”：`WalkingEngine`
- 运动总控决定“当前 MotionPhase 怎么转成关节”：`MotionEngine`
- 最终硬件执行看：`JointRequest`

### 下一步建议

下一步建议分析 **Motion 线程完整数据流**，重点看：

1. [Config/Scenarios/Default/threads.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/Default/threads.cfg) 中 `Motion` 段  
   看 `MotionRequest`、`WalkGenerator`、`JointRequest` 分别由谁提供。

2. [Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp)  
   看它如何根据 `MotionRequest` 选择 walk/stand/kick/getup 等 phase。

3. [Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp)  
   看 `WalkGenerator` 和 `WalkingEngineOutput` 怎么生成。

4. [Config/Scenarios/CompetitionWalk/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg)  
   这是你的竞走目标最直接入口，先理解它覆盖了哪些默认步态参数。、

### 一句话总结

`CompetitionWalk` 的“竞走逻辑”目前主要不是一套独立行为，而是通过 `Config/Scenarios/CompetitionWalk/walkingEngine.cfg` 覆盖 WalkingEngine 参数，让默认步态走得更快、更高髋、更偏持续稳定行走；真正的步态执行仍走 `MotionRequest -> MotionEngine -> WalkingEngine -> WalkPhase -> JointRequest` 这条链。

### 架构图

```text
行为层 / Joystick / Console
  ↓
MotionRequest
  motion = walkAtRelativeSpeed / walkToPose / walkAtAbsoluteSpeed
  energySavingWalk = true/false
  ↓
MotionEngine
  选择 WalkGenerator 对应的运动相位
  ↓
WalkingEngine
  读取 walkingEngine.cfg + walkingEngineCommon.cfg
  ↓
WalkPhase / WalkDelayPhase / WalkHipShiftPhase
  生成左右脚轨迹、支撑脚切换、髋高、足端高度、陀螺仪平衡
  ↓
InverseKinematic
  ↓
JointRequest
```

### 数据流

```text
Config/Scenarios/CompetitionWalk/walkingEngine.cfg
  ↓ 覆盖
configuredParameters
  ↓
WalkingEngine::update(WalkGenerator)
  ↓
WalkGenerator.createPhase(...)
  ↓
WalkPhase(...)
  ↓
WalkPhase::update()
  - 更新时间
  - 动态步长
  - 髋高
  - 支撑脚切换判断
  ↓
WalkPhase::calcJoints()
  - 计算脚位置
  - 平衡脚姿态
  - 逆运动学
  - 陀螺仪补偿
  - 关节间隙补偿
  ↓
MotionEngine::update(JointRequest)
```

### 关键文件表格

| 文件路径 | 作用 | 我需要看什么 |
|---|---|---|
| [Config/Scenarios/CompetitionWalk/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg) | 竞走场景步态覆盖参数 | 速度、动态步更新、energySavingWalk |
| [Config/Robots/Default/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngine.cfg) | 默认 WalkingEngine 参数 | 和 CompetitionWalk 对比 |
| [Config/Robots/Default/walkingEngineCommon.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngineCommon.cfg) | 通用步态参数 | 髋高、脚抬高、支撑切换、平衡参数 |
| [Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp) | 步态核心实现 | `WalkPhase::update()`、`calcFootOffsets()`、`calcJoints()` |
| [Src/Modules/MotionControl/WalkingEngine/WalkingEngine.h](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.h) | 参数结构定义 | `ConfiguredParameters`、`KinematicParameters`、`BalanceParameters` |
| [Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp) | 运动总控 | 如何从 `MotionRequest` 创建 walk phase |
| [Src/Representations/MotionControl/MotionRequest.h](/home/fishros/BHumanCodeRelease/Src/Representations/MotionControl/MotionRequest.h) | 行为层运动请求 | `walkAtRelativeSpeed`、`walkToPose`、`energySavingWalk` |
| [Src/Representations/MotionControl/WalkGenerator.h](/home/fishros/BHumanCodeRelease/Src/Representations/MotionControl/WalkGenerator.h) | WalkingEngine 暴露给 MotionEngine 的接口 | `createPhase`、`isNextLeftPhase`、`getTranslationPolygon` |
| [Src/Representations/MotionControl/WalkingEngineOutput.h](/home/fishros/BHumanCodeRelease/Src/Representations/MotionControl/WalkingEngineOutput.h) | 输出给行为层的步态能力 | 最大速度、最大步长、步周期 |
| [Src/Modules/BehaviorControl/SkillBehaviorControl/Skills/Walk/WalkToPoint.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Skills/Walk/WalkToPoint.cpp) | 行为层走到目标点 | 如何设置 `MotionRequest.energySavingWalk` |
| [Src/Modules/BehaviorControl/SkillBehaviorControl/Options/HandleFastestWalkLeaderboard.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Options/HandleFastestWalkLeaderboard.cpp) | 最快走路挑战逻辑 | 可参考如何发高速走路请求 |

### 关键类/模块

`CompetitionWalk` 与默认参数的主要差异在 [Config/Scenarios/CompetitionWalk/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg)：

```text
Default:
  maxSpeed x = 230
  minSpeed x = 200
  rotation = 90deg
  energySavingWalk = false
  dynamicStepUpdate = false

CompetitionWalk:
  maxSpeed x = 270
  minSpeed x = 250
  rotation = 120deg / 100deg
  energySavingWalk = true
  dynamicStepUpdate = true
```

含义：

- `maxSpeed/minSpeed`：允许更大的前进速度和旋转速度。
- `dynamicStepUpdate = true`：步子执行过程中允许更新步目标，更适合连续竞走/追目标。
- `energySavingWalk = true`：允许在满足条件时提高髋高，减少膝盖弯曲和发热。
- `walkSpeedParams.maxSpeed.y = 270`：理论最大侧向能力也更大，但注释里明确说 `walkSpeedParams` 不是普通走路随便用的上限。

### 竞走核心逻辑

#### 1. 竞走请求从行为层来

行为层最终会写 `MotionRequest`，例如：

[WalkAtRelativeSpeed.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Skills/Output/MotionRequest/WalkAtRelativeSpeed.cpp)

```cpp
theMotionRequest.motion = MotionRequest::walkAtRelativeSpeed;
theMotionRequest.walkSpeed = speed;
theMotionRequest.energySavingWalk = energySavingWalk;
```

[WalkToPose.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Skills/Output/MotionRequest/WalkToPose.cpp)

```cpp
theMotionRequest.motion = MotionRequest::walkToPose;
theMotionRequest.walkTarget = target;
theMotionRequest.walkSpeed = speed;
theMotionRequest.energySavingWalk = energySavingWalk;
```

所以“竞走”不是自动发生的，必须有行为层、Joystick 或 SimRobot console 发出走路请求。

#### 2. MotionEngine 选择 WalkingEngine

[MotionEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp) 中：

```cpp
generators[MotionRequest::walkAtRelativeSpeed] = &theWalkAtRelativeSpeedGenerator;
generators[MotionRequest::walkToPose] = &theWalkToPoseGenerator;
```

然后每帧：

```cpp
newPhase = generators[motionRequest.motion]->createPhase(motionRequest, *phase);
phase->calcJoints(...);
```

也就是说 `MotionEngine` 不直接算脚，它负责选择当前 phase，并最终生成 `JointRequest`。

#### 3. WalkingEngine 生成 WalkPhase

[WalkingEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp) 里 `walkGenerator.createPhase` 会决定下一步是：

```text
WalkPhase
WalkDelayPhase
WalkHipShiftPhase
```

其中：

- `WalkPhase` 是正常一步。
- `WalkDelayPhase` 是延迟相位，用来等重心/支撑更稳定。
- `WalkHipShiftPhase` 是主动髋部侧移相位，当前默认 `useHipShift = false`，所以一般走 `WalkDelayPhase` 机制。

#### 4. 无腾空主要靠支撑脚逻辑保证

在 `WalkPhase::update()` 中，支撑脚切换不是纯时间驱动，而是结合 FSR/FootSupport：

```cpp
isSwitchAllowed = tBase > supportSwitchPhaseRange.min * stepDuration;
isNormalSwitch = isSwitchAllowed && theFootSupport.switched;
isOvertimeSwitch = tBase > supportSwitchPhaseRange.max * stepDuration;
```

`CompetitionWalk` 里：

```cfg
supportSwitchPhaseRange = { min = 0.5; max = 10; };
```

意义是：

- 一步至少走到 50% 相位之后，才允许正常切换支撑脚。
- 如果一直没有检测到支撑切换，最多可以拖很久，避免过早换脚导致不稳。
- 这对“无腾空”很关键：不是到了时间就强行认为脚换了，而是等实际支撑状态。

#### 5. 脚高度由 `baseFootLift` 控制

在 [walkingEngineCommon.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngineCommon.cfg)：

```cfg
kinematicParameters = {
  baseWalkPeriod = 250;
  walkHipHeight = { min = 230; max = 240; };
  baseFootLift = 13;
};
```

在 `WalkPhase::calcFootOffsets()` 中：

```cpp
footHeightSwing = maxFootHeight * parabolicFootHeight(...);
footHeightSupport = footHeightSupport0;
```

含义：

- 支撑脚高度基本保持在地面附近。
- 摆动脚按抛物线抬起。
- 当前不是“完全拖地竞走”，而是低抬脚双足步态。
- 如果你追求更严格的“竞走/无腾空/低冲击”，`baseFootLift` 是后续重点参数之一。

#### 6. 稳定由陀螺仪、足底压力、关节间隙补偿共同做

在 `WalkPhase::calcJoints()` 中有几层稳定控制：

```text
RobotStableState.predictRotation()
JointSpeedController
balanceFeetPoses()
InverseKinematic::calcLegJoints()
addGyroBalance()
jointPlayOffsetController->update()
```

相关参数在 [walkingEngineCommon.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngineCommon.cfg)：

```cfg
balanceParameters
walkStepAdjustmentParams
soleRotationParameter
speedControlParams
jointPlayOffsetParameters
walkDelayParameters
sideStabilizeParameters
```

### 对竞走任务的意义

当前 `CompetitionWalk` 已经做了两件事：

1. **提高速度上限**  
   前进速度从默认 `230` 提到 `270`，最小速度也从 `200` 提到 `250`，更像“持续快走”。

2. **打开节能高髋步态**  
   `energySavingWalk = true`，并且 `walkingEngineCommon.cfg` 中 `walkHipHeight = { min = 230; max = 240; }`。  
   `WalkingEngine::updateWalkHipHeight()` 会在低/中速时把髋高提高到接近 `240`，这会让膝盖弯曲更少，更像竞走，也更省热。

但它还没有显式保证：

```text
任意时刻至少一只脚有足底压力
摆动脚永不抬得过高
支撑切换必须严格基于接触确认
```

这些目前是 WalkingEngine 机制间接保证的，不是专门为“竞走规则”写的硬约束。

### 下一步建议

优先看这 4 个点：

1. **先调配置，不急着改代码**  
   看 [Config/Scenarios/CompetitionWalk/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg) 和 [Config/Robots/Default/walkingEngineCommon.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngineCommon.cfg)。

2. **重点观察这些参数**  
   `baseFootLift`、`baseWalkPeriod`、`walkHipHeight`、`supportSwitchPhaseRange`、`balanceParameters`、`walkDelayParameters`。

3. **用 SimRobot/Plot 看脚高度和足底压力**  
   重点 plot：
   `module:WalkingEngine:current:left:z`、`module:WalkingEngine:request:left:z`、`module:WalkingEngine:current:right:z`、`module:WalkingEngine:request:right:z`。

4. **如果后面要改代码，入口在这里**  
   [WalkingEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp)：
   `WalkPhase::update()` 管支撑脚切换，`WalkPhase::calcFootOffsets()` 管脚轨迹高度，`WalkPhase::calcJoints()` 管稳定和逆运动学。


---
   ### 一句话总结

不要直接“调用 WalkingEngine/LinePerceptor 函数”，而是在 Cognition 的行为层写一个竞走 Option：读取白线 `FieldLines`，输出前进 `MotionRequest`，再让 Motion 线程自动把它变成 `JointRequest`。

### 架构图

```text
相机/仿真图像
  -> ECImage / RelativeFieldColors / ScanGrid
  -> ScanLineRegionizer
  -> LinePerceptor
  -> LinesPercept / CirclePercept
  -> FieldLinesProvider
  -> FieldLines / FieldLineIntersections
  -> Cognition: SkillBehaviorControl
  -> MotionRequest: walkAtRelativeSpeed / walkToPose
  -> Motion: MotionEngine
  -> WalkAtSpeedEngine / WalkGenerator
  -> WalkingEngine
  -> JointRequest
  -> NaoProvider / 机器人关节
```

### 数据流

前进走路链路：

```text
SkillBehaviorControl
  -> WalkAtRelativeSpeed / WalkToPoint
  -> MotionRequest.motion = walkAtRelativeSpeed
  -> MotionEngine 选择 WalkAtRelativeSpeedGenerator
  -> WalkAtSpeedEngine 计算下一步 stepTarget
  -> WalkingEngine 生成步态相位
  -> JointRequest
```

白线识别链路：

```text
CameraImage
  -> ECImage
  -> RelativeFieldColors
  -> ScanLineRegionizer: 白色/绿色扫描线区域
  -> LinePerceptor: 白线 spot + line fitting
  -> LinesPercept
  -> FieldLinesProvider: 过滤圆圈/球误检，生成 FieldLines
  -> Cognition 可读取 FieldLines 做控制
```

### 关键文件表格

| 文件路径 | 作用 | 我需要看什么 |
|---|---|---|
| [MotionRequest.h](/home/fishros/BHumanCodeRelease/Src/Representations/MotionControl/MotionRequest.h:24) | 行为层给运动层的请求 | `walkAtRelativeSpeed`、`walkSpeed` |
| [WalkAtRelativeSpeed.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Skills/Output/MotionRequest/WalkAtRelativeSpeed.cpp:11) | 最直接的“往前走”调用 | 怎么设置 `theMotionRequest` |
| [WalkToPoint.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Skills/Walk/WalkToPoint.cpp:14) | 走到目标点的高级封装 | 是否走直线、避障、站停 |
| [HandleFastestWalkLeaderboard.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Options/HandleFastestWalkLeaderboard.cpp:3) | 现成“快速走路挑战”例子 | 可参考它写竞走 Option |
| [MotionEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/MotionEngine/MotionEngine.cpp:43) | 根据 `MotionRequest` 选择运动生成器 | 走路请求如何进入 WalkingEngine |
| [WalkAtSpeedEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkGenerators/WalkAtSpeedEngine.cpp:35) | 把相对速度转为步长 | 前进速度如何变成每一步 |
| [WalkingEngine.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp:467) | 核心步态生成 | 步长、支撑脚、抬脚、平衡 |
| [WalkingEngine.h](/home/fishros/BHumanCodeRelease/Src/Modules/MotionControl/WalkingEngine/WalkingEngine.h:146) | 步态参数定义 | `baseWalkPeriod`、`baseFootLift`、平衡参数 |
| [walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg:1) | 竞走场景已有步态参数覆盖 | 先从这里调速度 |
| [walkingEngineCommon.cfg](/home/fishros/BHumanCodeRelease/Config/Robots/Default/walkingEngineCommon.cfg:1) | 通用步态参数 | 抬脚高度、步频、加速度、平衡 |
| [LinePerceptor.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/Perception/FieldPerceptors/LinePerceptor.cpp:23) | 白线检测核心 | 扫描白色区域、拟合线、白色校验 |
| [LinePerceptor.h](/home/fishros/BHumanCodeRelease/Src/Modules/Perception/FieldPerceptors/LinePerceptor.h:36) | 白线模块输入输出 | `REQUIRES`/`PROVIDES` 依赖 |
| [linePerceptor.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/Default/linePerceptor.cfg:1) | 白线阈值配置 | 白色比例、线宽、最少 spot |
| [ScanLineRegionizer.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/Perception/Scanlines/ScanLineRegionizer.cpp:21) | 图像扫描线分割 | 怎么把区域标成 white/field |
| [RelativeFieldColors.h](/home/fishros/BHumanCodeRelease/Src/Representations/Perception/ImagePreprocessing/RelativeFieldColors.h:44) | 白/绿相对颜色判断 | `isWhiteNearField` |
| [FieldLinesProvider.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/Perception/FieldPerceptors/FieldLinesProvider.cpp:21) | 把 `LinesPercept` 变成可用场地线 | 过滤圆圈、球误检、排序 |
| [threads.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/Default/threads.cfg:35) | 线程和模块绑定 | Perception/Cognition/Motion 数据怎么连 |

### 关键类/模块

前进走路你主要用这些：

- `WalkAtRelativeSpeed`：最简单，直接给相对速度，例如前进 `x = 0.4`。
- `WalkToPoint`：走到某个相对/全局目标，适合“沿跑道走到终点”。
- `MotionRequest`：行为层输出。
- `MotionEngine`：运动总调度。
- `WalkingEngine`：真正生成稳定步态和关节角。

白线识别你主要用这些：

- `ScanLineRegionizer`：先把图像切成白色/绿色区域。
- `LinePerceptor`：识别白线点和线段。
- `LinesPercept`：原始白线感知结果。
- `FieldLinesProvider`：生成更干净的 `FieldLines`。
- `FieldLines`：你在竞走控制代码里最应该读取的白线结果。

### 对竞走任务的意义

竞走的控制层建议这样拆：

```text
白线检测 FieldLines
  -> 判断跑道方向 / 边线 / 终点线
  -> 生成方向误差 headingError 和横向误差 yError
  -> 输出 WalkAtRelativeSpeed
  -> WalkingEngine 负责稳定步态
```

最小控制逻辑可以是：

```cpp
LookForward();

float forward = 0.35f;
float turn = 0.f;

if(!theFieldLines.lines.empty())
{
  // 选最长或最近的线，估计方向误差
  // turn = -kHeading * lineAngle - kY * lineCenterY;
}

WalkAtRelativeSpeed({.speed = {turn, forward, 0.f},
                     .energySavingWalk = false});
```

如果只是“直走 + 看见白线停下”，更简单：`theFieldLines.lines` 里出现足够长的横向线，就 `Stand()`，否则 `WalkAtRelativeSpeed()`。

### 下一步建议

1. 先在 SimRobot 里验证现有前进：用 [Walk.con](/home/fishros/BHumanCodeRelease/Config/Scenes/Plots/Walk.con:118) 里的 `MotionRequest motion = walkAtRelativeSpeed; walkSpeed.translation.x = 1`。
2. 打开白线调试视图：重点看 `representation:LinesPercept:image`、`representation:LinesPercept:field`、`representation:FieldLines:field`、`module:LinePerceptor:spots`。
3. 新增竞走行为时，优先仿照 [HandleFastestWalkLeaderboard.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Options/HandleFastestWalkLeaderboard.cpp:3) 写 `HandleCompetitionWalk`。
4. 如果行为层要直接读白线，需要在 [SkillBehaviorControl.h](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/SkillBehaviorControl.h:93) 加 `REQUIRES(FieldLines)`，并包含 `FieldLines.h`。
5. 在 `Config/Scenarios/CompetitionWalk/skillBehaviorControl.cfg` 里把你的 `HandleCompetitionWalk` 加进 `options`，这样不用污染默认足球行为。
6. 步态优化先改 cfg：`Config/Scenarios/CompetitionWalk/walkingEngine.cfg` 和必要时新建场景覆盖版 `walkingEngineCommon.cfg`。先调低 `maxSpeed`、加大 `baseWalkPeriod`、降低 `baseFootLift`、限制加速度。
7. 真要改步态算法时，再碰 `WalkingEngine.cpp` / `WalkPhaseBase.cpp`；这是最后一步，不建议一开始就改。

这次我只翻找和分析了文件，没有修改任何代码。

---
### 一句话总结

B-Human 是“配置驱动的多线程 Module/Representation 框架”：行为层产出 `MotionRequest`，运动层产出 `JointRequest`，感知层产出 `FieldLines` 等环境信息。

### 架构图

```text
启动
 -> Settings / searchPath
 -> threads.cfg
 -> Robot
 -> ModuleContainer per thread
 -> ModuleGraphRunner
 -> Modules update Representations
```

```text
Perception 线程
CameraImage -> ECImage -> ScanLineRegionizer -> LinePerceptor -> FieldLines

Cognition 线程
FieldLines / RobotPose / BallModel -> SkillBehaviorControl -> MotionRequest

Motion 线程
MotionRequest -> MotionEngine -> WalkingEngine -> JointRequest -> NaoProvider
```

### 关键层级

| 层级 | 作用 | 关键文件 |
|---|---|---|
| App 启动 | 创建 `Settings` 和 `Robot` | `Src/Apps/Nao/Main.cpp` |
| Framework | 线程、模块图、黑板、通信 | `Src/Libs/Framework/Robot.cpp`, `ModuleContainer.cpp`, `ModuleGraphRunner.cpp` |
| Threads | Perception/Cognition/Motion 调度 | `Src/Threads/*.cpp` |
| Modules | 实际算法模块 | `Src/Modules/**` |
| Representations | 线程间/模块间数据结构 | `Src/Representations/**` |
| Config | 决定哪些模块跑、参数是多少 | `Config/Scenarios/**`, `Config/Robots/**` |

### Module / Representation 机制

模块用宏声明依赖：

```cpp
MODULE(SomeModule,
{,
  REQUIRES(A),
  PROVIDES(B),
  LOADS_PARAMETERS(...)
});
```

实际注册：

```cpp
MAKE_MODULE(SomeModule);
```

`threads.cfg` 决定：

```text
representation = MotionRequest; provider = SkillBehaviorControl;
representation = JointRequest;  provider = MotionEngine;
representation = FieldLines;    provider = FieldLinesProvider;
```

所以不是手动调用模块，而是配置“谁提供哪个 Representation”。

### Motion 主链路

| 阶段 | 文件 |
|---|---|
| 行为生成走路请求 | `SkillBehaviorControl` |
| 最简单前进技能 | `Skills/Output/MotionRequest/WalkAtRelativeSpeed.cpp` |
| Motion 总调度 | `Modules/MotionControl/MotionEngine/MotionEngine.cpp` |
| 速度转步长 | `WalkingEngine/WalkGenerators/WalkAtSpeedEngine.cpp` |
| 步态核心 | `WalkingEngine/WalkingEngine.cpp` |
| 最终关节请求 | `Representations/Infrastructure/JointRequest.h` |

核心流：

```text
WalkAtRelativeSpeed()
 -> MotionRequest.motion = walkAtRelativeSpeed
 -> MotionEngine
 -> WalkAtRelativeSpeedGenerator
 -> WalkAtSpeedEngine
 -> WalkGenerator
 -> WalkingEngine
 -> JointRequest
```

### 白线识别主链路

| 阶段 | 文件 |
|---|---|
| 图像预处理 | `ECImageProvider.cpp` |
| 颜色阈值 | `RelativeFieldColors.h` |
| 扫描线分割 | `ScanLineRegionizer.cpp` |
| 白线检测 | `LinePerceptor.cpp` |
| 原始线表示 | `LinesPercept.h` |
| 场地线过滤/整理 | `FieldLinesProvider.cpp` |
| 可用于行为层的结果 | `FieldLines.h` |

核心流：

```text
CameraImage
 -> ECImage
 -> ColorScanLineRegions
 -> LinesPercept
 -> FieldLines
 -> Cognition / SelfLocator
```

### 对竞走任务的意义

你最终要做的是一个新的行为控制逻辑：

```text
读 FieldLines
 -> 判断跑道线/终点线/偏航误差
 -> 输出 WalkAtRelativeSpeed 或 WalkToPose
 -> 调 cfg 优化稳定步态
```

不要一开始改 `WalkingEngine.cpp`。优先顺序应是：

1. 先用现有 `WalkAtRelativeSpeed` 实现稳定直走。
2. 再读取 `FieldLines` 做白线跟随或停线判断。
3. 最后才调 `walkingEngine.cfg` / `walkingEngineCommon.cfg`。
4. 算法不够时，再考虑改 `WalkingEngine`。

### 我建议你下一步看

1. `Config/Scenarios/Default/threads.cfg`
2. `Src/Modules/BehaviorControl/SkillBehaviorControl/Options/HandleFastestWalkLeaderboard.cpp`
3. `Src/Modules/BehaviorControl/SkillBehaviorControl/Skills/Output/MotionRequest/WalkAtRelativeSpeed.cpp`
4. `Src/Modules/Perception/FieldPerceptors/LinePerceptor.cpp`
5. `Src/Modules/MotionControl/WalkingEngine/WalkingEngine.cpp`

本轮没有修改任何代码。


---
### 一句话总结

我看了，现有最适合参考的是 `FastestWalkLeaderboard`，但它只做“按定位走到终点”，没有用白线；如果你的竞走要“识别白线 + 前进”，需要新增一个行为 Option 来读 `FieldLines`。

### 我看到的关键点

| 文件路径 | 结论 |
|---|---|
| [HandleFastestWalkLeaderboard.cpp](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/Options/HandleFastestWalkLeaderboard.cpp:3) | 现成快速走路逻辑，用 `WalkToPoint` 前进 |
| [FastestWalkLeaderboard/skillBehaviorControl.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/FastestWalkLeaderboard/skillBehaviorControl.cfg:1) | 通过 `HandleFastestWalkLeaderboard` 启用该行为 |
| [FastestWalkLeaderboard/threads.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/FastestWalkLeaderboard/threads.cfg:130) | `FieldLines` 已经传到 Cognition 线程 |
| [SkillBehaviorControl.h](/home/fishros/BHumanCodeRelease/Src/Modules/BehaviorControl/SkillBehaviorControl/SkillBehaviorControl.h:93) | 目前 `SkillBehaviorControl` 没有 `REQUIRES(FieldLines)`，所以行为层还不能直接用 `theFieldLines` |
| [CompetitionWalk/walkingEngine.cfg](/home/fishros/BHumanCodeRelease/Config/Scenarios/CompetitionWalk/walkingEngine.cfg:1) | 现在只是步态参数覆盖，不是完整竞走行为入口 |

### 现有 FastestWalk 是怎么走的

```text
HandleFastestWalkLeaderboard
 -> 根据 RobotPose + FieldDimensions 算终点
 -> WalkToPoint
 -> WalkToPose
 -> MotionRequest
 -> MotionEngine
 -> WalkingEngine
 -> JointRequest
```

它的前进代码核心是：

```cpp
WalkToPoint({
  .target = finishPose,
  .reduceWalkSpeedType = ReduceWalkSpeedType::noChange,
  .disableObstacleAvoidance = true,
  .disableStanding = true
});
```

### 对你的竞走任务的判断

你不要直接改 `WalkingEngine`。更合理路线是：

1. 仿照 `HandleFastestWalkLeaderboard.cpp` 新建 `HandleCompetitionWalk`。
2. 在这个 Option 里读取 `FieldLines`。
3. 根据白线算偏航误差和横向偏移。
4. 输出 `WalkAtRelativeSpeed` 或 `WalkToPoint`。
5. 用 `CompetitionWalk/walkingEngine.cfg` 调稳定步态参数。

### 如果要用白线，必须关注

白线已经从感知线程传到了 Cognition，但 `SkillBehaviorControl` 当前没有声明依赖，所以以后真要写代码时要改：

```text
Src/Modules/BehaviorControl/SkillBehaviorControl/SkillBehaviorControl.h
```

需要加：

```cpp
#include "Representations/Perception/FieldPercepts/FieldLines.h"
REQUIRES(FieldLines)
```

然后你的 Option 才能用：

```cpp
theFieldLines.lines
```

本轮只分析，没有修改任何代码。
