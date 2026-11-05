图像存储与读取imread，imwrite，图像读取时可能窗口不够大，所以要nameWindow

图像的创建
Mat::zeros(size,CV_8UC3)
Mat::ones(size,CV_8UC1);
创建3通道的图片，只有第一个图层时1像素，对Mat进行int赋值也是修改第一个图层。要使用Scalar初始画布。

图像对象的读写操作：数组遍历和指针遍历；

色彩空间的转换cvtColor
COLOR_HSV2RGB
HSV中H(0~180),SV(0~255),V表示亮度，所以要修改亮度可以将BGR通道转换成HSV

OpenCv数据结构使用赋值是浅拷贝，深拷贝需要使用clone()或者copyTo;
浅拷贝会使Mat对象的引用计数加一，引用计数为0时进行内存释放。

参数的输入和输出，使用代理类对数据结构进行封装。