+++
date = "2018-01-11T11:53:58-08:00"
title = "FaceDetect算法对比测试"
tags = ["vision","test","facedetect","algorithm"]
topics = ["compute","vision"]

+++

## 测试说明

本对比测试使用不同的人脸检测算法对同样的两张图片样本进行检测：分别为单人和多人图片。

其中单人图片0_1_1.jpg大小为：550\*694，多人图片children.jpg大小为：660\*421.

通过对比各自算法在PC环境和Hi3519v101环境下的检测时间与检测准确率两项指标来评价算法的可行性。

## 测试结论

详细的测试步骤与输出信息见下面的测试环境结果，这里对这些信息进行简单的总结如下：

算法/性能 | PC Time(ms)|3519v101 Time(ms)|PC Correct|3519v101 Correct|
--------- |------------|-----------------|----------|----------------|
SeetaFaceEngine-FaceDet | 235    |3280   |6 in 6       |6 in 6             |
PICO - FaceDetetion     | 17|123    |4 in 6       |4 in 6             |
Opencv3.2 - HaarFrontalFace|170  |2144   |2 in 6       |2 in 6             |
Opencv3.2 - LBPFrontalFace|70    |213    |1 in 6       |1 in 6             |

注：PICO的性能在minsize和maxsize参数不同时，相差很大：

- min:16/max:200：PC 134ms, 3519v101 758ms
- min:50/max:200: PC 17ms,  3519v101 123ms

考虑到样本只有非常少的两个，因此上述测试数据不具有显著的性能指标代表性，只能是作为简单的参考数据。
为了得到更准确可靠的性能测试数据，可以通过提供大量的测试样本，比如规模上千的图片，通过脚本自动化工具对上述算法Demo进行计算测试，求取性能平均值即可（也就是标准的人脸检测测试集）。


## 对比环境：Ubuntu16.04(VM) + Core(TM)i5-4590@ 3.30GHz + 3GB RAM

### SeetaFaceEngine - FaceDetection

```bash
lzh@ubuntu:~/build$ ./facedet_test ../data/0_1_1.jpg ../model/seeta_fd_frontal_v1.0.bin 
Detections takes 0.23521 seconds 
OpenMP is used.
SSE is used.
Image size (wxh): 550x694

lzh@ubuntu:~/build$ ./facedet_test ../data/children.jpg ../model/seeta_fd_frontal_v1.0.bin 
Detections takes 0.233733 seconds 
OpenMP is used.
SSE is used.
Image size (wxh): 660x421
```

### PICO - FaceDetection/ObjectDetection

```bash
lzh@ubuntu:~$ ./picodetect_x86 ./facefinder ../data/0_1_1.jpg 
PICO Detections takes 0.118 seconds 
Detect Face num: 1
 0 Face Rect:  x:153  y:214 w:215 h:215
 
lzh@ubuntu:~$ ./picodetect_x86 ./facefinder ../data/children.jpg 
PICO Detections takes 0.134 seconds 
Detect Face num: 3
 0 Face Rect:  x:532  y:113 w:52 h:52
 1 Face Rect:  x:72  y:122 w:46 h:46
 2 Face Rect:  x:133  y:91 w:56 h:56
```

### OpenCV3.2 - Haar FrontalFace/LBP FrontalFace

#### Haar FrontalFace Cascade
```bash
$ ./cvfacedetect_x86 --cascade="./data/haarcascades/haarcascade_frontalface_alt.xml" --scale=1.3 ../3519v101_build/data/0_1_1.jpg
WARNING: Could not load classifier cascade for nested objects
Detecting face(s) in ../3519v101_build/data/0_1_1.jpg
detection time = 121.557 ms, faces num = 1
  face[0] circle center x = 261, y = 336, radius = 111
  
$ ./cvfacedetect_x86 --cascade="./data/haarcascades/haarcascade_frontalface_alt.xml" --scale=1.3 ../3519v101_build/data/children.jpg
WARNING: Could not load classifier cascade for nested objects
Detecting face(s) in ../3519v101_build/data/children.jpg
detection time = 167.765 ms, faces num = 2
  face[0] circle center x = 558, y = 141, radius = 25
  face[1] circle center x = 99, y = 147, radius = 26
```

#### LBP FrontalFace Cascade 

```bash
$ ./cvfacedetect_x86 --cascade="./data/lbpcascades/lbpcascade_frontalface.xml" --scale=1.3 ../3519v101_build/data/children.jpg
WARNING: Could not load classifier cascade for nested objects
Detecting face(s) in ../3519v101_build/data/children.jpg
detection time = 40.8521 ms, faces num = 1
  face[0] circle center x = 557, y = 138, radius = 24
  
$ ./cvfacedetect_x86 --cascade="./data/lbpcascades/lbpcascade_frontalface.xml" --scale=1.3 ../3519v101_build/data/0_1_1.jpg 
WARNING: Could not load classifier cascade for nested objects
Detecting face(s) in ../3519v101_build/data/0_1_1.jpg
detection time = 50.6144 ms, faces num = 1
  face[0] circle center x = 268, y = 330, radius = 114

```



