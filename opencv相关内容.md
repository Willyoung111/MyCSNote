# make opencv相关内容

#### 滑动条

```c++
int createTrackbar(constsring& trackbarname, 	//轨迹条的名字
                   constsring& winname, 		//附加在的窗口的名字
                   int* value, 					//指向整形的指针，滑块的初识位置
                   int count, 					//最大值
                   TrackbarCallback onChange=0, //回调函数
                   void* userdata=0);			//用户返回回调函数的数据
//创建轨迹条
createTrackbar( "滑动条", "窗口1", &g_nAlphaValueSilder, 100, onChange);

//获取当前轨迹条的位置并返回
getTrackbarPos(conststring& trackbarname, conststring& winname);
```



#### 鼠标操作

```c++
void setMouseCallback(conststring& winname, 	//窗口名
                      MouseCallback onMouse, 	//回调函数
                      void* userdata=0);		//传递到回调函数的参数

```



## OpenCV core组件

### Mat(OpenCV如何存储和处理图像)

Mat是一个类，由两个部分组成

1. 矩阵头：包含有矩阵尺寸、存储方法、存储地址等信息
2. 一个指向存储所有像素值的矩阵的指针



如何在函数中传递图像？

​	首先，因为图像的处理往往计算量很大，占用资源也不小，所以不到万不得已一般都不会进行图像的复制。在OpenCV的思路中，**让每个传递的Mat对象有自己的信息头，但共享同一个矩阵。**

```c++
Mat A,C; //创建信息头部分
A = imread("1.jpg", CV_LOAD_IMAGE_COLOR);	//为矩阵开辟内存，并存入对应的数据
Mat B(A);	//使用拷贝构造函数来获得一个新的Mat变量
C = A;		//通过赋值运算符来获得一个新的Mat变量

```

​	以上的A,B,C三个Mat对象，他们最终都指向同一个也是唯一一个数据矩阵。**对其中任何一个的改变都会导致其他两个的改变**。他们只是拥有不同的矩阵头，但是指向同一个矩阵空间。

​	也可以创建只引用部分数据的信息头。

```c++
Mat D(A, Rect(10,10,100,100));	//矩形界定
Mat E = A(Range:all(), Range(1,3));	//行列界定
```

##### 矩阵空间的释放:

​	Mat对象中有一个计数机制，每当复制一次Mat对象的信息头的时候，都会使其引用次数加一，反之，每当一个头被释放的时候，这个计数就会被减一，当这个计数值为零时，就表示当前矩阵没有在任何地方被引用，矩阵空间就会被清理/



##### 矩阵空间的完全复制：

​	当然也会有偶尔需要完全复制矩阵空间的时候，可以用clone()和copyTo()来实现。

```c++
Mat F = A.clone();
Mat G;
A.copyTo(G);
```

​	这种完全复制的情况下， 对其中某个矩阵的改变就不会影响其他矩阵。



##### 像素值的存储方式

- 颜色空间

  灰度级

  RGB

  HSV色彩模型（Hue 色度，Saturation 饱和度，Value 纯度）

  HLS色彩模型（Hue 色度，Lightness 亮度，Saturation 饱和度）

- 数据类型

  char：最小，占一个字节或8位，无符号（0~255）或者有符号（-127~+127）



#### 显示创建Mat对象的七种方法

1. 使用Mat()构造函数

```c++
Mat M(2,2, CV_8UC3, Scalar(0,0,255));
//定义了一个二维三通道图像
//重定义类型中的
Mat(int rows, int cols, int type, const Scalar& s);

```

​	其中的type类型有如下的规则定义

​	CV_【The number of bits per item】【Signed or Unsigned】【Type prefix】C【The channel number】

​	CV_【位数】【带符号与否】【类型前缀】C【通道数】

2. 在C\C++中通过构造函数进行初始化

```c++
int sz[3] = {2,2,2};
Mat L(3,sz, CV_8UC, Scalar::all(0));
//如何创建一个超过二维的矩阵
Mat(int ndims, const int* sizes, int type, const Scalar& s);

//第一个参数代表维数
//第二个参数是一个指向数组的指针，数组包含每个维度的尺寸
```



3. 为已存在的IplImage指针创建信息头

   ```c++
   IplImage* img = cvLoadImage("1.jpg", 1);
   Mat mtx(img);	//转换IplImage*->Mat
   ```

   

