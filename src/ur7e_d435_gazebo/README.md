# UR7e + D435 外侧眼在手上 Gazebo 场景

ROS 2 Humble / Gazebo Classic 11 功能包。直接启动 UR7e、D435 外形和传感器、外侧支架、
两面墙和一个球。沿用上一版初始关节姿态、模型名、话题、相机位置和控制接口。

## 构建与启动

```bash
cd /home/wqj/realtime_nvblox_ws
./build_ur7e_d435_scene.sh
./run_ur7e_d435_scene.sh
```

可追加 `gazebo_gui:=false`。先关闭其他使用同一 ROS 域和 Gazebo 端口的仿真实例。
两个脚本只加载系统 ROS 和当前工作空间，不需要 source 旧机械臂、相机、STORM 工作空间，
也不需要 `source_realtime_nvblox.sh` 的 Torch/cuVSLAM 环境。

标准 ROS 入口（新终端）：

```bash
source /opt/ros/humble/setup.bash
source /home/wqj/realtime_nvblox_ws/install/local_setup.bash
ros2 launch ur7e_d435_gazebo scene.launch.py
```

## 依赖

功能包内包含相机插件源码、当前 UR7e 模型、所有引用网格、相机内部描述、外参、支架、
障碍物、控制器配置和生成脚本。运行路径通过 ament 包索引查找，不引用旧工作空间文件。
第三方来源和许可证见 `THIRD_PARTY.md` 与 `licenses/`。

ROS/Gazebo 系统依赖完整声明在 `package.xml`；新机器安装依赖可运行：

```bash
cd /home/wqj/realtime_nvblox_ws
rosdep install --from-paths src/ur7e_d435_gazebo --ignore-src --rosdistro humble -r -y
```

构建需要系统 C++ 编译器、CMake 和 colcon。相机渲染需要可用的图形环境；关闭 GUI 仍会渲染深度。
本包不会启动 nvblox 或 MPPI，不会修改既有 realtime_nvblox 包。

## 配置与输出

- `config/ur7e_d435_side_sim_mount.yaml`：直接 `tool0 → camera_color_optical_frame`，
  平移 `[-0.0325, -0.075, 0.0357]` m，旋转 xyzw `[0, 0, 0, 1]`。这是仿真设计值。
- `urdf/d435_side_mount.urdf.xacro`：外侧支架，中央工具轴留空。
- `config/initial_positions.yaml`：六关节初始位置。
- `config/collision_world_gazebo_tall.yml`：两面墙和球；生成时沿用 `sage_clean_*` 模型名。
- `config/controllers.yaml`：100 Hz 的 controller_manager、关节状态和位置控制器。
- `/camera/color/image_raw`、`/camera/depth/image_rect_raw`、`/camera/depth/color/points`：1280×720。
- `/joint_states`、`/forward_position_controller/commands`：保留原控制接口。

修改安装配置后重启即可（symlink-install）。不保证任意机械臂动作都无自碰撞；
原 MPPI 碰撞球尚未包含新增相机和支架。深度未对齐到 RGB，未模拟 IMU。

## 启动验证

仿真运行后，在加载系统 ROS 和本工作空间的终端执行：

```bash
ros2 run ur7e_d435_gazebo verify_scene.py --output /tmp/ur7e_d435_scene_check.json
```

检查实际图像/点云的连续时间戳、TF、六关节初始状态、两个 active 控制器、三个障碍物和机械臂，
以及所有网格引用都位于当前功能包。该检查应在启动 MPPI 或发出关节运动命令之前运行。

2026-09-15 已在当前工作空间完成实际构建和 GUI 启动，上述检查通过。
Gazebo 加载的是本包构建的相机插件；新旧完整展开 URDF 在归一化资源路径后相同，
25 个 link、24 个 joint 和 15 处网格引用一致，网格内容 SHA256 一致。
初始姿态、相机外参和障碍物配置也与原场景逐字节一致。
本次运行证据保存在工作空间 `log/`：

- `ur7e_d435_scene_validation.json`：实际模型、控制器、TF、相机消息与关节状态。
- `ur7e_d435_scene_equivalence.json`：新旧模型一致性检查结果。
- `ur7e_d435_package_scene.png`：Gazebo 界面截图。
