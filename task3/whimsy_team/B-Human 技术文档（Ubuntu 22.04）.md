------

# B-Human 开发环境配置与部署指南（Ubuntu 22.04）

## 一、设置开发环境

在 Ubuntu 终端中执行以下命令安装依赖：

```
sudo apt install ccache clang cmake git graphviz libasound2-dev libbox2d-dev \
libgl-dev libqt6opengl6-dev libqt6svg6-dev libstdc++-12-dev llvm mold \
net-tools ninja-build pigz qt6-base-dev rsync xxd
```

------

## 二、克隆 B-Human 源码

> **注意**：必须使用 `--recursive`，因为 B-Human 使用 Git 子模块。

```
git clone --recursive https://github.com/bhuman/BHumanCodeRelease.git
```

或者下载 zip/tar.gz 后，将缺失的子模块手动补齐。

------

## 三、编译 B-Human 代码

进入 Linux 编译目录：

```
cd BHumanCodeRelease/Make/Linux/
```

生成编译文件：

```
./generate
```

编译：

```
./compile
```

编译完成后，生成目录：

```
Build/Linux/
  ├── CMake
  ├── DeployDialog
  ├── Nao
  ├── SimRobot
  └── Tests
```

可执行文件 `DeployDialog` 位于：

```
Build/Linux/DeployDialog/Develop/DeployDialog
```

------

## 四、配置 NAO 团队信息

编辑：

```
BHumanCodeRelease/Config/settings.cfg
```

示例（SPL 分配编号 103）：

```
teamNumber = 103;
fieldPlayerColor = black;
goalkeeperColor = blue;
playerNumber = 3;
location = Default;
scenario = Default;
magicNumber = 103;
```

------

## 五、网络配置

### 5.1 IP 规则（默认即可）

- 有线：`192.168.<team>.<robot>`
- 无线：`10.0.<team>.<robot>`

配置位置：

```
BHumanCodeRelease/Install/createRobot
```

------

### 5.2 无线网络配置

配置文件位置：

```
BHumanCodeRelease/Install/Profiles
```

示例（`SPL_E` 配置）：

```
"SPL_E":
  password: "Nao?!Nao?!"
```

在部署时可通过 `-w` 选项选择网络配置，例如：

```
-w SPL_E
```

选择 `NONE` 则关闭无线功能。

------

## 六、创建机器人配置

进入 Install 目录：

```
cd BHumanCodeRelease/Install
```

### 方法一（机器人在线且运行 Robocup 基镜像）

```
./createRobot [-t <team>] -r <robot> -i <ip> <name>
```

- `<team>`：团队编号（默认 `settings.cfg` 中的 teamNumber）
- `<robot>`：机器人编号（决定 IP 的最后一位）
- `<ip>`：当前机器人 IP
- `<name>`：给机器人起的名字

------

### 方法二（推荐，适用于所有情况）

```
./createRobot [-t <team>] -r <robot> -s <headId> -b <bodyId> <name>
```

- `<headId>` / `<bodyId>`：机器人头部和身体序列号（P 开头的 20 位字符）
- `<name>`：机器人名称

------

## 七、创建根镜像（Root Image）

安装所需依赖：

```
sudo apt install bzip2 debootstrap patchelf
```

创建根镜像：

```
sudo Install/createRootImage <path-to-original-Aldebaran-OPN>
```

生成文件：

```
Install/root.ext3
```

------

## 八、制作 B-Human 镜像并刷机

在 Make/Common 目录生成 `.opn` 镜像：

```
cd BHumanCodeRelease/Make/Common
./deploy Develop -i -nc -v 50 -w SPL_E
```

- `-i`：制作镜像
- `-nc`：不清除缓存
- `-v 50`：压缩等级
- `-w SPL_E`：无线配置文件

生成的镜像文件位于：

```
Build/image/bhuman.opn
```

使用 **NaoFlasher** 工具将 `.opn` 文件刷入 U 盘，然后给 NAO 刷机。

------

## 九、部署软件（无需重新刷机）

在机器人已连接网络的情况下，可以直接部署：

```
./deploy Develop <robotName> -w SPL_E
```

 其中 `<robotName>` 是 `createRobot` 时定义的机器人名称。 

---

# B-Human 框架中“行为/点球”相关的结构（你需要知道的核心概念）

## 一、线程 / 模块 / 表示（representations）

- B-Human 使用**blackboard、MODULE、representations** 的模块框架。模块通过 `REQUIRES(...)` / `PROVIDES(...)` 声明输入/输出的 representation；框架会自动排序模块执行顺序。模块的参数可以用 `DEFINES_PARAMETERS` 或 `LOADS_PARAMETERS`（默认从对应的 `.cfg` 文件加载，模块名首字母小写并加 `.cfg`）。这是官方指定的模块化接口风格。

## 二、行为层与 CABSL（options）

- **从 2024 年变动**：高层行为现在以 **option（选项）** 的方式组织与调用。

## 三、“点球”在框架中的位置（如何被触发）

- 行为（例如点球）通常是 `SkillBehaviorControl` 下的一个 option。场景（`Config/Scenarios/<scenario>/threads.cfg`）控制哪些提供者（providers，哪一模块提供某个 representation）在运行时被启用。切换 `scenario`（或 deploy 时传 `-s`）会启/关不同的行为 provider，从而切换到点球模式。

---

# 在代码仓库中：哪些目录 / 文件是“点球”相关的**首要修改点**（官方文档里明确出现的路径）

1. **行为 / 高层（options/技能）**
   
   - `Src/Modules/BehaviorControl/` —— 行为控制相关模块的根目录（`SkillBehaviorControl` 会在这里）。
   
   - **（技能接口）** `Src/Representations/BehaviorControl/SkillInterfaces.h` —— 用来在行为层和 Motion 层之间传递技能请求的接口（文档在 Motion Framework 中列出此文件名作为需修改/扩展的地方）。

2. **配置 / 场景（控制哪些模块在哪种情形下启用）**
   
   - `Config/Scenarios/<scenario>/threads.cfg` —— 场景线程/提供者配置（决定某个 representation 由哪个模块在该场景下提供）；切换点球/常规模式通常受场景配置影响。
   
   - `Config/settings.cfg` —— 团队号、默认变量（deploy/创建机器人会用到）。

3. **Deploy / 工具脚本（在 dev→robot 的链路上关键）**
   
   - `Make/Common/deploy`、`Install/createRobot`、`Make/Common/login`（这些脚本由文档明确列出并给出调用示例）。

4. **感知 / 罚球点检测**（如果你需要更稳的点球定位）
   
   - 文档有专门的“Ball and Penalty Mark Detection”页面；感知改进直接影响点球定位与校准。

技术来源：

B-Human官网文档：[B-Human官网](https://docs.b-human.de/coderelease2024/)

B-Human在github上传的仓库：[github仓库](https://github.com/bhuman/BHumanCodeRelease)