4. 利用Create()函数

```c++
M.create(4,4,CV_8UC(2));
//这种方法不能用于矩阵的初始化，只能在改变尺寸时重新为矩阵数据开辟内存
```

5. Matlab的初始化方式

```c++
Mat E = Mat::ones(2,2, CV_32F);
//可以使用Matlab的zeros(), ones(), eyes()方法
```

6. 对小矩阵使用逗号分隔式初始化函数

```c++
Mat C = (Mat_<double>(3,3) << 0,-1,0,-1,5,-1,0,-1,0);
```

7. 为已存在的对象创建新信息头

使用clone()和copyTo()创建一个新的信息头

```C++
Mat RowClone = C.row(1).clone();
```



## core组件进阶

### 图像中的像素操作

#### 颜色空间缩减、LUT操作

颜色空间缩减：比如将空间域为256的灰度域压缩为26个值

LUT： Look Up Table，查找表操作，为了避免每次都对每个像素进行颜色空间缩减的计算操作，提前将每个可能的像素值进行计算，并存入表中，在使用时只要读取即可。

```c++
operationsOnArrays:LUT()<lut>
//用于批量进行图像元素查找，扫描和操作图像

void LUT(InputArray src, InputArray lut, OutputArray dst)
```

#### 计时函数

```C++
getTickCount();
//CPU自某个事件以来走过的时钟周期
getTickFrequency();
//CPU一秒钟所走过的时钟周期

//example
double time0 = static_cast<double>(getTickCount());
//do something here
time0 = ((double)getTickCount() - time0)/getTickFrequency();
```

#### 访问图像中像素的三种方法

- 指针访问
- 迭代器iterator
- 动态地址计算

##### 指针访问

```c++
void colorReduce(Mat& inputImage, Mat& outputImage, int div)
{
	//参数准备
	outputImage = inputImage.clone();  //拷贝实参到临时变量
	int rowNumber = outputImage.rows;  //行数
	int colNumber = outputImage.cols*outputImage.channels();  //列数 x 通道数=每一行元素的个数

															  //双重循环，遍历所有的像素值
	for (int i = 0; i < rowNumber; i++)  //行循环
	{
		uchar* data = outputImage.ptr<uchar>(i);  //获取第i行的首地址
		for (int j = 0; j < colNumber; j++)   //列循环
		{
			// ---------【开始处理每个像素】-------------     
			data[j] = data[j] / div*div + div / 2;
			// ----------【处理结束】---------------------
		}  //行处理结束
	}
}
```

##### 迭代器iterator

```c++
void colorReduce(Mat& inputImage, Mat& outputImage, int div)  
{  
	//参数准备
	outputImage = inputImage.clone();  //拷贝实参到临时变量
	//获取迭代器
	Mat_<Vec3b>::iterator it = outputImage.begin<Vec3b>();  //初始位置的迭代器
	Mat_<Vec3b>::iterator itend = outputImage.end<Vec3b>();  //终止位置的迭代器

	//存取彩色图像像素
	for(;it != itend;++it)  
	{  
		// ------------------------【开始处理每个像素】--------------------
		(*it)[0] = (*it)[0]/div*div + div/2;  
		(*it)[1] = (*it)[1]/div*div + div/2;  
		(*it)[2] = (*it)[2]/div*div + div/2;  
		// ------------------------【处理结束】----------------------------
	}  
}  
```

相比指针直接访问可能出现越界问题，迭代器是非常安全的。

##### 动态地址计算

```c++
void colorReduce(Mat& inputImage, Mat& outputImage, int div)  
{  
	//参数准备
	outputImage = inputImage.clone();  //拷贝实参到临时变量
	int rowNumber = outputImage.rows;  //行数
	int colNumber = outputImage.cols;  //列数

	//存取彩色图像像素
	for(int i = 0;i < rowNumber;i++)  
	{  
		for(int j = 0;j < colNumber;j++)  
		{  	
			// ------------------------【开始处理每个像素】--------------------
			outputImage.at<Vec3b>(i,j)[0] =  outputImage.at<Vec3b>(i,j)[0]/div*div + div/2;  //蓝色通道
			outputImage.at<Vec3b>(i,j)[1] =  outputImage.at<Vec3b>(i,j)[1]/div*div + div/2;  //绿色通道
			outputImage.at<Vec3b>(i,j)[2] =  outputImage.at<Vec3b>(i,j)[2]/div*div + div/2;  //红是通道
			// -------------------------【处理结束】----------------------------
		}  // 行处理结束     
	}  
}  
```

