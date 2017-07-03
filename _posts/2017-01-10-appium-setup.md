---
layout: post
title: Appium自动化测试环境建立 
categories: [Mobile]
tags: [自动化测试]
description: 在Ubuntu上搭建Appium自动化测试环境
---

tomorrow.cyz@gmail.com

# 1.Appium Server的安装

## 1.1 删除nodejs
官方的Appium文档支出最好不要使用sudo命令安装node、npm软件
 
  * sudo apt-get remove nodejs

## 1.2 下载二进制nodejs安装
  * [下载地址](https://nodejs.org/download/release/v5.6.0/)
    - 解压，将bin目录添加到PATH环境变量
    - 测试nodejs是否配置成功
          
        $node -v

        $npm -v

## 1.3 安装appium
        
        $npm install -g appium

## 1.4 配置Android SDK
        
        ANDROID_HOME = your_android_sdk_path/sdk
        PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools

## 1.5 配置JAVA
        
        export JAVA_HOME="/usr/lib/jvm/jdk7"
        export JRE_HOME=$JAVA_HOME/jre
        export PATH=$JAVA_HOME:$PATH/bin

## 1.6 测试
        
        $appium-doctor --android

## 1.7 安装Appium Python Client
        
        $ pip install Appium-Python-Client
        $ pip install pytest

## 1.8 模拟器配置
如果要使用模拟器，按照如下步骤设置

* $android list targets

* android create avd -n emulator-22 -t 10 --abi default/x86 
    - n后面跟的是名字
    - t后面跟的是id编号
    - $ emulator -avd emulator-22

## 1.9 运行
* 下载[github上官方测试样例脚本](https://github.com/appium/sample-code)
* 修改android_simple.py文件的setup方法
* appium -a 127.0.0.1 -p 4723  –U  4ca1558c  --no-reset 
    - 其中4ca1558c是真机的device name
* $ py.test android_simple.py

## 参考
### 1.[Ubuntu系统安装Appium及样例运行教程](http://www.jianshu.com/p/c7e27e0aefa9)
        
