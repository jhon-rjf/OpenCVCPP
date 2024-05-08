# OpenCV

> C++에서 opencv 사용하기
> 


>💡 개발 환경: 
>OS : Windows 11 Home 23H2
>
>프로세서 : Intel(R) Core(TM) i7-10700 CPU @ 2.90GHz   2.90 GHz
>
>RAM : 32.0GB
>
>IDE:  Microsoft Visual Studio Community 2022 (64-bit) - Current버전 17.8.2
>

1. **opencv 다운로드**

[https://opencv.org/releases/](https://opencv.org/releases/)

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled.png)

가장 위의 버전을 다운로드 했습니다. 해당 하는 버전을 잘 기억해서 파일 경로 명에 착오 없도록 합니다.

2. **다운로드 된 파일 실행 → 경로를 C: 로 바꾸기**

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%201.png)

3. **visual studio 2022 프로젝트 설정**
- 프로젝트 생성

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%202.png)

만들기 클릭

- 프로젝트 → “프로젝트명” 속성
- C/C++ →일반 → 추가포함 디렉터리 (오른 쪽 빈공간 더블클릭)
- [추가포함디렉터리] 빈 박스 오른쪽 … 클릭
- [추가포함 디렉터리] C:\opencv\build\include 포함시키기

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%203.png)

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%204.png)

---

> 링커 → 일반 → 추가 라이브러리 디렉터리 → 빌드환경에 맞는 lib폴더 경로 추가
> 

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%205.png)

---

> 링커 → 입력 → 추가 종속성 → opencv_world490d.lib 추가
> 

> C:\opencv\build\x64\vc16\lib\opencv_world490d.lib 파일을 프로젝트가 있는 폴더에 붙여넣기 (ex) C:\Users\user\source\repos\PJ_name\PJ_name
> 

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%206.png)

4. **테스트**

테스트코드

```c
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    cv::Mat image = cv::imread("example.jpg"); // 테스트 이미지 파일
    if (image.empty()) {
        std::cout << "이미지를 불러올 수 없습니다." << std::endl;
        return -1;
    }
    cv::imshow("Image", image);
    cv::waitKey(0);
    return 0;
}
```

→ 실행시 사진이 없다고 뜬다

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%207.png)

---

이럴땐 원하는 사진을 해당 위치에 넣어주면 된다C:\Users\user\source\repos\PJ_name\PJ_name 
example.jpg라고 이름을 바꿔서 넣으면 성공

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%208.png)

---

### **edge.cpp 파일 실행하기**

- 명령 인수 설정
    - 프로젝트 → 속성 → 디버깅 → 명령인수 → example.jpg 추가
    
    ![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%209.png)
    
- edge.cpp

```c
#include "opencv2/core/utility.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"

#include <stdio.h>

using namespace cv;
using namespace std;

int edgeThresh = 1;
int edgeThreshScharr=1;

Mat image, gray, blurImage, edge1, edge2, cedge;

const char* window_name1 = "Edge map : Canny default (Sobel gradient)";
const char* window_name2 = "Edge map : Canny with custom gradient (Scharr)";

// define a trackbar callback
static void onTrackbar(int, void*)
{
    blur(gray, blurImage, Size(3,3));

    // Run the edge detector on grayscale
    Canny(blurImage, edge1, edgeThresh, edgeThresh*3, 3);
    cedge = Scalar::all(0);

    image.copyTo(cedge, edge1);
    imshow(window_name1, cedge);

    /// Canny detector with scharr
    Mat dx,dy;
    Scharr(blurImage,dx,CV_16S,1,0);
    Scharr(blurImage,dy,CV_16S,0,1);
    Canny( dx,dy, edge2, edgeThreshScharr, edgeThreshScharr*3 );
    /// Using Canny's output as a mask, we display our result
    cedge = Scalar::all(0);
    image.copyTo(cedge, edge2);
    imshow(window_name2, cedge);
}

static void help(const char** argv)
{
    printf("\nThis sample demonstrates Canny edge detection\n"
           "Call:\n"
           "    %s [image_name -- Default is fruits.jpg]\n\n", argv[0]);
}

const char* keys =
{
    "{help h||}{@image |fruits.jpg|input image name}"
};

int main( int argc, const char** argv )
{
    help(argv);
    CommandLineParser parser(argc, argv, keys);
    string filename = parser.get<string>(0);

    image = imread(samples::findFile(filename), IMREAD_COLOR);
    if(image.empty())
    {
        printf("Cannot read image file: %s\n", filename.c_str());
        help(argv);
        return -1;
    }
    cedge.create(image.size(), image.type());
    cvtColor(image, gray, COLOR_BGR2GRAY);

    // Create a window
    namedWindow(window_name1, 1);
    namedWindow(window_name2, 1);

    // create a toolbar
    createTrackbar("Canny threshold default", window_name1, &edgeThresh, 100, onTrackbar);
    createTrackbar("Canny threshold Scharr", window_name2, &edgeThreshScharr, 400, onTrackbar);

    // Show the image
    onTrackbar(0, 0);

    // Wait for a key stroke; the same function arranges events processing
    waitKey(0);

    return 0;
}

```