必须在编译期知道图像的数据类型

### ROI区域体香叠加&图像混合

#### ROI：感兴趣区域（region of interest）

定义ROI区域的两种方法：

- Rect:定义矩形左上角和矩形的长宽
- Range:指定感兴趣的行或列的范围

```c++
Mat imageROI;
imageROI = image(Rect(500,250,logo.cols,logo.rows));

imageROI = image(Range(250,250+logoImage.rows),Range(500,500+logoImage.cows));
```

#### 线性混合操作、计算数组加权和：addWeighted()

```c++
void addWeighted(InputArray src1,	//需要加权的第一个数组
                 double alpha,		//第一个数组的权重
                 InputArray src2, 	//第二个数组，要求和第一个数组有相同的尺寸和通道数
                 double beta, 		//第二个数组的权重
                 double gamma, 		//一个加到权重总和上的标量值
                 OutputArray dst, 	//输出的数组
                 int dtype = -1);	//阵列的可选深度，-1表示输入的两个数组的相同深度

dst = src1[I]*alpha + src2[I]*beta + gamma;
//当数组的深度为CV_32S时，内存溢出不能用这个公式
```

### 分离颜色通道、多通道图像混合

#### 通道分离：split()函数

```c++
void split(InputArray m, OutputArrayOfArrays mv);

	//【0】定义相关变量
	Mat srcImage;
	Mat logoImage;
	vector<Mat> channels;
	Mat  imageBlueChannel;

	//=================【蓝色通道部分】=================
	//	描述：多通道混合-蓝色分量部分
	//============================================

	// 【1】读入图片
	logoImage = imread("dota_logo.jpg", 0);
	srcImage = imread("dota_jugg.jpg");


	//【2】把一个3通道图像转换成3个单通道图像
	split(srcImage, channels);//分离色彩通道

	 //【3】将原图的蓝色通道引用返回给imageBlueChannel，注意是引用，相当于两者等价，修改其中一个另一个跟着变
	imageBlueChannel = channels.at(0);

```

将三通道Mat分离到Mat类型的向量中

#### 通道合并：merge()函数

```c++
void merge(const Mat* mv, size_tcount, OutputArray dst);
void merge(InputArrayOfArrays mv, OutputArray dst);
```

### 图像对比度、亮度

```mathematica
g(x) = a * f(x) + b
g(i,j) = a * f(i,j) + b
```

a:增益（对比度）

b:偏置（亮度）

```c++
g_dstImage.at<Vec3b>(y, x)[c] = 
    saturate_cast<uchar>
    ((g_nContrastValue*0.01)*(g_srcImage.at<Vec3b>(y, x)[c]) + g_nBrightValue);

//在opencv中，saturate_cast常用于溢出保护
//如上，在使用了saturate_cast<uchar>后，括号内的内容就被限制在了0~255之间
//大致原理如下
if(data<0)
    data = 0;
else if(data>255)
    data = 255;
```

### DFT：离散傅里叶变换

#### dft()函数详解

```c++
void dft(InputArray src, 		//输入矩阵，可以为实数或者虚数
         OutputArray dst, 		//输出矩阵，保存运算结果，尺寸和类型取决于标识符，即flags
         int flags=0, 			//转换的标识符，int,具体内容见表《dft标识符取值列表》，p153
         int nonzeroRows=0);	//

//DFT最优尺寸大小
int getOptimalDFTSize(int vecsize);		

//扩充图像边界
void copyMakeBorder(InputArray src, 
                    OutputArray src, 
                    int top, 
                    int bottom, 
                    int left, 
                    int right, 			//表示在四个方向上扩充多少像素
                    int borderType, 
                    const Scalar& value=Scalar());
    
//计算二维矢量的幅值
void magnitude(InputArray x, InputArray y, OutputArray magnitude);

//计算Log
void log(InputArray x, InputArray y);

//矩阵归一化
void normalize( InputArray src, 
               InputOutputArray dst, 
               double alpha = 1, 			//归一化后最大值
               double beta = 0, 			//归一化后最小值
               int norm_type = NORM_L2, 
               int dtype = -1, 
               InputArray mask = noArray()
              );
```

#### 离散傅里叶变换的示例

##### 一、载入原始图像

