---
layout: post
title: 源码分析之 - LoaderManager
date: 2023-07-07 14:44:45 +0800
categories: [Android源码, LoaderManager]
tags: [Android源码, LoaderManager]
author: 
---

> LoaderManager 

## LoaderManager 简介

`AsyncTask` 是 Android 实现异步方式之一，可以在子线程进行数据操作，主线程更新 `UI` 操作。

`AsyncTask` 被设计为一个围绕 Thread 和 Handler 的帮助类，并不构成一个通用的线程框架。

## LoaderManager 基本使用