실행결과

![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled%2010.png)


---

### **demhist.cpp 파일 실행하기**
demhist.cpp
```C++
#include "opencv2/core/utility.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include "opencv2/highgui.hpp"

#include <iostream>

using namespace cv;
using namespace std;

int _brightness = 100;
int _contrast = 100;

Mat image;

/* brightness/contrast callback function */
static void updateBrightnessContrast(int /*arg*/, void*)
{
    int histSize = 64;
    int brightness = _brightness - 100;
    int contrast = _contrast - 100;

    /*
     * The algorithm is by Werner D. Streidt
     * (http://visca.com/ffactory/archives/5-99/msg00021.html)
     */
    double a, b;
    if (contrast > 0)
    {
        double delta = 127. * contrast / 100;
        a = 255. / (255. - delta * 2);
        b = a * (brightness - delta);
    }
    else
    {
        double delta = -128. * contrast / 100;
        a = (256. - delta * 2) / 255.;
        b = a * brightness + delta;
    }

    Mat dst, hist;
    image.convertTo(dst, CV_8U, a, b);
    imshow("image", dst);

    calcHist(&dst, 1, 0, Mat(), hist, 1, &histSize, 0);
    Mat histImage = Mat::ones(200, 320, CV_8U) * 255;

    normalize(hist, hist, 0, histImage.rows, NORM_MINMAX, CV_32F);

    histImage = Scalar::all(255);
    int binW = cvRound((double)histImage.cols / histSize);

    for (int i = 0; i < histSize; i++)
        rectangle(histImage, Point(i * binW, histImage.rows),
            Point((i + 1) * binW, histImage.rows - cvRound(hist.at<float>(i))),
            Scalar::all(0), -1, 8, 0);
    imshow("histogram", histImage);
}

const char* keys =
{
    "{help h||}{@image|baboon.jpg|input image file}"
};

int main(int argc, const char** argv)
{
    CommandLineParser parser(argc, argv, keys);
    parser.about("\nThis program demonstrates the use of calcHist() -- histogram creation.\n");
    if (parser.has("help"))
    {
        parser.printMessage();
        return 0;
    }
    string inputImage = parser.get<string>(0);

    // Load the source image. HighGUI use.
    image = imread(samples::findFile(inputImage), IMREAD_GRAYSCALE);
    if (image.empty())
    {
        std::cerr << "Cannot read image file: " << inputImage << std::endl;
        return -1;
    }

    namedWindow("image", 0);
    namedWindow("histogram", 0);

    createTrackbar("brightness", "image", &_brightness, 200, updateBrightnessContrast);
    createTrackbar("contrast", "image", &_contrast, 200, updateBrightnessContrast);

    updateBrightnessContrast(0, 0);
    waitKey();

    return 0;
}

```

실행결과
![Untitled](OpenCV%20e0f1bb11a5044e739b183c7a61be5d8d/Untitled-3.png)
