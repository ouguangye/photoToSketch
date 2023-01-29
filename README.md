# photoToSketch
基于uniapp开发的图片转素描项目

## 算法实现原理
手机端由于性能上和平台上的限制，而且出于即时性的需求，这里采用的算法较为简单。 
算法实现的流程为： 图像转灰度图 -> 图像取反 -> 将取反的图像进行高斯模糊 -> 去色后的图像（灰度图）和取反模糊后的图像以混合模式为颜色减淡进行融合 -> 对图像进行ostu二值化处理。

出于跨平台的考虑，手机端我使用了uniapp进行开发， 能够在h5,android,ios多平台进行兼容。主要的数据图像处理工具选择了opencv.js.   
界面介绍 以及 相关图片试验成果：
点击选择图片可以选择自己相册中的图片。  上传完图片之后， 上面的三个按钮一次是刷新按钮， 下载按钮，分享按钮。 刷新按钮可以回到初始页面，重新上传图片。分享按钮 可以把图片分享到微信或者朋友圈。


## 运行结果

![a](https://user-images.githubusercontent.com/78332649/215319335-fc1b3c13-cdf6-4f58-82dc-74d27550fae9.jpg)
![b](https://user-images.githubusercontent.com/78332649/215319336-aba667db-a742-4eb2-8a12-ca88d7fbf5b9.jpg)
![c](https://user-images.githubusercontent.com/78332649/215319337-a706296b-b8e9-42e2-afdc-1c029a996d0c.jpg)
![d](https://user-images.githubusercontent.com/78332649/215319338-783c02c8-dea5-4d0a-90f0-73cb999490c6.jpg)
![g](https://user-images.githubusercontent.com/78332649/215319340-4a9f2656-ecfa-4558-92aa-a73a7849da7e.jpg)
