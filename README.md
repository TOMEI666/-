#include <opencv2/opencv.hpp>
#include <iostream>
#include <vector>
#include <cmath>

using namespace cv;
using namespace std;

// 全局变量用于实时调整
int lowH1 = 0, highH1 = 10;
int lowH2 = 160, highH2 = 180;
int lowS = 100, highS = 255;
int lowV = 50, highV = 255;

// 灯条结构体
struct LightBar {
    RotatedRect rect;
    vector<Point> contour;
};

// 图像处理函数
void processImage(Mat& img) {
    // 1. 颜色空间转换 (BGR转HSV)
    Mat hsv;
    cvtColor(img, hsv, COLOR_BGR2HSV);
    
    // 2. 设置红色阈值
    Scalar lower_red1(lowH1, lowS, lowV);
    Scalar upper_red1(highH1, highS, highV);
    Scalar lower_red2(lowH2, lowS, lowV);
    Scalar upper_red2(highH2, highS, highV);
    
    // 3. 创建颜色掩膜并合并
    Mat mask1, mask2, mask;
    inRange(hsv, lower_red1, upper_red1, mask1);
    inRange(hsv, lower_red2, upper_red2, mask2);
    bitwise_or(mask1, mask2, mask);
    
    // 4. 形态学操作
    Mat kernel = getStructuringElement(MORPH_RECT, Size(7, 7));
    morphologyEx(mask, mask, MORPH_CLOSE, kernel);  // 填充空洞
    morphologyEx(mask, mask, MORPH_OPEN, kernel);   // 去除噪点
    
    // 5. 轮廓检测
    vector<vector<Point>> contours;
    findContours(mask, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
    
    // 6. 筛选灯条
    vector<LightBar> lightBars;
    for (size_t i = 0; i < contours.size(); i++) {
        double area = contourArea(contours[i]);
        if (area < 100) continue;
        
        RotatedRect rect = minAreaRect(contours[i]);
        float width = rect.size.width;
        float height = rect.size.height;
        if (width < 1e-5 || height < 1e-5) continue;
        
        float aspectRatio = max(width, height) / min(width, height);
        if (aspectRatio > 1.2 && aspectRatio < 6.0) {
            LightBar lightBar;
            lightBar.rect = rect;
            lightBar.contour = contours[i];
            lightBars.push_back(lightBar);
        }
    }
    
    // 7. 配对灯条形成装甲板
    vector<RotatedRect> armorPlates;
    vector<bool> used(lightBars.size(), false);
    
    for (size_t i = 0; i < lightBars.size(); i++) {
        if (used[i]) continue;
        
        for (size_t j = i + 1; j < lightBars.size(); j++) {
            if (used[j]) continue;
            
            const RotatedRect& rect1 = lightBars[i].rect;
            const RotatedRect& rect2 = lightBars[j].rect;
            
            // 计算灯条中心距离
            float distance = norm(rect1.center - rect2.center);
            
            // 计算灯条角度差（考虑角度周期性）
            float angleDiff = abs(rect1.angle - rect2.angle);
            if (angleDiff > 90.0) angleDiff = 180.0 - angleDiff;
            
            // 计算高度差（y坐标差）
            float heightDiff = abs(rect1.center.y - rect2.center.y);
            
            // 配对条件（可根据实际情况调整）
            float maxHeight = max(rect1.size.height, rect2.size.height);
            if (distance > maxHeight * 0.5 && 
                distance < maxHeight * 5.0 &&
                angleDiff < 15.0 && 
                heightDiff < maxHeight * 0.5) {
                
                // 合并两个灯条的轮廓点
                vector<Point> mergedPoints = lightBars[i].contour;
                mergedPoints.insert(mergedPoints.end(), 
                                   lightBars[j].contour.begin(), 
                                   lightBars[j].contour.end());
                
                // 计算合并轮廓的最小外接矩形
                if (!mergedPoints.empty()) {
                    armorPlates.push_back(minAreaRect(mergedPoints));
                    used[i] = true;
                    used[j] = true;
                    break; // 找到配对后跳出内层循环
                }
            }
        }
    }
    
    // 8. 绘制结果 - 只绘制装甲板
    Mat result = img.clone();
    for (const auto& rect : armorPlates) {
        Point2f vertices[4];
        rect.points(vertices);
        for (int i = 0; i < 4; i++) {
            line(result, vertices[i], vertices[(i+1)%4], Scalar(0, 255, 0), 3);
        }
        
        // 显示装甲板中心
        circle(result, rect.center, 5, Scalar(0, 0, 255), -1);
    }
    
    // 9. 显示处理过程
    imshow("红色掩膜1", mask1);
    imshow("红色掩膜2", mask2);
    imshow("合并掩膜", mask);
    imshow("最终结果", result);
    
    // 10. 输出当前参数
    cout << "当前红色范围1: H(" << lowH1 << "-" << highH1 << ") "
         << "S(" << lowS << "-" << highS << ") "
         << "V(" << lowV << "-" << highV << ")" << endl;
    cout << "当前红色范围2: H(" << lowH2 << "-" << highH2 << ") "
         << "S(" << lowS << "-" << highS << ") "
         << "V(" << lowV << "-" << highV << ")" << endl;
    cout << "检测到装甲板数量: " << armorPlates.size() << endl;
}

// 轨迹条回调函数
void onTrackbar(int, void* userData) {
    Mat* img = static_cast<Mat*>(userData);
    processImage(*img);
}

int main() {
    // 1. 使用绝对路径读取图像
    string image_path = "/home/tmoei/C++Projects/gn001/1.png";
    Mat img = imread(image_path);
    
    if (img.empty()) {
        cerr << "错误：无法加载图像! 请检查路径: " << image_path << endl;
        return -1;
    }
    
    // 创建调试窗口
    namedWindow("参数调整", WINDOW_NORMAL);
    resizeWindow("参数调整", 600, 300);
    
    // 创建轨迹条
    createTrackbar("低红H1", "参数调整", &lowH1, 30, onTrackbar, &img);
    createTrackbar("高红H1", "参数调整", &highH1, 30, onTrackbar, &img);
    createTrackbar("低红H2", "参数调整", &lowH2, 180, onTrackbar, &img);
    createTrackbar("高红H2", "参数调整", &highH2, 180, onTrackbar, &img);
    createTrackbar("低S", "参数调整", &lowS, 255, onTrackbar, &img);
    createTrackbar("高S", "参数调整", &highS, 255, onTrackbar, &img);
    createTrackbar("低V", "参数调整", &lowV, 255, onTrackbar, &img);
    createTrackbar("高V", "参数调整", &highV, 255, onTrackbar, &img);
    
    // 初始处理
    processImage(img);
    
    // 等待用户调整
    while (true) {
        int key = waitKey(30);
        if (key == 27) break; // ESC退出
        if (key == 's') {     // 保存当前参数
            cout << "保存的参数:" << endl;
            cout << "低红H1: " << lowH1 << endl;
            cout << "高红H1: " << highH1 << endl;
            cout << "低红H2: " << lowH2 << endl;
            cout << "高红H2: " << highH2 << endl;
            cout << "低S: " << lowS << endl;
            cout << "高S: " << highS << endl;
            cout << "低V: " << lowV << endl;
            cout << "高V: " << highV << endl;
        }
    }
    
    return 0;
}
