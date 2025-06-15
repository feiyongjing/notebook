### 图像的组成
- 图像是由像素构成


### 图像的分类
- 二值图像：只有黑和白两种颜色的像素图片
- 灰度图像：由黑到灰到白一共256种渐变颜色通道像素组成的图片
- RGB图像：由红、绿、蓝三种原生颜色各256种颜色通道，三种颜色通道组成了16777216种颜色基本涵盖了人类可感知的所有颜色，由这些颜色像素组成的彩色图片

## 安装opencv
~~~shell
# 提前准备python环境

# opencv-python 和 opencv-contrib-python 包是 OpenCV 的不同版本，它们分别是核心模块和扩展模块

# 核心模块安装
pip install opencv-python

# 扩展模块安装，包含了更多 OpenCV 的模块和功能，适合需要深度学习、物体识别等复杂任务的开发者
pip install opencv-contrib-python
~~~