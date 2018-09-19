title: IOS-真机调试
date: 2016/3/9 8:51:22               
categories: IOS
---

# IOS-真机调试 #

## 真机调试的主要步骤 ##
1. 登录开发者主页
2. 生成cer证书：cer是一个跟电脑相关联的证书文件，让电脑具备真机调试的功能
3.     - 即苹果要知道哪一台电脑要进行真机调试(定位真机调试的电脑)
3. 添加App ID：调试哪些app？
4.     - 可以添加许多app
4. 注册真机设备：哪台设备需要做真机调试？
5.     - 需要知道你的手机的identifier
5. 生成MobileProvision文件：结合2、3、4生成一个手机规定文件
6. 导入cer、MobileProvision文件


- 完成以上两个步骤，最终会得到2个文件
>- Cer文件：让电脑具备真机调试的功能
>- MobileProvision文件：哪台设备、哪些app、哪台电脑需要做真机调试？

## 详细步骤图 ##

- 登录开发者主页
>- https://developer.apple.com/membercenter/index.action
>- 想要进行真机调试，必须要先买一个证书，这个证书有两个作用： 调试app； 在appStore上发布app
>    - 个人证书99$， 企业证书299$
>        - 企业证书不能把app发布到appStore，但是可以直接通过链接来下载app，主要用于针对某些特定领域的app发布使用
>    - 也可以去淘宝上买一个盗版的， 不过只能用于调试

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%951.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%952.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%953.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%954.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%955.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%956.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%957.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%958.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%959.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%9510.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%9511.png)


![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%9512.png)

![](http://7xrbxa.com1.z0.glb.clouddn.com/IOS%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%9513.png)


