装甲板视觉检测系统
 
 
https://img.shields.io/badge/OpenCV-4.5%2B-brightgreen   https://img.shields.io/badge/Platform-Linux%20%7C%20Windows-blue  
 
 
 
项目概述
 
 
基于OpenCV的红色装甲板实时检测系统，通过HSV颜色空间分割、形态学处理和轮廓分析技术，精准识别战场环境中的装甲目标。支持参数实时调整，适用于RoboMaster等机器人竞赛场景 。
 
 
 
demo.gif 
(示例效果图，建议补充实际截图)
 
 
 
核心功能
 
 
 
双范围红色检测
 
 
自适应HSV阈值处理（处理0-10°和160-180°红色范围）
 
支持实时参数调整（8个滑轨条动态调参）
 
灯条智能识别
 
 
面积过滤（>100像素）
 
长宽比筛选（1.2-6.0）
 
角度差约束（<15°）
 
装甲板精准定位
 
 
灯条配对算法（距离/高度/角度多约束）
 
融合轮廓最小外接矩形计算
 
实时绘制装甲板边界框和中心点
 
调试可视化
 
 
分阶段显示掩膜处理结果
 
实时控制台参数反馈
 
快捷键支持（ESC退出/S保存参数）
 
 
 
安装指南
 
 
 
环境依赖
# Ubuntu
sudo apt install build-essential libopencv-dev

# Windows
vcpkg install opencv[core]
编译运行
git clone https://github.com/yourname/armor-detector.git
cd armor-detector
mkdir build && cd build
cmake ..
make -j4
./armor_detector
使用说明
 
 
 
1.图像路径配置
修改 main.cpp 中的绝对路径：
string image_path = "/your/path/here.png"; // 替换为实际路径
2.参数调整界面
params_ui.png 
实时调整HSV阈值范围，界面包含：
 
 
低红H1/高红H1：第一红色范围
 
低红H2/高红H2：第二红色范围
 
低S/高S：饱和度范围
 
低V/高V：明度范围
 
3.快捷键操作
按键	功能
ESC	退出程序
S	保存当前参数到终端
算法流程
graph TD
    A[图像输入] --> B[HSV空间转换]
    B --> C{双范围阈值处理}
    C --> D[掩膜合并]
    D --> E[形态学优化]
    E --> F[轮廓检测]
    F --> G[灯条筛选]
    G --> H[装甲板配对]
    H --> I[结果可视化]
    项目结构
    ├── CMakeLists.txt
├── include/
│   └── armor_detector.h
├── src/
│   ├── main.cpp          # 主程序入口
│   └── image_processor.cpp # 核心处理逻辑
├── assets/               # 测试图像
│   └── test_samples/
├── README.md             # 本文档
└── LICENSE               # MIT许可证
常见问题
 
 
Q: 无法加载图像
 
 
检查 image_path 路径权限，确保使用绝对路径
 
 
Q: 灯条检测不稳定
 
 
调整以下参数：
 
 
增大 lowS 减少环境光影响
 
降低 highV 避免过曝
 
修改形态学核大小（当前7×7）
 
 
 
Q: 装甲板误匹配
 
 
优化配对条件：
// 修改距离约束系数
distance > maxHeight * 0.8 // 原值0.5

 
未来扩展
 
 
 
  视频流实时处理
 
  深度学习辅助识别
 
  三维姿态估计
 
  ROS节点封装
 
 
 
许可证
 
 
本项目采用 MIT License  ，允许商业和非商业用途的自由使用 
