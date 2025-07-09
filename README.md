
# UR5 ROS 2 Humble 仿真 + MoveIt2 快速上手指南

> **目标**：在 **Ubuntu 22.04 + ROS 2 Humble** 环境一次性跑通 Gazebo 仿真、ros2\_control、MoveIt 2 与 RViz。全部使用官方仓库提供的 launch 文件，无需手写代码。

---

## 目录

1. [系统准备](#系统准备)
2. [安装 ROS 2 与 Gazebo](#安装-ros2-与-gazebo)
3. [创建工作区并获取源码](#创建工作区并获取源码)
4. [安装依赖并编译](#安装依赖并编译)
5. [运行 Gazebo + MoveIt + RViz](#运行-gazebo--moveit--rviz)
6. [常见扩展](#常见扩展)
7. [关闭与重启](#关闭与重启)

---

## 系统准备

| 检查项           | 命令                | 期望输出             |                          |
| ------------- | ----------------- | ---------------- | ------------------------ |
| **Ubuntu 版本** | `lsb_release -ds` | `Ubuntu 22.04.*` |                          |
| **ROS 发行版**   | `rosversion -d`   | `humble`         |                          |
| **显卡状况(可选)**  | \`glxinfo         | grep -i vendor\` | 能输出 OpenGL 信息；若无显卡可用软件渲染 |

---

## 安装 ROS2 与 Gazebo(强烈建议使用鱼香ros2--- fishros，视频参考古月居21讲)

```bash
# 1. 添加 ROS 2 APT 源
sudo apt update && sudo apt install -y curl gnupg lsb-release
curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | \
  sudo gpg -o /usr/share/keyrings/ros-archive-keyring.gpg --dearmor

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list

# 2. 安装桌面完整版 + Gazebo 插件
sudo apt update
sudo apt install -y \
  ros-humble-desktop \
  ros-humble-gazebo-ros-pkgs \
  ros-humble-gazebo-ros2-control \
  python3-colcon-common-extensions

# 3. 初始化 rosdep（只第一次）（鱼香提供了国内版本 rosdepc 下载视频参考古月居）
sudo rosdep init
rosdep update
```

> **作用**：安装 RViz、MoveIt2、Gazebo11、ros2\_control 插件及 colcon 编译工具。

---

## 创建工作区并获取源码

```bash
# 4. 创建工作区
mkdir -p ~/dev_ws/src && cd ~/dev_ws/src

# 5. 克隆官方仿真包
git clone -b humble \
  https://github.com/UniversalRobots/Universal_Robots_ROS2_Gazebo_Simulation.git \
  ur_simulation_gazebo

# 6. 克隆官方驱动 & MoveIt 配置
git clone -b humble \
  https://github.com/UniversalRobots/Universal_Robots_ROS2_Driver.git \
  ur_robot_driver

# 7.（可选）若需修改 URDF
# git clone -b humble https://github.com/UniversalRobots/Universal_Robots_ROS2_Description.git ur_description
```

---

## 安装依赖并编译

```bash
cd ~/dev_ws
# 8. 安装系统依赖（跳过 warehouse驱动）
rosdep install --ignore-src --from-paths src -y \
  --skip-keys "libmongocxx-dev libbsoncxx-dev mongodb-mongocxx-driver-dev"

# 9. 编译源码并刷新环境
colcon build --symlink-install --cmake-clean-cache
source install/setup.bash
```

> **说明**：若确实需要 MoveIt Warehouse 功能，请先安装 Mongo C/C++ 驱动后再去掉 `--skip-keys` 重新编译。warehouse 可以跳过 如需下载请使用git clone 源码仓库 apt下载无法找到mongocxx等库 universe库同样

---

## 运行 Gazebo + MoveIt + RViz

### 终端 1 — Gazebo 仿真

```bash
export LIBGL_ALWAYS_SOFTWARE=1   # 无独显或远程时可加
ros2 launch ur_simulation_gazebo ur_sim_control.launch.py ur_type:=ur5
```

成功标志：终端出现

```
... joint_state_broadcaster successfully started.
... joint_trajectory_controller successfully started.
```

### 终端 2 — MoveIt 2 + RViz

```bash
ros2 launch ur_moveit_config ur_moveit.launch.py \
     ur_type:=ur5 sim:=true launch_rviz:=true
```

* 拖动 RViz 交互标并点击 *Plan & Execute* → Gazebo 中 UR5 按轨迹运动。

---

## 常见扩展

| 需求              | 命令/参数                                            |
| --------------- | ------------------------------------------------ |
| 更换机器人型号         | `ur_type:=ur10e`、`ur_type:=ur3e` 等               |
| 仅启动仿真（无 MoveIt） | 只运行终端 1                                          |
| 仅启动 MoveIt（实机）  | `sim:=false use_fake_hardware:=false`，并连接真实控制器   |
| 无 GUI 渲染        | `GAZEBO_HEADLESS_RENDERING=1` 或 `headless:=true` |

---

## 关闭与重启

```bash
# 优雅关闭：各终端 Ctrl-C
# or 强制：
pkill -9 gzserver gzclient rviz2
pkill -f ros2        # 杀 ros2 launch/run
```

重新启动只需重复 **终端 1 → 终端 2** 两步。

---

> **至此，你已拥有一套完整的 UR5 + Gazebo + MoveIt2 开发环境！**
>
> * 如要修改机器人模型，可编辑 `ur_description` 后 `colcon build`。
> * 如要保存/加载规划场景，安装 Mongo 驱动并编译 `warehouse_ros_mongo`。
>
> 祝你开发顺利 🚀
