# go2_masterthesis_work

# 🐾 GO2 机器人 ROS 任务及动捕系统连接指南

本文档汇集了控制 GO2 机器人（或相关仿真-实物任务）以及连接 VRPN 动捕系统的核心 ROS 启动和运行指令。

## 1. 🌐 动捕系统连接（VRPN Client）

该指令用于连接 VRPN（Virtual Reality Peripheral Network）服务器，通常用于接收如 OptiTrack 等动捕系统提供的外部位姿数据。

| 功能 | 命令 | 描述 |
| :--- | :--- | :--- |
| **连接 VRPN** | `roslaunch vrpn_client_ros sample.launch server:=192.168.31.214` | 启动 VRPN 客户端，连接到指定的服务器 IP 地址。请根据实际情况修改 `server:=` 后面的 IP。 |

## 2. 🤖 GO2 机器人 ROS 任务运行

以下命令用于启动 GO2 机器人的各种控制节点。在运行这些命令之前，**请确保您已在当前终端中执行了 `source devel/setup.sh`**。

1.  **启动 ROS Master：** 必须在单独的终端中运行 `roscore`。
2.  **多窗口并行启动：** 除了 VRPN 连接和模式切换外，**获取步态观察值、导航控制、步态控制、低级命令初始化**这四个核心任务节点需要分别在各自的终端窗口中启动。
3.  **低级初始化（关键）：** `go2_lowcmd_init` 节点必须在确保**前三个节点（获取步态观察值、导航控制、步态控制）都有数据输出**，并且**机器人处于趴着（低姿态或待机）状态**时才能启动。

| 任务类别 | 命令 | 描述 |
| :--- | :--- | :--- |
| **获取步态观察值** | `source devel/setup.sh && rosrun GO2_tasks go2_getgaitobs eth0` | 启动节点，从机器人获取步态相关的观测数据。**`eth0`** 指定了机器人通信的网络接口。 |
| **导航控制** | `source devel/setup.sh && rosrun go2_sim2real navigation_play.py` | 启动用于导航任务的 ROS 节点或脚本，通常用于仿真到实物（Sim2Real）的测试。 |
| **步态控制** | `source devel/setup.sh && rosrun go2_sim2real gait_play.py` | 启动用于步态控制和播放的 ROS 节点或脚本，通常用于 Sim2Real 的步态部署。 |
| **低级命令初始化** | `source devel/setup.sh && rosrun GO2_tasks go2_lowcmd_init eth0` | 初始化机器人的低级控制命令，为后续的低级控制模式做准备。 |
| **关节软化** | `source devel/setup.sh && rosrun GO2_tasks go2_soft eth0` | 启动节点，使机器人的关节处于软化（Soft）模式，方便手动操作或进行安全测试。 |

## 3. ⚙️ 机器人模式切换与状态查看

| 功能 | 命令 | 描述 |
| :--- | :--- | :--- |
| **模式切换** | `source devel/setup.sh && rosrun GO2_tasks go2_robot_state eth0` | 启动节点用于切换 GO2 机器人的控制模式，例如在**高级模式**（High-Level）和**低级模式**（Low-Level）之间切换。 |
| **查看里程计数据** | `rostopic echo /vins_estimator/odometry` | 实时查看 `/vins_estimator/odometry` 话题的输出数据，包括机器人的**线速度**和**位置**信息。 |
