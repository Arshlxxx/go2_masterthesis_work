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
3.  **低级初始化（关键）：** `go2_lowcmd_init` 节点必须在确保**前三个节点（获取步态观察值、导航控制、步态控制）都有数据输出**，并且**机器人处于趴着（低姿态或待机）状态**时才能启动。窗口中按下回车是导航点的切换

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

# 🦾 ARX-X5 机械臂操作与 ROS 控制指南

本文档汇集了 ARX-X5 机械臂的 CAN 总线配置、ROS 控制节点启动以及相关的 Python 测试脚本命令。

## 1. ⚙️ CAN 总线检查与初始化

在开始任何 ROS 或 Python 控制之前，需要确保 CAN 通信接口正常工作。

| 步骤 | 命令 | 描述 |
| :--- | :--- | :--- |
| **进入 CAN 目录** | `cd ARX_CAN/arx_can/` | 切换到 CAN 驱动和测试脚本所在的目录。 |
| **CAN 口配置** | `./ "tap" 1 "tap"` | 执行特定的 CAN 口初始化配置。 |
| **CAN 状态检查** | `src/ARX_X5/ARX_CAN/arx_can/arx_can1.sh` | **（可选）** 运行此脚本检查 `can1` 接口的工作状态是否正常。 |

> **注意：** 所有操作中默认使用的都是 **`can1`** 接口。

## 2. 🐍 Python 脚本运行

ARX-X5 提供了一组 Python 脚本用于快速测试和功能验证。每次运行 Python 脚本前，都需要加载环境变量。

| 步骤 | 命令 | 描述 |
| :--- | :--- | :--- |
| **加载环境** | `cd src/ARX_X5/py/arx_x5_python` | 切换到 Python 脚本目录。 |
| **设置环境** | `source ./setup.sh` | **（每次运行前都需要执行）** 加载 Python 相关的环境变量。 |
| **键盘控制 Demo** | `python3 test_keyboard.py` | 运行 Python 版本的键盘控制演示。 |
| **单臂控制 Demo** | `python3 test_single_arm.py` | 运行 Python 版本的单臂运动控制演示。 |

## 3. ⌨️ ROS 键盘控制启动

ROS 控制主要通过启动一个用于接收指令的 Launch 文件和一个发布指令的节点来实现。

| 步骤 | 命令 | 描述 |
| :--- | :--- | :--- |
| **进入 ROS 目录** | `cd .. && cd .. && cd ROS/X5_ws/` | 切换到 ROS 工作空间目录。 |
| **启动控制接收** | `source devel/setup.sh && roslaunch arx_x5_controller open_keyboard_control.launch` | 启动 ROS 控制器的接受端 Launch 文件。 |
| **启动键盘发布** | `source devel/setup.sh && rosrun arx_x5_controller KeyBoard` | 启动 ROS 节点，将键盘输入发布为控制指令。 |

## 4. 👁️ 视觉检测与 ROS 运动控制

这些命令常用于手眼标定后的抓取任务，通常需要多个终端同时开启。

| 任务 | 命令 | 描述 |
| :--- | :--- | :--- |
| **开启视觉检测** | `source devel/setup.sh && rosrun calibration_info detection.py` | 启动视觉检测节点，通常用于物体识别和定位。**（如果使用 Todesk 远程开相机，请确保相机已连接并配置。）** |
| **运动控制 Demo** | `source devel/setup.sh && rosrun arx_x5_controller single_arm_test` | ROS 版本的单臂运动控制 Demo。常与 `detection.py` 配合进行抓取测试。 |
| **测试节点** | `source devel/setup.sh && rosrun arx_x5_controller qytest` | 运行特定的测试节点。 |

> **注意：** 在执行抓取任务时，通常是先开 `open_keyboard_control.launch`，然后开视觉检测 (`detection.py`)，最后开运动控制 Demo (`single_arm_test`)。

## 5. 🛑 结束操作

| 功能 | 命令 | 描述 |
| :--- | :--- | :--- |
| **终止节点** | `Ctrl + c` | **每次运行完成后都需要执行**，用于安全终止当前终端中的 ROS 节点或脚本。 |

## 6. 📐 手眼标定结果

以下是相机（Cam）与夹爪（Gripper）之间的齐次变换矩阵 $T$。

### $T_{\text{cam}\to\text{gripper}}$ (手到眼的矩阵)

将相机坐标系下的点转换到夹爪坐标系下：
$$
T_{\text{cam}\to\text{gripper}} = 
\begin{pmatrix}
-0.04634663 & -0.47539334 & 0.87855174 & 0.07414396 \\
-0.99799694 & 0.05994736 & -0.02020965 & 0.05281745 \\
-0.04305933 & -0.87772859 & -0.47721946 & 0.09238157 \\
0.0 & 0.0 & 0.0 & 1.0
\end{pmatrix}
$$

### $T_{\text{gripper}\to\text{cam}}$ (相机坐标 $\to$ 夹爪坐标)

将夹爪坐标系下的点转换到相机坐标系下：
$$
T_{\text{gripper}\to\text{cam}} = 
\begin{pmatrix}
-0.04634663 & -0.99799693 & -0.04305932 & 0.06012586 \\
-0.47539335 & 0.05994736 & -0.87772860 & 0.11316722 \\
0.87855174 & -0.02020964 & -0.47721946 & -0.01998560 \\
0.0 & 0.0 & 0.0 & 1.0
\end{pmatrix}
$$


| **查看里程计数据** | `rostopic echo /vins_estimator/odometry` | 实时查看 `/vins_estimator/odometry` 话题的输出数据，包括机器人的**线速度**和**位置**信息。 |
