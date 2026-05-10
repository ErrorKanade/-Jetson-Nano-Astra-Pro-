# -Jetson-Nano-Astra-Pro-
从零开始在 Jetson Nano 上配置 Astra Pro 相机驱动的“踩坑”全过程

本次学习中遇到的核心问题
1. 核心矛盾：OpenCV 版本冲突
现象：编译时出现大量 undefined reference（未定义引用）或版本不匹配错误。

本质：Jetson Nano 系统预装了 OpenCV 4.1，但 ROS Melodic 的核心组件（如 cv_bridge）依赖的是 OpenCV 3.2。

教训：在嵌入式环境中，多种库版本并存是常态。不能简单地卸载（会导致系统崩溃），必须学会通过 CMakeLists.txt 强制指定库路径。

2. 环境感知：路径与变量
现象：roslaunch 报错找不到包，或 setup.bash 报错找不到文件。

本质：ROS 是基于环境变量运行的。新开的每一个终端都是“白纸”，必须手动或自动（.bashrc）加载工作空间的路径。

3. 图形界面：显示转发障碍
现象：gedit 或 rviz 提示 cannot open display。

本质：SSH 远程连接默认不带图形回传。在嵌入式开发中，应习惯使用 nano 等命令行编辑器。

以下为Astra Pro 环境配置 SOP
本标准作业程序适用于 Jetson Nano (Ubuntu 18.04 + ROS Melodic) 环境。

第一步：基础依赖环境检查
在安装驱动前，必须确保系统拥有 OpenCV 3.2 的核心库。

检查库文件：
ls /usr/lib/aarch64-linux-gnu/libopencv_core.so.3.2

修复软链接（若缺失）：

Bash
sudo ln -s /usr/lib/aarch64-linux-gnu/libopencv_flann.so.3.2.0 /usr/lib/aarch64-linux-gnu/libopencv_flann.so.3.2
sudo ldconfig
第二步：源码获取与预编译配置
下载源码：

Bash
cd ~/ros_ws/src
git clone https://github.com/orbbec/ros_astra_camera.git
设置 USB 权限（至关重要）：

Bash
cd ~/ros_ws/src/ros_astra_camera
sudo ./scripts/install_init_rules.sh
# 拔插一次相机 USB
第三步：强制版本链接（核心解决手段）
若编译报错，需修改 ros_astra_camera/CMakeLists.txt。在文件末尾手动指定 OpenCV 3.2 路径，防止链接到 4.1 版本：

CMake
# 强制指定 3.2 库路径
set(FORCE_OPENCV_LIBS

  "/usr/lib/aarch64-linux-gnu/libopencv_core.so.3.2"
  
  "/usr/lib/aarch64-linux-gnu/libopencv_imgproc.so.3.2"
  
  "/usr/lib/aarch64-linux-gnu/libopencv_imgcodecs.so.3.2"
  
)
target_link_libraries(${PROJECT_NAME}_node ${FORCE_OPENCV_LIBS} ${catkin_LIBRARIES})

第四步：编译与环境刷新
清理旧缓存：rm -rf ~/ros_ws/build ~/ros_ws/devel

执行编译：

Bash
cd ~/ros_ws
catkin_make -j2 -DPYTHON_EXECUTABLE=/usr/bin/python2.7
写入环境变量：
echo "source ~/ros_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc

第五步：运行与调试（三剑客顺序）
必须按此顺序开启三个终端：

终端 A：roscore （开启通信总机）

终端 B：roslaunch astra_camera astrapro.launch （开启相机硬件）

终端 C：rviz （图形化显示）

第六步：RViz 画面调校
Fixed Frame：改为 camera_link。

彩色图：Add -> Image -> Topic 选 /camera/rgb/image_raw。

点云图：Add -> PointCloud2 -> Topic 选 /camera/depth/points。

结语
学习路上真的很不容易，请你务必坚持学习
