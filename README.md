### BilibiliVideoDownload
----
基于bootstrap + jq 的单网页，可以下载bilibili视频
### 版本
----
v1.1.1 `2017-11-10`

修复视频格式显示错误

v1.1.0 `2017-11-05`

实现分P下载

注：由于api请求限制，目前只可以手动输入分p数获取分p下载链接。分p数为阿拉伯数字

v1.0.1 `2017-10-24`

优化了视频封面的获取方法

v1.0.0 `2017-10-21`

实现了基本功能
### 演示
----
![图片1](https://raw.githubusercontent.com/blogwy/BilibiliVideoDownload/master/img/1.png)
![图片2](https://raw.githubusercontent.com/blogwy/BilibiliVideoDownload/master/img/2.png)
![图片3](https://raw.githubusercontent.com/blogwy/BilibiliVideoDownload/master/img/3.png)
![图片4](https://raw.githubusercontent.com/blogwy/BilibiliVideoDownload/master/img/4.png)
### 演示地址
----
[点击查看](http://bt720p.com/bilibili/)
### 实现的功能
----
1. 获取视频直链
2. 获取视频大小和时长
3. 获取视频封面
4. 获取视频标题up主名字
5. 支持移动端
1. 获取分p下载链接
## 已知的问题
1. 由于使用了国外（Images.weserv.nl）的图片缓存代理，封面获取较慢。用图片缓存代理是为了突破b站视频封面域名限制，直接调用会出现403
2. 暂时没发现
### 感谢
----
1.⑨BiLiBiLi 第三方哔哩哔哩API提供平台

[http://9bl.bakayun.cn/](http://9bl.bakayun.cn/)

2.Images.weserv.nl图片缓存代理

[https://github.com/andrieslouw/imagesweserv](https://github.com/andrieslouw/imagesweserv)

3.bs and jq