##### 二、将图像扩大到合适的尺寸

图像的尺寸是2、3、5的整数倍时计算速度最快（为什么？）

先获得最优尺寸大小，再扩充

```c++
//【2】将输入图像延扩到最佳的尺寸，边界用0补充
int m = getOptimalDFTSize(srcImage.rows);
int n = getOptimalDFTSize(srcImage.cols);
//将添加的像素初始化为0.
Mat padded;
copyMakeBorder(srcImage, padded, 0, m - srcImage.rows, 0, n - srcImage.cols, BORDER_CONSTANT, Scalar::all(0));
```

##### 三、为傅里叶变换的结果分配存储空间

```c++
//【3】为傅立叶变换的结果(实部和虚部)分配存储空间。
//将planes数组组合合并成一个多通道的数组complexI
Mat planes[] = { Mat_<float>(padded), Mat::zeros(padded.size(), CV_32F) };
Mat complexI;
merge(planes, 2, complexI);
```

##### 四、进行离散傅里叶变换

```c++
dft(complexI, complexI);
```

##### 五、复数转换为幅值

```c++
//【5】将复数转换为幅值，即=> log(1 + sqrt(Re(DFT(I))^2 + Im(DFT(I))^2))
split(complexI, planes); 
// 将多通道数组complexI分离成几个单通道数组，planes[0] = Re(DFT(I)), planes[1] = Im(DFT(I))
magnitude(planes[0], planes[1], planes[0]);// planes[0] = magnitude  
Mat magnitudeImage = planes[0];
```

##### 六、对数尺度缩放

用对数尺度来替换线性尺度

M1 = log(1+M)

```c++
//【6】进行对数尺度(logarithmic scale)缩放
magnitudeImage += Scalar::all(1);
log(magnitudeImage, magnitudeImage);//求自然对数
```

##### 七、剪切和重分布幅度图像限

即零频率点中心

```c++
//【7】剪切和重分布幅度图象限
//若有奇数行或奇数列，进行频谱裁剪      
magnitudeImage = magnitudeImage(Rect(0, 0, magnitudeImage.cols & -2, magnitudeImage.rows & -2));
//重新排列傅立叶图像中的象限，使得原点位于图像中心  
int cx = magnitudeImage.cols / 2;
int cy = magnitudeImage.rows / 2;
Mat q0(magnitudeImage, Rect(0, 0, cx, cy));   // ROI区域的左上
Mat q1(magnitudeImage, Rect(cx, 0, cx, cy));  // ROI区域的右上
Mat q2(magnitudeImage, Rect(0, cy, cx, cy));  // ROI区域的左下
Mat q3(magnitudeImage, Rect(cx, cy, cx, cy)); // ROI区域的右下
//交换象限（左上与右下进行交换）
Mat tmp;
q0.copyTo(tmp);
q3.copyTo(q0);
tmp.copyTo(q3);
//交换象限（右上与左下进行交换）
q1.copyTo(tmp);
q2.copyTo(q1);
tmp.copyTo(q2);
```

##### 八、归一化

```c++
//【8】归一化，用0到1之间的浮点值将矩阵变换为可视的图像格式
//此句代码的OpenCV2版为：
//normalize(magnitudeImage, magnitudeImage, 0, 1, CV_MINMAX); 
//此句代码的OpenCV3版为:
normalize(magnitudeImage, magnitudeImage, 0, 1, NORM_MINMAX);
```

##### 九、显示

#### 文件操作：XML和YAML

## imgproc组件

Image Process

## 图像处理

#### 线性滤波





# GMM算法详解（opencv3版）

## 测试代码

```c++
#include"opencv2/opencv.hpp"
#include"opencv2/core/core.hpp"
#include"opencv2/highgui/highgui.hpp"
#include"opencv2/imgproc/imgproc.hpp"
#include"opencv2/video/background_segm.hpp"
#include<iostream>
using namespace cv;
using namespace std;

int main()
{
	VideoCapture capture(0);
	Ptr<BackgroundSubtractorMOG2> bg_model = createBackgroundSubtractorMOG2();
	Mat image, fgimage, fgmask;

	while (1)
	{
		capture >> image;	//get new frame from camera
		if (!image.data)
		{
			cerr << "picture error!";
			return -1;
		}
		if (fgimage.empty())
			fgimage.create(image.size(), image.type());
		bg_model->apply(image, fgmask, -1);
		fgimage = Scalar::all(0);
		image.copyTo(fgimage, fgmask);
		Mat bgimage;
		bg_model->getBackgroundImage(bgimage);

		imshow("image", image);
		imshow("fgimage", fgimage);
		imshow("fgmask", fgmask);
		if (!bgimage.empty())
			imshow("bgimage", bgimage);

		waitKey(30);
	}
	return 0;
}
```

