---
layout: post
title: AS共享一个或n个Library
tags:
- Android Studio
- 多项目共享Library
categories: android
description:  刚从eclipse向androidstudio转项目有点多，公用的library也多，要是每个项目都导入library想起来都害怕于是想： androidstudio能不能不同的project共享library？
---
#### 多项目共享Library ####
前言：刚从eclipse向androidstudio转项目有点多，公用的library也多，要是每个项目都导入library想起来都害怕于是想： androidstudio能不能不同的project共享library？
在网上也有不少这样的解决方案但是我怎么也不能够成功，于是继续人肉搜索
最后在stackoverflow一个老外的回答
Error:Configuration with name 'default' not found in Android Studio
中得到了些线索于是就有了例一

<!-- more -->
## (1)Androidstudio多个project项目共享一个Library

例一：成立条件MySharedLibrary'要使用llib_A的useProject在同级目录

         1.新建一个MySharedLibrary'

         2. MySharedLibrary'中到如导入你已有的library< lib_A >（或者新建library< lib_A >）

         3.修改library  build.gradle加入

          dependencies {
             compile fileTree(dir:'libs', include: ['*.jar'])
          }

    4.在使用的useProject的setting.gradle加上

          include 'app', ':MySharedLibrary'
          project(':MySharedLibrary').projectDir = new File('../MySharedLibrary/lib_A)
     5.在使用的useProject的module中的build.gradle的dependencies加入以下代码

          compile project(':MySharedLibrary')

## (2).Androidstadi多个project项目共享n个Library

上面的例一成功了于是自己又在此基础上扩展了下面的例二

Androidstadi多个project项目共享n个Library

例二：成立条件MySharedLibrary'要使用llib_A，lib_B，lib_C的useProject在同级目录(MySharedLibrary')

    1.新建一个MySharedLibrary'

         2. MySharedLibrary'中到如导入你已有的library< lib_A >,< lib_B >，< lib_C >（或者新建library  l< lib_A >,< lib_B >，< lib_C >）

         3.修改< lib_A >,< lib_B >，< lib_C >  build.gradle加入

          dependencies {
             compile fileTree(dir:'libs', include: ['*.jar'])
          }

    4.在使用的useProject的setting.gradle加上

          include 'app', ':MySharedLibrary/lib_A'
          include 'app', ':MySharedLibrary/lib_B'
          include 'app', ':MySharedLibrary/lib_C'
          project(':MySharedLibrary').projectDir = new File('../MySharedLibrary/lib_A)
          project(':MySharedLibrary').projectDir = new File('../MySharedLibrary/lib_B)
          project(':MySharedLibrary').projectDir = new File('../MySharedLibrary/lib_C)
     5.在使用的useProject的module中的build.gradle的dependencies加入以下代码

          compile project(':MySharedLibrary/lib_A ')
          compile project(':MySharedLibrary/lib_B ')
          compile project(':MySharedLibrary/lib_C ')

作者：天空蓝蓝的  www.lskyf.com   www.lskyf.xyz  
版权所有，欢迎保留原文链接进行转载：)