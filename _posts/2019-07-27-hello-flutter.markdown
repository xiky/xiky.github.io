---
layout:     post
title:      "Hello Flutter"
subtitle:   " \"Hello World, Hello Flutter\""
date:       2019-07-27 12:00:00
author:     "Yxi"
header-img: "img/post-bg-2015.jpg"
catalog: true
<!-- tags:
    - 生活 -->
---

> “Yeah It's on. ”


## 前言

最近iOS团队越来越壮大了，趁着近期有时间

跟小伙伴们一起研究一下Flutter。


<p id = "build"></p>
---

## 安装

Flutter的安装很简单，只需要`flutter sdk` 和 *Android Studio* / *VSCode* 上的 **Dart** 与 **Flutter** 插件。

而 **Flutter** 是依赖于 **Dart**的，所以只需要安装 **Flutter** 插件即可


## 环境配置
[入门: 在macOS上搭建Flutter开发环境](https://flutterchina.club/setup-macos/)

## Flutter SDK
- 下载压缩包
- 解压到 指定目录
- 配置环境变量

[官方](https://flutter.dev/docs/get-started/install) 或者 [Flutter中文网](https://flutterchina.club/setup-macos/) 的教程都已经很详细了，这里配置就不再赘述了

安装过程中多使用 `flutter doctor` 命令来检查错误项，按照提示进行处理

如果在安装iOS pod环境时，下载过慢，可以在 `.gitconfig` 文件中，设置代理，并开启全局模式

```
# 设置终端翻墙代理
# [http]
# 	proxy = socks5://127.0.0.1:1086
# [https]
# 	proxy = socks5://127.0.0.1:1086
```
 
配置成功

```
flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, v1.7.8+hotfix.3, on Mac OS X 10.14.5 18F132, locale zh-Hans-CN)
[✓] Android toolchain - develop for Android devices (Android SDK version 29.0.1)
[✓] Xcode - develop for iOS and macOS (Xcode 10.2.1)
[✓] iOS tools - develop for iOS devices
[✓] Android Studio (version 3.4)
[✓] VS Code (version 1.36.1)
[✓] Connected device (2 available)

• No issues found!
```

---

## 配置IDE

#### Visual Studio Code (VS Code) 安装

*VS Code*: 轻量级编辑器，支持Flutter运行和调试.

##### 安装 VS Code

- [VS Code](https://code.visualstudio.com), 安装1.20.1或更高版本.

##### 安装Flutter插件

1. 启动 VS Code
2. 调用 **View>Command Palette…**
3. 输入 `install`, 然后选择 **Extensions: Install Extension** action
4. 在搜索框输入 `flutter` , 在搜索结果列表中选择 `Flutter`, 然后点击 **Install**
5. 选择 'OK' 重新启动 VS Code

##### 通过Flutter Doctor验证您的设置

1. 调用 **View>Command Palette…**
2. 输入 `doctor`, 然后选择 **‘Flutter: Run Flutter Doctor’** action
3. 查看“OUTPUT”窗口中的输出是否有问题


