# RoboMaster 装甲板检测系统（以红色为例）

这是一个基于 OpenCV 的 C++ 程序，用于检测 RoboMaster 比赛中的装甲板目标。程序通过颜色阈值分割和形态学处理识别红色灯条，然后配对灯条形成装甲板，最后在图像中标记检测结果。

## 功能特点

1. **双通道红色检测**：同时检测 HSV 空间中两种红色范围（0-10° 和 160-180°）
2. **实时参数调整**：提供轨迹条界面动态调整颜色阈值
3. **灯条筛选**：基于面积、长宽比等特征筛选有效灯条
4. **装甲板配对**：根据灯条距离、角度差和高度差配对形成装甲板
5. **可视化调试**：显示中间处理步骤和最终结果

## 依赖项

- OpenCV 4.x (建议 4.5+)
- C++17 兼容编译器

## 构建与运行

### Linux 环境

```bash
# 安装依赖
sudo apt install build-essential cmake libopencv-dev

# 克隆仓库
git clone https://github.com/yourusername/robo-detection.git
cd robo-detection

# 创建构建目录
mkdir build && cd build

# 配置和构建
cmake ..
make -j4

# 运行程序
./armor_detector
```

### Windows 环境

1. 安装 Visual Studio 2019 或更高版本
2. 安装 vcpkg 并配置 OpenCV
3. 使用 CMake 生成解决方案并构建

## 使用说明

1. **修改图像路径**：
   在 `main()` 函数中修改图像路径：
   ```cpp
   string image_path = "/path/to/your/image.png";
   ```

2. **参数调整界面**：
   ![参数调整界面](docs/params_ui.png)
   - 使用轨迹条调整 HSV 阈值：
     - 低红H1/高红H1：第一红色范围 (0-30)
     - 低红H2/高红H2：第二红色范围 (0-180)
     - 低S/高S：饱和度范围 (0-255)
     - 低V/高V：亮度范围 (0-255)

3. **快捷键**：
   - `ESC`：退出程序
   - `s`：保存当前参数到控制台

4. **输出窗口**：
   - 红色掩膜1：第一红色范围检测结果
   - 红色掩膜2：第二红色范围检测结果
   - 合并掩膜：形态学处理后的最终掩膜
   - 最终结果：装甲板检测结果（绿色框为装甲板，红点为中心点）

## 代码结构

```
src/
├── main.cpp                  # 主程序入口
include/                      
├── armor_detector.h          # 装甲板检测类声明
docs/                         
├── params_ui.png             # 参数界面截图
CMakeLists.txt                # CMake 构建配置
README.md                     # 项目说明
```

关键函数说明：
- `processImage()`：核心图像处理流程
- `onTrackbar()`：轨迹条回调函数
- `LightBar` 结构体：存储灯条信息
- 装甲板配对逻辑：基于距离、角度差和高度差

## 参数调整建议

1. **红色范围**：
   - 典型值：H1(0-10), H2(160-180)
   - 根据实际灯光条件调整边界值

2. **饱和度(S)**：
   - 建议范围：100-255
   - 调高可减少浅色干扰

3. **亮度(V)**：
   - 建议范围：50-255
   - 调高可避免暗区误检

4. **形态学操作**：
   - 内核大小 (7x7) 可根据图像分辨率调整
   - 分辨率高时可增大内核
     
实践可行：（低红H1-25,高红H1-27，低红H2-22,高红H2-180，低S-0,高S-255,低V-63,高V-255）
（低蓝H-0,高蓝H-180,低S-0,高S-255,低V-39，高V-255）
## 性能优化

1. 对于视频流处理，可添加帧率计数器：
   ```cpp
   auto start = chrono::high_resolution_clock::now();
   // 处理代码
   auto end = chrono::high_resolution_clock::now();
   auto duration = chrono::duration_cast<chrono::milliseconds>(end - start);
   cout << "处理时间: " << duration.count() << "ms" << endl;
   ```

2. 开启 OpenCV 优化：
   ```cpp
   setUseOptimized(true);
   setNumThreads(4); // 根据CPU核心数设置
   ```

## 已知问题与改进方向

1. **当前限制**：
   - 仅处理静态图像
   - 未实现目标跟踪
   - 参数调整需手动进行

2. **改进方向**：
   ```mermaid
   graph LR
   A[当前版本] --> B[视频流处理]
   A --> C[自动参数校准]
   A --> D[目标跟踪]
   B --> E[实时性能优化]
   C --> F[机器学习辅助]
   D --> G[预测运动轨迹]
   ```

3. **待实现功能**：
   - 装甲板数字识别
   - 3D位置估计
   - 多目标跟踪
   - 自动曝光控制

## 贡献指南

欢迎提交 Issue 和 Pull Request：
1. 报告问题时请附上测试图像和环境信息
2. 新功能开发请创建独立分支
3. 遵循现有代码风格（Google C++ Style）

## 许可协议

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件。