## 算法原理

### 单高斯背景模型

​	将图像中每一个像素点的颜色值看成一个随机过程，并假设其出现概率服从高斯分布。对每一个像素位置建立一个高斯模型，模型中保存其均值的方差。比如对于一个像素点$(x,y)$ ，可设其均值：$\mu(x,y)$ ，方差：$\sigma^2(x,y)$，标准差$\sigma(x,y)$。

​	同时由于视频序列的输入，模型参数会不断更新，对应不同时刻参数有不同的值，可将模型参数表示为三个变量$(x,y,t)$的函数。均值：$\mu(x,y,t)$ ，方差：$\sigma^2(x,y,t)$，标准差:$\sigma(x,y,t)$。

​	单高斯模型运动检测的基本过程：

​	1、 模型初始化

$$\begin{cases}\mu(x,y,0)=I(x,y,0)\\\sigma^2(x,y,0)=std\_init^2\\\sigma(x,y,0)=std\_init\end{cases}$$

​	其中$I(x,y,0)$表示视频序列中的第一张图像在$(x,y)$处的像素值，std_init为设置的初始常数。

​	2、更新参数并检测

​	每当读入一帧新的图象时，需要判断新图像中对应点像素是否在高斯模型描述的范围内，是则为背景（表示没有突变），否则为前景。假设前景检测的结果图是$output$

$$output(x,y,t)=\begin{cases}0,	|I(x,y,t)-\mu(x,y,t-1)|<\lambda\times\sigma(x,y,t-1)\\1,otherwise\end{cases}$$

​	其中$\lambda$是设置的常数，含义：判断新图片中对应点像素和对象模型中像素的均值的距离小于标准差的$\lambda$倍，则该点为背景，否则为前景。

​	模型的更新

