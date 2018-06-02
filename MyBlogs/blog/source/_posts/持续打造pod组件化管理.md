---
title: 持续打造pod组件化管理
date: 2018-06-01 18:11:34
categories: 工具篇
tags: [工具,技巧,效率]
---

![工具](持续打造pod组件化管理/tool.jpg)

首先你会把一个项目的部分模块独立出来,然后放到项目的某个文件夹里,项目需要pod管理就得创建一个后缀为podspec的文件进行关联管理 

```swift
///MARK:- 执行以下语句 会生成一个DemoName.podspec文件根据模板和文档进行编辑  
pod spec create DemoName
```

<!-- more -->

 ./ 是当前目录 
 ../是返回上一层

 ~/ 系统根目录 一般来说桌面DeskTop就是在系统根目录的

#### 本地项目管理 [精简版]

```ruby
Pod::Spec.new do |s|
  s.name         = "TestCodeDemo"
  s.version      = "0.0.1"
  s.summary      = "Order for tests pods testsDemo."
  s.homepage     = "https://www.baidu.com"
  s.author             = { "Use Code Change The World" => "" }
  s.platform     = :ios, "9.0"
  s.source       = { :git => "/Users/ddys/Desktop/TestCodeDemo/TestCodeDemo.podspec", :tag => "#{s.version}" }
  s.source_files  = "*/TestObject.{h,m}"
  s.requires_arc = true
end
```
 

##### 写好之后 免警告⚠️检查
```
pod lib lint --allow-warnings
```

##### 引用的时候
```
platform :ios, '9.0'
target "demo" do
pod "TestCodeDemo", :path => '/Users/ddys/Desktop/TestCodeDemo/TestCodeDemo.podspec' 
end 

```


#### git或者svn
```
// 本地文件
s.source = { :git => '~/Desktop/ChartboostSDK' }    

// git
s.source = { :git => 'https://github.com/silan-liu/RGB.git' }    

// zip 
s.source = {:http=> 'http://xxx.zip', :tag => "1.0.0" }

// svn
s.source = {:svn=> 'http://path', :tag => "1.0.0" }
```