## 测试环境：Hi3519v101 + 256MB RAM

辅助使用opencv3.2进行图像的读取写入等操作。

### SeetaFaceEngine - FaceDetection

```bash
/nfsroot/faceDetect/3519v101_build # ./facedetect_test ./data/0_1_1.jpg ./model/seeta_fd_frontal_v1.0.bin 
Detections takes 3.535 seconds 
OpenMP is not used. 
SSE is not used.
Image size (wxh): 550x694
Detect Face num: 1
 face[0] pos: x = 158,y = 216,w = 224,h = 224
 ```
 ![检测结果](/img/facedetect/0_1_1-seeta-fd.jpg)

 ```bash
/nfsroot/faceDetect/3519v101_build # ./facedetect_test ./data/children.jpg  ./model/seeta_fd_frontal_v1.0.bin 
Detections takes 4.128 seconds 
OpenMP is not used. 
SSE is not used.
Image size (wxh): 660x421
Detect Face num: 5
 face[0] pos: x = 533,y = 115,w = 48,h = 48
 face[1] pos: x = 75,y = 125,w = 47,h = 47
 face[2] pos: x = 271,y = 77,w = 51,h = 51
 face[3] pos: x = 348,y = 115,w = 38,h = 38
 face[4] pos: x = 430,y = 76,w = 53,h = 53
```
 ![检测结果](/img/facedetect/children-seeta-fd.jpg)

### PICO - FaceDetection/ObjectDetection

```bash
/nfsroot/faceDetect/PICO_FaceDetect # ./picodetect ./facefinder ../3519v101_build/data/0_1_1.jpg 
PICO Detections takes 0.705 seconds 
Detect Face num: 1
 0 Face Rect:  x:153  y:214 w:215 h:215
 ```
 ![检测结果](/img/facedetect/0_1_1-pico-fd.jpg)

 ```bash
/nfsroot/faceDetect/PICO_FaceDetect # ./picodetect ./facefinder ../3519v101_build/data/children.jpg 
PICO Detections takes 0.758 seconds 
Detect Face num: 3
 0 Face Rect:  x:532  y:113 w:52 h:52
 1 Face Rect:  x:72  y:122 w:46 h:46
 2 Face Rect:  x:133  y:91 w:56 h:56
```
 ![检测结果](/img/facedetect/children-pico-fd.jpg)

### OpenCV 3.2 - Haar/LBP Cascade

#### Haar FrontalFace_Alt

```bash
$ ./cvfacedetect_3519v101 --cascade="./data/haarcascades/haarcascade_frontalface_alt.xml" --scale=1.3 ../3519v101_build/data/0_1_1.jpg
WARNING: Could not load classifier cascade for nested objects
Detecting face(s) in ../3519v101_build/data/0_1_1.jpg
detection time = 2520.52 ms, faces num = 1
  face[0] circle center x = 261, y = 336, radius = 111
```
![检测结果](/img/facedetect/0_1_1-cvhaar-fd.jpg)

```bash
$ ./cvfacedetect_3519v101 --cascade="./data/haarcascades/haarcascade_frontalface_alt.xml" --scale=1.3 ../3519v101_build/data/children.jpg
WARNING: Could not load classifier cascade for nested objects
Detecting face(s) in ../3519v101_build/data/children.jpg
detection time = 2144.5 ms, faces num = 2
  face[0] circle center x = 99, y = 147, radius = 26
  face[1] circle center x = 558, y = 141, radius = 25
``` 
![检测结果](/img/facedetect/children-cvhaar-fd.jpg)
 
#### LBP FrontalFace

```bash
$ ./cvfacedetect_3519v101 --cascade="./data/lbpcascades/lbpcascade_frontalface.xml" --scale=1.3 ../3519v101_build/data/0_1_1.jpg 
WARNING: Could not load classifier cascade for nested objects
Detecting face(s) in ../3519v101_build/data/0_1_1.jpg
detection time = 268.299 ms, faces num = 1
  face[0] circle center x = 268, y = 330, radius = 114
```
![检测结果](/img/facedetect/0_1_1-cvlbp-fd.jpg)

```bash
$ ./cvfacedetect_3519v101 --cascade="./data/lbpcascades/lbpcascade_frontalface.xml" --scale=1.3 ../3519v101_build/data/children.jpg 
WARNING: Could not load classifier cascade for nested objects
Detecting face(s) in ../3519v101_build/data/children.jpg
detection time = 213.388 ms, faces num = 1
  face[0] circle center x = 557, y = 138, radius = 24
```
![检测结果](/img/facedetect/children-cvlbp-fd.jpg)