$$\begin{cases}\mu(x,y,t=(1-\alpha)\times\mu(x,y,t-1)+\alpha\times\mu(x,y,t)\\\sigma^2(x,y,t)=(1-\alpha)\times\sigma^2(x,y,t-1)+\alpha\times[I(x,y,t)-\mu(x,y,t)]^2\\\sigma(x,y,t)=\sqrt{\sigma^2(x,y,t)}\end{cases}$$

​	其中$\alpha$表示更新率，此常数使模型在背景缓慢变化时具有一定的鲁棒性。

### 混合高斯模型

单高斯模型只能描述单一模式的背景，多模态形式下的背景检测极易出错。

> 多模态：模态（modality），在这里可以理解为背景可能呈现的多种状态。比如树叶的晃动，树叶在不同图像中会呈现几种固定的状态。

混合高斯模型的原理，就是对每个像素点建立多个高斯模型，试图将可能出现微小律动的像素点的几种状态都预估，那么当背景出现微小变化时，其仍然符合某个高斯模型，以此滤除非关注区域。

#### 1、像素模型的定义

每个像素都由多个单模型描述：$P(p)={[\omega_i(x,y,t),\mu_i(x,y,t),\sigma_i(x,y,t)^2]},i=1,2,...,K$。其中K就表示为每个像素点创立几个高斯模型，其值一般在3~5之间。$\omega_i(x,y,t)$表示每个模型的权重，满足：

$$\sum^K_{i=1}\omega_i(x,y,t)=1$$

三个参数（权，均值，方差）确定一个单模型。

#### 2、更新参数并进行前景检测

 Step1:

​	判断如果新读入的视频图像序列中的图片在$(x,y)$处的像素值对$i=1,2,...,K$

$$output(x,y,t)=\begin{cases}0,	|I(x,y,t)-\mu_i(x,y,t-1)|<\lambda\times\sigma_i(x,y,t-1)\\1,otherwise\end{cases}$$

如果判断为背景，进入step2,如果判断为前景，进入step3

Step2:

​	修正与新像素匹配的单模型的权值，权值增量为$dw=\alpha\times(1-\omega_i(x,y,t-1))$,则新的权值为

$$\omega_i(x,y,t)=\omega_i(x,y,t-1)+\alpha\times(1-\omega_i(x,y,t-1))$$

​	修正匹配模型的均值和方差，同高斯模型。

​	Step2完成后直接进入Step4

Step3:

​	如果新像素不如任何一个单模型匹配，则：

​	1、如果但钱单模型的数目已经到达允许的最大数目，则去除当前重要性（计算见后文）最小的单模型

​	2、增加一个新的单模型，新模型的权重为一个很小的值（比如0.001）,均值为新像素值，方差为给定的较大的值（比如20）。（这样可以做到背景模型的更新）

Step4:权重归一化

$$\omega_i(x,y,t)=\frac{\omega_i(x,y,t)}{\sum^K_{i=1}\omega_i(x,y,t)},(i=1,2,...,K)$$



#### 3、重要性排序和删减

​	根据背景模型的两个特点：

​	1、权重大：背景出现的频率高

​	2、方差小：像素值变化不大

​	则定义

$sort\_key = \frac{\omega_i(x,y,t)}{\sigma_i(x,y,t)}$

作为排序依据。

过程如下：

​	1、计算每个单模型的重要性值sort_key

​	2、对于各个单模型按照重要性的大小进行排序，重要性大的排在前面

​	3、若前N个单模型的权重满足

$\sum^K_{i=1}\omega_i(x,y,t)>T$，则仅用这N个单模型作为北京模型，删除其他模型，一般T=0.7



## His3536 sample_ive_gmm.c解析

### GMM主体

对某一模块的初始化有以下步骤

1、设置用作初始化的结构体的参数

2、调用去初始化函数，清空信息

3、将参数传入设置结构体

4、调用初始化函数初始化模块

```c++
/******************************************************************************
* function : show GMM sample
******************************************************************************/
HI_S32 SAMPLE_IVE_GMM(HI_BOOL bEncode, HI_BOOL bVo)
{
    //前面是一些视频处理基本参数的设定，以下逐条解释
    HI_S32 s32Ret = HI_SUCCESS, i;	////定义视频缓存池属性结构体实例
	
    VB_CONF_S stVbConf,stModVbConf;
	SIZE_S astSize[VPSS_CHN_NUM] = {{HD_WIDTH, HD_HEIGHT}, {QVGA_WIDTH, QVGA_HEIGHT}};
    PAYLOAD_TYPE_E enStreamType = PT_H264;
	
    VDEC_CHN_ATTR_S stVdecChnAttr;
    VdecThreadParam stVdecSendParam;
    pthread_t VdecThread;
	HI_U32 u32VdecChnCnt = 1;
	
    VPSS_CHN aVpssChn[VPSS_CHN_NUM] = {VPSS_CHN0, VPSS_CHN3};
    VPSS_GRP_ATTR_S stVpssGrpAttr;
	HI_U32 u32VpssGrpCnt = 1;
	
    VENC_CHN VeH264Chn = 0;
    SAMPLE_RC_E enRcMode = SAMPLE_RC_CBR;
    PIC_SIZE_E enPicSize = PIC_HD1080;
    VIDEO_NORM_E enVideoNorm = VIDEO_ENCODING_MODE_PAL;
	HI_U32 u32VencChnCnt = 1;
		
    VO_DEV   VoDev	 = SAMPLE_VO_DEV_DHD0;
	VO_CHN   VoChn   = 0;
    VO_LAYER VoLayer = SAMPLE_VO_LAYER_VHD0;
    VO_PUB_ATTR_S stVoPubAttr;
    VO_VIDEO_LAYER_ATTR_S stVoLayerAttr;
    SAMPLE_VO_MODE_E enSampleVoMode = VO_MODE_1MUX;

	SAMPLE_IVE_GMM_S stGmm;
	SAMPLE_IVE_GMM_THREAD_PARAM_S stGmmThreadParam;
    pthread_t IveThread;
	HI_CHAR *pchStreamName = "./data/input/ive_perimeter_scene_1080p_01.h264";
	
    /******************************************
     step  1: Init SYS and common VB
    ******************************************/
	SAMPLE_COMM_IVS_SysConf(&stVbConf, astSize, VPSS_CHN_NUM);
	s32Ret = SAMPLE_COMM_SYS_Init(&stVbConf);
	SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_SYS_VB, "init SYS and common VB failed!\n");
	
    /******************************************
     step 2: Init VDEC mod VB. 
    *****************************************/    	
    SAMPLE_COMM_VDEC_ModCommPoolConf(&stModVbConf, enStreamType, &astSize[0],
        u32VdecChnCnt);	
    s32Ret = SAMPLE_COMM_VDEC_InitModCommVb(&stModVbConf);
	SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_SYS_VB, "init VDEC mod VB failed!\n");
	
    /******************************************
     step  3: Init GMM
    ******************************************/
	s32Ret = SAMPLE_IVE_GMM_Init(&stGmm, astSize[1].u32Width,astSize[1].u32Height);
	SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_GMM_INIT, "SAMPLE_IVE_GMM_Init failed\n");
	
    /******************************************
     step 4:  start VDEC.
    *****************************************/
    SAMPLE_COMM_VDEC_ChnAttr(u32VdecChnCnt, &stVdecChnAttr, enStreamType, &astSize[0]);
	s32Ret =SAMPLE_COMM_VDEC_Start(u32VdecChnCnt, &stVdecChnAttr);
	SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret,END_VDEC_START, "start VDEC failed with!\n");

    /******************************************
     step 5:  start VPSS and bind to VDEC.
    *****************************************/
    SAMPLE_COMM_IVS_VpssGrpAttr(u32VpssGrpCnt, &stVpssGrpAttr, &astSize[0]);
    s32Ret = SAMPLE_COMM_IVS_VpssStart(u32VpssGrpCnt, astSize, aVpssChn, VPSS_CHN_NUM, &stVpssGrpAttr);
	SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_VPSS_START, "start VPSS failed!\n");
	
    for(i=0;i<u32VpssGrpCnt;i++)
    {	
        s32Ret = SAMPLE_COMM_VDEC_BindVpss(i,i);
		SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_VPSS_BIND_VDEC, "SAMPLE_COMM_VDEC_BindVpss failed!\n");
    }
	
    /************************************************
    step 6:  start VO
    *************************************************/
    if(bVo)
    {		
		stVoPubAttr.enIntfSync = VO_OUTPUT_1080P30;
		stVoPubAttr.enIntfType = VO_INTF_VGA; //VO_INTF_HDMI | VO_INTF_VGA;
    	stVoPubAttr.u32BgColor = 0x0000FF;
		
		stVoLayerAttr.bClusterMode = HI_FALSE;
		stVoLayerAttr.bDoubleFrame = HI_FALSE;
		stVoLayerAttr.enPixFormat = SAMPLE_PIXEL_FORMAT;
		
		s32Ret = SAMPLE_COMM_VO_GetWH(stVoPubAttr.enIntfSync, \
			&stVoLayerAttr.stDispRect.u32Width, &stVoLayerAttr.stDispRect.u32Height, &stVoLayerAttr.u32DispFrmRt);
		SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_VPSS_BIND_VDEC, "SAMPLE_COMM_VO_GetWH failed!\n");
		
		stVoLayerAttr.stDispRect.s32X = 0;
		stVoLayerAttr.stDispRect.s32Y = 0;
		stVoLayerAttr.stImageSize.u32Width = stVoLayerAttr.stDispRect.u32Width;
		stVoLayerAttr.stImageSize.u32Height = stVoLayerAttr.stDispRect.u32Height;

		s32Ret = SAMPLE_COMM_VO_StartDev(VoDev, &stVoPubAttr);
		SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_VO_START, "SAMPLE_COMM_VO_StartDev failed!\n");
		
		s32Ret = SAMPLE_COMM_VO_HdmiStart(stVoPubAttr.enIntfSync);
		SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_HDMI_START, "SAMPLE_COMM_VO_HdmiStart failed!\n");
		
		s32Ret = SAMPLE_COMM_VO_StartLayer(VoLayer, &stVoLayerAttr);
		SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_LAYER_START, "SAMPLE_COMM_VO_StartLayer failed!\n");
		
		s32Ret = SAMPLE_COMM_VO_StartChn(VoLayer, enSampleVoMode);
		SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_VO_CHN, "SAMPLE_COMM_VO_StartChn failed!\n");
		
    }

	/************************************************
    step 7:  start VENC
    *************************************************/
    if(bEncode)
    {
    	s32Ret = SAMPLE_COMM_VENC_Start(VeH264Chn, enStreamType,enVideoNorm,enPicSize,enRcMode);
		SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_VENC_START, "SAMPLE_COMM_VENC_Start(h264 Venc) failed!\n");
	}

    /******************************************
    step 8: VDEC start send stream. 
    ******************************************/
    SAMPLE_COMM_VDEC_ThreadParam(u32VdecChnCnt, &stVdecSendParam, &stVdecChnAttr, pchStreamName);
    SAMPLE_COMM_VDEC_StartSendStream(u32VdecChnCnt, &stVdecSendParam, &VdecThread);

    /******************************************
    step 9: stream VENC process -- get stream, then save it to file. 
    ******************************************/
    if(bEncode)
    {
    	s32Ret = SAMPLE_COMM_VENC_StartGetStream(u32VencChnCnt);
		SAMPLE_CHECK_EXPR_GOTO(HI_SUCCESS != s32Ret, END_VENC_GET_STREAM, "SAMPLE_COMM_VENC_StartGetStream failed!\n"); 
    }
	
    /******************************************
    step 10: GMM process -- get stream, process and send to VO or VENC. 
    ******************************************/
    SAMPLE_IVE_GMM_ThreadParam(&stGmmThreadParam,bEncode,bVo,u32VpssGrpCnt,VPSS_CHN_NUM,
    						  aVpssChn,VeH264Chn,VoLayer,VoChn, &stGmm);
    SAMPLE_IVE_GMM_ThreadProc(&stGmmThreadParam, &IveThread);
   
    /******************************************
    step 11: exit process
    ******************************************/
	SAMPLE_IVE_GMM_CmdCtrl(&stGmmThreadParam, IveThread);
	
END_VENC_GET_STREAM:
    if(bEncode)
    {
    	SAMPLE_COMM_VENC_StopGetStream();
    }
    SAMPLE_COMM_VDEC_StopSendStream(u32VdecChnCnt,&stVdecSendParam,&VdecThread);

	if(bEncode)
	{       
	END_VENC_START:
    	SAMPLE_COMM_VENC_Stop(VeH264Chn);
	}
	
	if(bVo)
	{
	END_VO_CHN: 
    	SAMPLE_COMM_VO_StopChn(VoLayer, enSampleVoMode);
   
	END_LAYER_START: 
    	SAMPLE_COMM_VO_StopLayer(VoLayer);
   
	END_HDMI_START: 
    	SAMPLE_COMM_VO_HdmiStop();
	
	END_VO_START:
    	SAMPLE_COMM_VO_StopDev(VoDev);
	}

END_VPSS_BIND_VDEC:
    for(i=0;i<u32VdecChnCnt;i++)
    {
        SAMPLE_COMM_VDEC_UnBindVpss(i,i);
    }
    
END_VPSS_START:
    SAMPLE_COMM_IVS_VpssStop(u32VpssGrpCnt,VPSS_MAX_CHN_NUM);	
    
END_VDEC_START:
    SAMPLE_COMM_VDEC_Stop(u32VdecChnCnt);
	
END_GMM_INIT:
	SAMPLE_IVE_GMM_Uninit(&stGmm);
	
END_SYS_VB:
    SAMPLE_COMM_SYS_Exit();
    
    return s32Ret;
}
```

#### 扩展：图像分辨率、显示分辨率、设备分辨率

图像分辨率：放置各个通道图像的画布大小

显示分辨率：图像分辨率中描述的画布经过VO放大后的显示区域

设备分辨率：与设备时序一致

![](D:\我的坚果云\note\视频层属性的相关概念.jpg)

### GMM进程步骤

1. 从文件中读取视频码流，分为1920x1080和320x240两种格式，分别通过通道0、1发送码流
2. 在GMM进程中分别读取为BigFrm和SmallFrm
3. 将gmm中的src设置为SmallFrm，并将源图像用DMA拷贝到dst
4. 对src进行5x5滤波，输出到img1
5. 对img1进行gmm操作，获得fg和bg
6. 对fg进行膨胀，输出到img1
7. 对img1进行腐蚀，输出到img2
8. 对img2进行8连通区域标记，获得连通区域信息blob
9. 根据BigFrm和SmallFrm的缩放比例，用blob进行矩形框选
10. 在BigFrm中添加矩形框





### 待解决的问题

1、初始10~20帧无法检测运动目标

2、运动物体颜色与背景相似时漏检

3、单个运动物体出现多个选框

4、缓慢运动行人漏检