[参考一](https://www.jianshu.com/p/5b950e89e1ac)
[参考二](https://www.jianshu.com/p/d2b0acfad0d8)
[参考三](https://www.jianshu.com/p/f146fa5358d5?mType=Group)
```
Pod::Spec.new do |s|   
s.name = 'DemoSdk'    
s.version = '0.0.1' 
s.license = 'Commercial' 
s.summary = 'DemoSdk is very beauty.' 
s.homepage = 'https://example.com/'
 s.author = { 'today' => '666666@qq.com' }  
s.platform        = :ios,"7.0"
s.source          =  { :git => "~/Desktop/DemoSdk" }  #本地的路径
s.source_files    = "DemoSdk/**/*.{h,m}"              #制定需要引入的文件
s.requires_arc    = true
end

```

```
 platform :ios,'7.0'
  pod 'DemoSdk' ,  :path =>'~/desktop/DemoSdk'
```

#### 分子模块的写法 [可参考AFNetworking.podspec](https://github.com/AFNetworking/AFNetworking/blob/master/AFNetworking.podspec)
```
该测试库的目录结构如下: 
- |_WGBTestPodFiles
	-|__ AppGroup
		-|___ Animation
			-|____ WGBAnimation.h
			-|____ WGBAnimation.m
		-|___ AppOne
			-|____ AppOne.h
			-|____ AppOne.m
		-|___ Person
			-|____ Person.h
			-|____ Person.m

如果不分子模块,所有文件将会装到一个文件下,那样文件一多就不容易找,不够清晰
一些大库,都是分子模块的... 如下语法: 

Pod::Spec.new do |s|
    s.name         = 'WGBTestPodFiles'
    s.version      = '1.0.1'
    s.summary      = 'An easy way to test pod files'
    s.homepage     = 'https://github.com/WangGuibin/WGBTestPodFiles'
    s.license      = "MIT"
    s.authors      = {'WGB' => '864562082@qq.com'}
    s.platform     = :ios, '9.0'
    s.source       = {:git => 'https://github.com/WangGuibin/WGBTestPodFiles.git', :tag => s.version}
    # s.source_files = ''
    s.requires_arc = true
    
    s.subspec 'Animation' do |ss|
    ss.source_files = 'WGBTestPodFiles/**/WGBAnimation.{h,m}'
    end
  
  s.subspec 'AppOne' do |ss|
    ss.source_files = 'WGBTestPodFiles/**/AppOne.{h,m}'
  end
  
  s.subspec 'Person' do |ss|
    ss.source_files = 'WGBTestPodFiles/**/Person.{h,m}'
  end
  
end
```




### 打造私有库 podspec

- 第一步把项目托管git 并打上tag,相当于版本号
```
 git tag -m"tag" "0.0.1"
 git push --tags
```
- 在github上创建私有spec repo 即私有库管理的一个库, 拷贝链接 执行以下代码
```
pod repo add MySpec https://github.com/CoderWGB/WGBSpec.git
```
它会克隆一份,
然后在` ~/.cocoapods/repos`下可以看到 `MySpec`

- 回到项目的.podspec所在目录, 检查填写正确✅,推送到远端同步起来,基本ok
```
pod repo push MySpec WGBCustomAlertPopupView.podspec
```
看到如下打印,说明已经成功了一半
```
wangguibindeMacBook-Pro:WGBCustomAlertView wangguibin$ pod repo push MySpec WGBCustomAlertPopupView.podspec

Validating spec
 -> WGBCustomAlertPopupView (0.0.1)

Updating the `MySpec' repo

Your configuration specifies to merge with the ref 'refs/heads/master'
from the remote, but no such ref was fetched.

Adding the spec to the `MySpec' repo

 - [Add] WGBCustomAlertPopupView (0.0.1)

Pushing the `MySpec' repo

wangguibindeMacBook-Pro:WGBCustomAlertView wangguibin$ 

```

- 以上不用审核可以直接使用
发布到cocopods官方,基于上面的打了tag,spec文件也编写正确,有以下操作即可,以后修改只要改改版本号,tag打高一点点就行啦(PS. 时间可能会有点长,因为要更新其他人的库文件,需要耐心等待,干点其他的去) 
```
//若没有注册指定pod trunk的邮箱和用户名 得先执行
pod trunk register xxxxx@163.com 'UserName'  --verbose
// 推送到pod官方去审核
pod trunk push WGBCustomAlertPopupView.podspec
//更新版本也是这个操作 打完tag 修改一下podspec文件的东西细节
然后执行 pod trunk push WGBAutoScrollNoticeView.podspec --allow-warnings
```
完事之后呈现如下打印
```
Validating podspec
 -> WGBCustomAlertPopupView (0.0.1)

Updating spec repo `master`

--------------------------------------------------------------------------------
 🎉  Congrats

 🚀  WGBCustomAlertPopupView (0.0.1) successfully published
 📅  January 23rd, 10:21
 🌎  https://cocoapods.org/pods/WGBCustomAlertPopupView
 👍  Tell your friends!
--------------------------------------------------------------------------------
wangguibindeMacBook-Pro:WGBCustomAlertView wangguibin$ 
```


- 引用项目的时候,podfile填写细节
```
source "https://github.com/CoderWGB/WGBSpec.git" 
#以上这句必须填写 不然找不到这个库是从哪里加载过来的
platform :ios,'8.0'
target "WGB_MyDemoApp" do
pod 'WGBCustomAlertPopupView','0.0.1'
end
```
参考了以下文章 感谢他!!!
https://www.jianshu.com/p/5b950e89e1ac


cocoapods发布成功之后,`pod search repoName`一下
如果搜索不出来,原因有两种可能,一是没有发布成功!二是cocoapods的本地配置问题

##### CocoaPods: pod search 搜索类库失败的解决办法


执行`pod search`来搜索类库信息时，却总是
`[!]Unable to find a pod with name, author, summary, or descriptionmatching`

- 执行`pod setup`
其实在你安装CocoaPods执行pod install时，系统会默认操作`pod setup`，然而由于中国强大的墙可能会`pod setup`不成功。这时就需要手动执行`pod setup`指令，如下：
终端输入：`pod setup`
会出现`Setting up CocoaPods master repo`，稍等几十秒，最底下会输出`Setup completed`。说明执行`pod setup`成功。
- 如果`pod search`操作还是搜索失败，如下：
终端输入：`pod search AFNetworking`
输出：`Unable to find a pod with name, author, summary, or descriptionmatching 'AFNetworking'` 这时就需要继续下面的步骤了。
删除`~/Library/Caches/CocoaPods`目录下的`search_index.json`文件
pod setup成功后，依然不能`pod search`，是因为之前你执行`pod search`生成了`search_index.json`，此时需要删掉。
终端输入：`rm ~/Library/Caches/CocoaPods/search_index.json`
删除成功后，再执行`pod search`。
- 执行`pod search`
终端输入：`pod search afnetworking`(不区分大小写)
输出：`Creating search index for spec repo 'master'.. Done!`，稍等片刻······就会出现所有带有`afnetworking`字段的类库。


#### 更新 推送到cocopods管理的时候,遇到如下问题
```
unknown: Encountered an unknown error (/usr/bin/xcrun simctl list -j devices

xcrun: error: unable to find utility "simctl", not a developer tool or in PATH
) during validation.
```
`这个问题是XCode命令行工具的问题` 重新选择Xcode 的developer就好了
```
 sudo xcode-select -switch Xcode路径/Contents/Developer 即可 (Xcode路径打开finder查看包内容,打开contents > developer 把文件夹拖到终端就是路径了)
```
[参考segment问答](https://segmentfault.com/q/1010000012705430/a-1020000012710662)