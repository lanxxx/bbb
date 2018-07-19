---
title: iOS：使用jenkins实现xcode自动打包（最新）
date: 2018-07-10 10:55:21
tags:
  - iOS
  - Python
---
参考各种教程实现Jenkins自动化打包遇到点坑，特此把自己成功安装的步骤记录一下。

**一.安装jenkins**

**首先使用osx系统自带的homebrew来安装jenkins。**

**在终端中运行：**

> **$ brew install Jenkins**  
>
> 第一步需要安装至少java1.8 ，如果没有安装会有提示，[java安装地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

![img](https://upload-images.jianshu.io/upload_images/3499748-e8624d879d33bacd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

无java时报错

java安装完毕，继续下面步骤，链接 launchd 配置文件

> $ ln -sfv/usr/local/opt/jenkins/*.plist ~/Library/LaunchAgents  
>
> //如果要其他机器也可以访问，把ip地址改为广播地址:--httpListenAddress=0.0.0.0
>
> $ launchctl load~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist  

命令行启动Jenkins

> $ jenkins

一切顺利的话，打开浏览器输入：http://localhost:8080/

就能看到jenkins已经运行起来了，如果你更换了端口就是你后来设置的端口。

接下来打开Jenkins后会让去一个填写password的页面如下图，存储password的地方就是图片上那行红色字体目录下，使用终端 cat + 红色字体路径就看到了

![img](https://upload-images.jianshu.io/upload_images/3499748-3744bb8265501e3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

然后将我们得到的password输入到“Administrator password“中，即可进入如下界面，接着安装一些建议的插件（左边的），安装过程中，有的插件可能会安装失败，强烈建议点击右下角的重试，直到把建议安装的都装好。（因为我这边安装之后，在Jenkins插件管理安装插件一直失败，如果这一步没把有些必须的插件装好，如git，只能一个个下载上传插件就很麻烦）

![img](https://upload-images.jianshu.io/upload_images/3499748-c330d4e3801bc125.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

插件安装完成后，可能会卡在如下界面，不会自动跳转，刷新下界面即可：

![img](https://upload-images.jianshu.io/upload_images/3499748-92680cf279d6f69c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

插件

在刷新后的界面中注册，输入用户名和密码，建议输入后点蓝色按钮保存完成，如下：

![img](https://upload-images.jianshu.io/upload_images/3499748-355ccb29dace90e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

注册成功后，点击”Start using Jenkins”

![img](https://upload-images.jianshu.io/upload_images/3499748-1a9a2f3dd1fd7a2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/491)

**二.jenkins的使用**

**到这一步，Jenkins的安装基本完成，接下来就是使用了，在新建任务之前需要安装好对应的插件**

**1.安装插件**

**Keychains and Provisioning Profiles Management（管理本地的keychain和iOS证书的插件）**

**Xcode integration （用于xcode构建）**

**打开系统管理，管理插件详见图**

![img](https://upload-images.jianshu.io/upload_images/3499748-6bda0d917eb1d719.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

**安装方式：**

**方案一：在可选中搜索插件名 ，勾选安装。若安装报错（参照安装Jenkins时不能联网安装插件问题解决），如果还不行，直接进入方案二。**

**方案二：去Jenkins-plug官网下载插件，然后选择高级tab，上传.hpi文件**

![img](https://upload-images.jianshu.io/upload_images/3499748-3e2da1a793135b1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

**2.配置Keychains and Provisioning Profiles Management**

接下来配置Keychains and Provisioning Profiles Management，根据顺序选择首页>*系统管理>Keychains and Provisioning Profiles Management*如图

![img](https://upload-images.jianshu.io/upload_images/3499748-7be0865bd31c03e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

**需要上传login.keychain文件，该文件获取方法，在终端中输入**

> cd ~/Library/Keychains

**在终端键入ls详见下图**

![img](https://upload-images.jianshu.io/upload_images/3499748-df6b6eb19bce631d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

将login.keychain文件upload之后，会出现下图的界面，根据需要将证书添加进去即可，但是由于macOS10.12以及以后的系统里面没有login.keychain文件，只有login.keychain-db,可以复制出来删除-db，也可以创建一个快捷方式名字叫做login.keychain，upload就好了。（上传keychain，路径用自己改名后的那个）

![img](https://upload-images.jianshu.io/upload_images/3499748-13e8ae3bc326ce0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

然后添加Provisioning Profiles，上传方法和上传login.keychain一样，去选择Provisioning Profiles文件，然后upload，然后结果如下图，**蓝色框**内的是固定格式的**/Users/用户名/Library/MobileDevice/Provisioning Profiles**

![img](https://upload-images.jianshu.io/upload_images/3499748-c748ac2b114c670e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

**3.开始配置任务**

然后创建一个任务，自由风格的任务，因为构建方法会有两种,除了共同的地方，构建会分为两部分解答，第一部分是macOS10.12以前的构建方法，但是由于Jenkins的Xcode和Mac的系统版本问题，所以建议使用第二种方法。

General

创建一个自由风格的任务，然后在选择丢弃旧的构建，至于天数和保持的最大个数，按照自己的需求来就好，如图

![img](https://upload-images.jianshu.io/upload_images/3499748-3ce2cf68feb817fc.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

源码管理

接下来是源码管理，在**Repository URL**里面添加你的git地址，我这里添加的是*http*的，如果你的项目是使用的ssh的，那么就将git开头的地址填写上，然后店家**Add**添加你的git帐号，如果你的事ssh的，将ssh的密匙填写上，具体的自己百度一下就好咯，我就不多写了，结果如图

![img](https://upload-images.jianshu.io/upload_images/3499748-60eaaca3a34ac160.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

构建触发器

接下来是构建触发器，也就是什么时候触发自动打包我这里填写的是H 20 * * *这个意思就是H小时然后，后面跟着数字，在后面就是日月年，*代表的我认为是每次都触发，也就是每天每月每年，但是Jenkins的时间不是绝对的，一般都是在随机在半点，也就是设置20点，大概会在20:30分左右会触发，如果需要两个时间，那么格式可以这样H 20,22 * * *结果如图

![img](https://upload-images.jianshu.io/upload_images/3499748-fb89921af6677d23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

构建环境

在构建环境里面勾选**Keychains and Code Signing Identities**和**Mobile Provisioning Profiles**，**Keychains and Code Signing Identities**是打包需要的证书，**Mobile Provisioning Profiles**是打包需要的配置文件，都是可以自己选择的，我的如图14

![img](https://upload-images.jianshu.io/upload_images/3499748-6ba2dc28a1b1abb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

接下来就是构建了，因为Jenkins对新版的Xcode插件兼容不好，只能用脚本来打包，但是老版本的Xcode可以使用Jenkins的Xcode插件，下面将构建一为使用Xcode插件的，构建二是脚本的

**构建一，使用Xcode插件**

在构建里面点击**增加构建步骤**，然后点击Xcode.

**General build settings**

在**Target**里面填写你的项目名称，**Clean before build**勾选YES，勾选**Pack application, build and sign .ipa?**，然后会有新的选项**Export method**是你要打的包的类型，就是你在手动打包的时候选额的ad-hoc或者Appstore那四个选项，这个按照你要打的类型填写，**.ipa filename pattern**是你打出包ipa的名字，我的这里填写了项目名字和-$(BUILD_DATE)，意思就是在后面追加时间，**Output directory**是导出ipa的目录，如果不填写，会在Jenkins默认的目录.

详见图

![img](https://upload-images.jianshu.io/upload_images/3499748-6dc4898f135c412b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

**Code signing & OS X keychain options**

勾选**Unlock Keychain?**,在**Keychain path**那里填写${HOME}/Library/Keychains/login.keychain,意思是找到你的login.keychain（登陆钥匙串），如果你的是复制出来改的名字，**那么就填写你相对应的目录**，**Keychain password**就是你电脑的登陆密码。详见图

![img](https://upload-images.jianshu.io/upload_images/3499748-09036c0b33938b73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

**Advanced Xcode build options**

勾选**Clean test reports?**，如果你使用了cocoapods那么填写**Xcode Workspace File**，如果没有使用cocoapods填写**Xcode Project Directory**，然后填写**Build output directory**就是你到处ipa的路径,详见图

![img](https://upload-images.jianshu.io/upload_images/3499748-111db80c71ce4e00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

完成之后，回到任务操作页面，点击立即构建，如果配置没问题基本就能构建成功

![img](https://upload-images.jianshu.io/upload_images/3499748-8fe67e5458292a1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

构建进行中

**偶尔会出现报错，点击任务编号，也就是上图的#4#5#6，然后进入任务详情页，进入控制台输出看看哪里出了问题，我之前是报了profile文件不匹配的error（target填错了），然后改了就好了**

![img](https://upload-images.jianshu.io/upload_images/3499748-ef98f94e9f348e1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

控制台输出log

![img](https://upload-images.jianshu.io/upload_images/3499748-1809e09adaa3f87c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

成功，小太阳出来了

![img](https://upload-images.jianshu.io/upload_images/3499748-48e4f9a1f7c80d32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

对应目录生成了文件，所生成的内容也是需要自己选择配置的

**构建二，使用脚本（蒲公英上传请参考https://www.jianshu.com/p/6bab38e569a5）**

因为Jenkins对现在的Xcode9插件兼容性不好，打不了包，所以我们使用了**xcodebuild**打包，下面是我的脚本，并且里面附上了自己非常详细的注释，可能有点啰嗦。

> \#!/bin/sh  #因为Jenkins打包可能是自动的，那么build号是不会自己再去修改然后push到git上面的，所以这个buildPlist就是修改build号的路径。 buildPlist="/Users/apple/.jenkins/workspace/longxin_a/eCloud/Build/LongHu/Config/eCloud-Info.plist"  #这个获取现在的 月日时分 用它来做build号 buildNumber=$(date +"%m%d%H%M")  #修改plist文件需要/usr/libexec/PlistBuddy -c命令，CFBundleVersion是修改的这个build号，$buildNumber是你要修改的数值，$buildPlist是你修改哪个地方的plist文件。 /usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "$buildPlist"  #这个是获取当前的build号，本来是用来看看有没有修改成功的 #newBuildName=$(/usr/libexec/PlistBuddy -c "print :CFBundleVersion" "$buildPlist") #这个是打印，带自动换行的打印 echo $newBuildName  #因为我怕他修改plist的时候需要时间，所以索性在这里我让他等了3秒，当然你也可以去掉 sleep 3  #这个buildPath是到时候我生成xcarchive文件的路径和打ipa时候需要找到xcarchive的路径 buildPath="/Users/apple/.jenkins/workspace/longxin_a/build/Release-iphoneos/eCloud.xcarchive"  #这个路径是我生成ipa的路径 ipaPath="/Users/apple/Documents/longhuBuild/"  #这个是ExportOptions.plist的路径，有这个就不用在用脚本写证书了，这个plist你只要手动打过包，那么在生成ipa的文件夹里面就会有，找一个自己不经常修改的地方放在那里，写上这个路径就好，当然如果你不想这么做，想用shell语言设置证书，我会在 问题 列表里面有介绍 exportOptionsplistPath='/Users/apple/.jenkins/workspace/ExportOptions.plist'  #因为我使用了cocoapods所以这里用的-workspace，如果你没有使用cocoapods使用-project，下面的命令都一样 #这个命令主要是用来clean，clean的是Release的路径，clean的是/Users/app***eCloud.xcworkspace路径的eCloud， xcodebuild -workspace /Users/apple/.jenkins/workspace/longxin_a/eCloud.xcworkspace -scheme eCloud -configuration "Release" clean  #这个是生成xcarchive，Release的 xcodebuild -workspace /Users/apple/.jenkins/workspace/longxin_a/eCloud.xcworkspace -scheme eCloud -archivePath ${buildPath} -configuration "Release" archive  #这个是将xcarchive文件打包成ipa xcodebuild -exportArchive -archivePath ${buildPath} -exportPath ${ipaPath} -exportOptionsPlist ${exportOptionsplistPath} -allowProvisioningUpdates

**邮箱通知**

到这里，其实你就已经打包成功了，但是打包成功后是不是我们需要通知一些人呢？Jenkins是有邮件通知的。

现在开始设置，首先你已经安装了插件Email Extension Plugin，这个在插件那里直接安装就好这是第一步；

然后进入**系统管理**->**系统设置**找到**Jenkins Location**模块，在**系统管理员邮件地址**填写你的系统管理邮箱，这个邮箱是你发送通知邮件的邮箱，

然后找到**Extended E-mail Notification**模块

填写方法如图18图19

![img](https://upload-images.jianshu.io/upload_images/3499748-6aa97f934b9e01b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

图18

![img](https://upload-images.jianshu.io/upload_images/3499748-740f5dc4bb67defb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

图19

然后找到**构建后操作**，点击**增加构建后的操作步骤**点击**Editable Email Notification**，在**Project From**里面写上管理者邮箱，也就是发送邮件的邮箱，然后点击**Advanced Setting**;

里面有三个选项，分别是你在系统设置里面勾选的那几个，根据需求填写就好，我这里填写的是Always，也就是无论构建成功还是失败,**Recipient List**是接收者的邮箱，**这里多个邮箱用英文逗号隔开——’,’**详见图20

![img](https://upload-images.jianshu.io/upload_images/3499748-b58f75ffeb8b689f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

图20

上传到fir

先去下载fir插件

然后在Jenkins里面点击**系统管理**->**管理插件**->**高级**，然后滑动到上传插件那里，选择刚才下载的插件，点击**上传**，等待上传成功后，进入到你的项目配置里面滑动到最下面，也就是找到**构建后操作**，点击**增加构建后操作步骤**，选择**Upload to fir.im**，打开你的浏览器，打开fir官方网站,获取方法见图21

![img](https://upload-images.jianshu.io/upload_images/3499748-fa9ca79d036f4791.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

21的

然后输入你的**IPA/APK Files (optional)**这个是你ipa的路径，如果不选择，会是Jenkins默认的路径

这里有fir的官方文档，根据fir的官方文档即可就可以Jenkins上传到fir文档

到此为止关于Jenkins整合Xcode的配置项并自动上传到蒲公英差不多就说好了。那么可以稍微构建一下项目试试了，构建完项目后，你就会发现测试人员不需要天天来烦你，再也不需要听到“开发狗，赶紧给我安装一个最新的版本”了。

各位可能会用到的友情链接：

[jenkins 密码错误去掉密码登录](https://www.cnblogs.com/xiami303/p/3625829.html)(mac路径为/Users/用户名/.jenkins)

[Jenkins 修改登录密码](https://blog.csdn.net/qq105319914/article/details/52094463)

[Jenkins 卸载](https://www.cnblogs.com/EasonJim/p/6277708.html)

参考教程：

http://blog.csdn.net/u014641783/article/details/50866196

https://blog.x1be.win/index.php/2018/06/19/jenkins%E5%AE%89%E8%A3%85%E3%80%81%E9%85%8D%E7%BD%AE%E3%80%81%E6%9E%84%E5%BB%BA%E3%80%81%E8%84%9A%E6%9C%AC%E3%80%81%E9%85%8D%E7%BD%AE%E9%82%AE%E7%AE%B1%E3%80%81%E4%B8%8A%E4%BC%A0fir/