# Typora图片上传七牛云

我在写笔记的时候，都是基于Typora来完成的。虽然Typora很方便，但是它的图片是保存在本地的，笔记写完之后，我想上传到公众号或者博客，就会出现图片丢失的情况。

为了解决这一问题，我选择了七牛云图床＋picGo。

如果操作呢？

## 1. 下载Typora

首先，需要有Typora，[下载地址](https://typora.io/)。

PS：免费，用过的都说好。

## 2. 七牛云安装及操作

2. 我们需要注册七牛云账号，在此就不必叙述，注册认证后有10G永久免费空间，每月10G国内和10G国外流量。[七牛云官网](https://portal.qiniu.com/)

2. 上述操作完成之后，点击右侧菜单栏中的【对象存储】，选择【空间管理】，完成如下的操作


![](http://img.akangaroo.cn/typora/typoraQiniuyun1.png))

4. 接着我们会看到提示【空间创建成功】，建议绑定自己的域名，测试域名一个月之后会回收。
5. 此处假设你有域名，在加速域名中输入你想要加速的域名，这里基于你已经备案了，如果不会的，可以联系我询问。如果是图片的话可以改成image.akangaroo.cn，名称无所谓，只是方便区别而已。

![image-20200912183013545](http://img.akangaroo.cn/typora/typoraQiniuyun2.png)

6. 接着，我们需要配置CNAME。在 **七牛开发者平台** 页面选择 【CDN】，选择 域名管理，鼠标停留在域名对应CNAME值上即可点击复制。打开阿里云，进入域名解析页面，点击【添加记录】。

![img](http://img.akangaroo.cn/typora/typoraQiniuyun3.png)

7. 七牛云的操作已经完成。

## 3. picGo操作

1. 下载picGo，[下载地址](https://github.com/Molunerfinn/PicGo/releases)，地址为Github的，如果下载慢的话，之后我会分享。

2. 打开图床设置，选择七牛图床

   - AccessKey和SecretKey我们打开七牛云，点击右上角的头像，前往【个人中心]，【密钥管理】中有对应的值，复制进来即可。

   - 存储空间名是我们刚刚设置的，我刚刚写的是akangaroo2
   - 设定访问网址为http://vedio.akangaroo.cn，也是我们刚刚设置的，记住一定要加http://
   - 确认存储区域填z0-z5中的一个即可，如果填了之后不行的，待会我们可以打开日志文件查看。
   - 设定网址后缀可不填
   - 指定存储路径的话，建议/typora，方便区分。

   ![image-20200912185159489](http://img.akangaroo.cn/typora/typoraQiniuyun4.png)

3. 如上图，我们选择上传区，选择图片上传，查看是否有效（进度条走完显示绿色为有效，红色则无效），有效的话，可以在七牛云图床中看到我们刚刚上传的图片。

4. 如果无效的话，我们查看自己是否将七牛云的空间设置为公开。来到主页面，选择【对象存储】，然后【空间管理】，查看空间是否为公开，没有的话点击右边的设置，在空间设置中，将访问控制设置为公开。

   我这里是已经为公开了，所以显示设置为私有空间。

   ![image-20200912190040621](http://img.akangaroo.cn/typora/typoraQiniuyun5.png)

5. 如果还是不行的话，如下图所示，点击PicGo设置，选择【设置日志文件】

![image-20200912190212940](http://img.akangaroo.cn/typora/typoraQiniuyun6.png)

![image-20200912190242161](http://img.akangaroo.cn/typora/typoraQiniuyun7.png)

![image-20200912190342128](http://img.akangaroo.cn/typora/typoraQiniuyun8.png)

找到如上图所示的语句，然后将之前的确认存储区域修改为z2。本来我设置的是z0，至于为什么，我也不是很清楚，但是看日志还是很重要的！

6. 至此，我们已经可以将本地图片直接上传到七牛云了。

## 4. Typora整合picGo

1. 在上述所有操作都没有问题之后，最后一步，也是最简单的一步。
2. 打开Typora，点击左上角的文件，选择【偏好设置】。

![image-20200912190808079](http://img.akangaroo.cn/typora/typoraQiniuyun9.png)

3. 然后可以点击【验证图片上传选项】，不出意外，是会成功的。
4. 至此，所有操作都ok了！
5. 我们可以在写笔记的时候，图片自动的上传到图床，真正实现了一次编写，随处移植