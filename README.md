# -#include <opencv2/opencv.hpp>
#include <iostream>
#include <vector>

using namespace cv;
using namespace std;

// 全局变量用于实时调整
int lowH1 = 0, highH1 = 10;
int lowH2 = 160, highH2 = 180;
int lowS = 100, highS = 255;
int lowV = 50, highV = 255;
bool parametersConfirmed = false;

// 图像处理函数
Mat processImage(Mat& img) {
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
    
    // 6. 筛选装甲板轮廓
    vector<RotatedRect> armorPlates;
    
    for (size_t i = 0; i < contours.size(); i++) {
        double area = contourArea(contours[i]);
        if (area < 100) continue;
        
        RotatedRect rect = minAreaRect(contours[i]);
        float width = rect.size.width;
        float height = rect.size.height;
        if (width < 1e-5 || height < 1e-5) continue;
        
        float aspectRatio = max(width, height) / min(width, height);
        if (aspectRatio > 1.2 && aspectRatio < 6.0) {
            armorPlates.push_back(rect);
        }
    }
    
    // 7. 绘制结果
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
    
    // 8. 显示处理过程
    if (!parametersConfirmed) {
        imshow("红色掩膜1", mask1);
        imshow("红色掩膜2", mask2);
        imshow("合并掩膜", mask);
    }
    
    // 9. 输出当前参数
    if (!parametersConfirmed) {
        cout << "当前红色范围1: H(" << lowH1 << "-" << highH1 << ") "
             << "S(" << lowS << "-" << highS << ") "
             << "V(" << lowV << "-" << highV << ")" << endl;
        cout << "当前红色范围2: H(" << lowH2 << "-" << highH2 << ") "
             << "S(" << lowS << "-" << highS << ") "
             << "V(" << lowV << "-" << highV << ")" << endl;
        cout << "检测到装甲板数量: " << armorPlates.size() << endl;
    }
    
    return result;
}

// 轨迹条回调函数
void onTrackbar(int, void* userData) {
    Mat* img = static_cast<Mat*>(userData);
    Mat result = processImage(*img);
    imshow("最终结果", result);
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
    Mat result = processImage(img);
    imshow("最终结果", result);
    
    // 添加确认按钮提示
    putText(result, "按 'c' 确认参数并保存结果", Point(10, 30), 
            FONT_HERSHEY_SIMPLEX, 0.7, Scalar(0, 0, 255), 2);
    putText(result, "按 ESC 退出程序", Point(10, 60), 
            FONT_HERSHEY_SIMPLEX, 0.7, Scalar(0, 0, 255), 2);
    imshow("最终结果", result);
    
    // 等待用户调整
    while (true) {
        int key = waitKey(30);
        if (key == 27) break; // ESC退出
        if (key == 's') {     // 保存当前参数
            cout << "\n===== 保存的参数 =====" << endl;
            cout << "低红H1: " << lowH1 << endl;
            cout << "高红H1: " << highH1 << endl;
            cout << "低红H2: " << lowH2 << endl;
            cout << "高红H2: " << highH2 << endl;
            cout << "低S: " << lowS << endl;
            cout << "高S: " << highS << endl;
            cout << "低V: " << lowV << endl;
            cout << "高V: " << highV << endl;
            cout << "======================" << endl;
        }
        if (key == 'c') {     // 确认参数
            parametersConfirmed = true;
            
            // 关闭调试窗口
            destroyWindow("红色掩膜1");
            destroyWindow("红色掩膜2");
            destroyWindow("合并掩膜");
            
            // 重新处理图像
            Mat finalResult = processImage(img);
            
            // 保存结果
            imwrite("armor_detection_result.png", finalResult);
            cout << "结果已保存为 armor_detection_result.png" << endl;
            
            // 显示最终结果
            putText(finalResult, "参数已确认 - 结果已保存", Point(10, 30), 
                    FONT_HERSHEY_SIMPLEX, 0.7, Scalar(0, 255, 0), 2);
            imshow("最终结果", finalResult);
        }
    }
    
    return 0;
}
自学将任务图片进行图像处理，处理并框出装甲板
