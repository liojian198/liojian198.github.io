---
title: "使用flutter过程中，零碎的疑问点"
collection: talks
type: "talks"
excerpt: ''
permalink: /talks/flutter_problem
date: 2025-08-27
toc: true # 启用目录
toc_label: "使用flutter过程中，零碎的疑问点" # 目录标题
toc_sticky: true # 目录是否固定在侧边 (可选)
---


# 使用flutter过程中，零碎的疑问点

## flutter 桥接原生的代码库 例如蓝牙代码

Flutter 的核心理念是“自绘”，它的大部分 UI 渲染都不依赖于原生组件。但对于设备硬件功能，如蓝牙、GPS、摄像头等，Flutter 依然需要**桥接（bridge）**到原生的代码库。

Flutter 提供了 Platform Channels（平台通道）机制来完成这项任务。你可以把它想象成一个双向信使，用于在 Dart 代码和原生平台（iOS 的 Swift/Objective-C，Android 的 Kotlin/Java）之间传递信息和数据。

以蓝牙功能为例，由于 Dart 语言本身没有直接操作设备蓝牙硬件的 API，你需要通过以下步骤来调用原生的代码库：

桥接原生代码的“蓝图”
第一步：定义通信渠道（Platform Channel）
首先，你需要在 Dart 端和原生端定义一个具有唯一名称的通道。这个名称就像一个“邮局地址”，确保消息能被正确地发送和接收。

Dart 端：

Dart

static const platform = MethodChannel('com.yourapp.bluetooth');
第二步：从 Dart 端发送消息
在你的 Flutter 应用中，你可以通过 MethodChannel 发送一个异步消息，附带一个方法名和可选的参数。

Dart 端：

Dart

// 请求原生代码库扫描设备
Future<void> scanForDevices() async {
  try {
    // 调用原生方法，并等待结果
    final String result = await platform.invokeMethod('scanForDevices');
    print(result); 
  } on PlatformException catch (e) {
    print("Failed to scan devices: '${e.message}'.");
  }
}
第三步：在原生端处理消息
在 iOS (Swift) 和 Android (Kotlin) 代码中，你需要监听来自该通道的消息。

iOS (Swift):

Swift

import Flutter
import UIKit

// 在 AppDelegate 中设置消息监听器
let flutterViewController: FlutterViewController = window?.rootViewController as! FlutterViewController
let channel = FlutterMethodChannel(name: "com.yourapp.bluetooth", binaryMessenger: flutterViewController.binaryMessenger)

channel.setMethodCallHandler({
  (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
  // 检查方法名是否匹配
  if call.method == "scanForDevices" {
    // 这里调用 iOS 原生蓝牙代码库 (Core Bluetooth) 的逻辑
    // ...
    result("Native code is scanning for devices!")
  } else {
    result(FlutterMethodNotImplemented)
  }
})
Android (Kotlin):

Kotlin

import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

class MainActivity: FlutterActivity() {
    private val CHANNEL = "com.yourapp.bluetooth"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL).setMethodCallHandler { call, result ->
            // 检查方法名是否匹配
            if (call.method == "scanForDevices") {
                // 这里调用 Android 原生蓝牙代码库 (BluetoothAdapter) 的逻辑
                // ...
                result("Native code is scanning for devices!")
            } else {
                result(null)
            }
        }
    }
}
第四步：将结果返回给 Dart
原生代码通过 result() 回调将数据返回给 Dart 端，Dart 端的 invokeMethod 即可收到这个结果。

更现实的选择：使用现有插件
在实际开发中，你几乎不需要自己从头编写上述的平台通道代码。

Flutter 的生态系统非常成熟，社区已经创建了大量的插件（packages）。这些插件已经为你处理好了所有平台通道的细节。例如，flutter_blue_plus 就是一个流行的蓝牙插件，它已经为你封装好了 iOS 和 Android 的所有蓝牙原生代码，你只需要在 Dart 端调用其提供的 API 即可。

这个“调用原生代码库”的机制，正是 Flutter 能够同时提供高性能 UI 和强大硬件访问能力的关键。


## RN 桥接原生的蓝牙过程是怎么样的？

与 Flutter 类似，React Native 同样依赖于**桥接（Bridge）**机制来调用原生平台的硬件功能，如蓝牙。

然而，由于 React Native 本身就是基于原生组件渲染的，其桥接过程与 Flutter 的“从零开始”有所不同。它的核心在于创建一个原生模块（Native Module），并在 JavaScript 中暴露其接口。

以蓝牙为例，桥接过程如下：

第一步：创建原生模块 (Native Module)
你需要在 iOS (Objective-C/Swift) 和 Android (Java/Kotlin) 中分别创建原生模块。这个模块将包含所有与蓝牙硬件交互的逻辑。

iOS：创建一个继承自 RCTBridgeModule 的 Swift 或 Objective-C 类。

Android：创建一个继承自 ReactContextBaseJavaModule 的 Java 或 Kotlin 类。

这些原生模块内部会调用各自的蓝牙 SDK：iOS 的 Core Bluetooth 或 Android 的 BluetoothAdapter。

第二步：在 JavaScript 中暴露原生方法
原生模块需要明确地向 JavaScript 桥接层“注册”它所提供的所有方法。

在 iOS 中，使用 RCT_EXPORT_METHOD() 宏。

在 Android 中，使用 @ReactMethod 注解。

这些方法将作为 JavaScript 端可以调用的接口。例如，你可以暴露一个 scanForDevices() 方法。

第三步：从 JavaScript 端调用原生模块
在你的 React Native JavaScript 代码中，你可以通过 NativeModules API 来访问并调用你创建的原生模块和方法。

JavaScript

import { NativeModules } from 'react-native';

const { BluetoothModule } = NativeModules;

// 调用原生模块的蓝牙扫描方法
async function scanDevices() {
  try {
    const result = await BluetoothModule.scanForDevices();
    console.log(result);
  } catch (error) {
    console.error(error);
  }
}

scanDevices();
第四步：数据回传
原生模块可以通过**回调（Callbacks）**或 Promise 将结果返回给 JavaScript 端。当蓝牙扫描完成后，原生代码会通过这些机制将结果（如扫描到的设备列表）发送回 JavaScript，供你的 React Native 应用处理。

总结与实际选择
与 Flutter 类似，你通常不需要自己从头编写这些复杂的原生桥接代码。

React Native 社区已经创建了大量的第三方库（Libraries），例如 react-native-ble-plx，它们已经为你封装了所有原生蓝牙代码。你只需像使用普通 JavaScript 库一样，在项目中安装并使用它们即可。

这个“桥接”机制是 React Native 能够实现跨平台开发的关键，它使得 JavaScript 能够轻松地与底层硬件和原生功能进行通信。

## FlutterView 和 Skia 之间的关系是分层的，而不是直接调用

它触及了 Flutter 渲染架构的核心。

FlutterView 和 Skia 之间的关系是分层的，而不是直接调用。你可以把这个过程分解为以下几步：

Flutter 引擎: 这是 Flutter 的核心。它包含 Dart 虚拟机和 Skia 渲染引擎。

Skia 引擎: 这是一个用 C++ 编写的高性能 2D 图形库。它可以生成与平台无关的渲染指令。

原生图形 API: 这是 iOS（Metal）和 Android（OpenGL ES）提供的底层 API，用于直接与 GPU 交互。

为什么不是 Skia 直接调用？
Skia 本身是一个通用的图形库，它的设计目标是生成抽象的绘制命令。它需要一个更底层的后端来将这些命令转换为 GPU 可以理解的、特定于平台的指令。这个后端就是原生图形 API。

你可以这样理解：

Skia 就像一个高级的图形设计师。它知道如何绘制复杂的形状、路径和文本。它会把所有的设计图（例如，“画一个半径为 50 的蓝色圆形，再画一条红色的直线”）整理好。

原生图形 API (Metal/OpenGL ES) 就像一个能理解 GPU 语言的工匠。它接收设计师的图纸（Skia 生成的指令），并使用 GPU 的专用工具（硬件加速）来完成实际的绘制工作。

具体过程
Dart 代码: 你的 Flutter 代码（例如 CustomPaint 中的 canvas.drawCircle()）被编译成 Dart-to-C++ 指令，发送给 Flutter 引擎。

Skia: Flutter 引擎中的 Skia 引擎接收到指令，并将其转换为一系列与平台无关的绘制命令，这些命令存储在 Skia 的渲染后端中。

FlutterView: FlutterView 扮演了 Skia 引擎和原生 API 之间的中介角色。它提供了一个与原生平台图形上下文（Graphics Context）绑定的表面。当 Skia 准备好绘制时，FlutterView 会调用原生图形 API，将 Skia 的绘制命令作为参数传递给它。

GPU: 原生图形 API（Metal/OpenGL ES）接收到这些命令后，将其发送给 GPU。GPU 执行这些指令，最终在屏幕上渲染出像素。

这个分层设计的好处是，Flutter 开发者只需要和上层的 Dart API 交互，而底层复杂的渲染细节则由 Skia 和原生图形 API 共同处理。这种架构确保了 Flutter 在不同平台上都能获得原生级的性能，同时保持了代码的跨平台一致性。

## 什么是 Flutter 引擎嵌入器？

Flutter 引擎嵌入器（Engine Embedder） 是 Flutter 框架的一个核心组件，它充当 Flutter 引擎与特定原生平台（如 Android、iOS、Windows、macOS、Linux，甚至 HarmonyOS）之间的中介层。

简单来说，它的主要工作是：

为 Flutter 引擎提供一个“画布”：它从原生平台（例如，Android 上的 Surface 或 iOS 上的 UIView）获取一个图形表面，让 Flutter 引擎的 Skia 渲染器可以在上面绘制 UI。

管理原生平台事件：它接收来自原生平台的事件（如触摸、键盘输入、鼠标事件），并将其转换成 Flutter 引擎可以理解的格式。

处理生命周期：它将原生应用的生命周期事件（如应用启动、暂停、恢复、关闭）传递给 Flutter 引擎，确保 Flutter 应用能正确响应。

可以把嵌入器想象成一个翻译官和舞台管理员：它将 Flutter 引擎的指令翻译给原生系统，同时又为 Flutter 引擎准备好表演的舞台（画布）和道具（输入事件）。

嵌入器如何工作？
Flutter 的核心思想是平台无关。Flutter 引擎本身并不关心它运行在哪种操作系统上，它只知道如何运行 Dart 代码和渲染 UI。而嵌入器正是连接这个“平台无关”的引擎和“平台相关”的原生世界的关键。

以一个 Android 应用为例，嵌入器的工作流程是这样的：

应用启动：当你的 Android 应用启动 FlutterActivity 时，嵌入器会被初始化。

创建上下文：嵌入器会创建一个 FlutterEngine 实例，并在 Android 系统中获取一个 FlutterView（即画布）。

运行 Dart 代码：嵌入器将你的 Dart 代码加载到 Dart 虚拟机中，并开始执行。

渲染 UI：当 Dart 代码创建 UI 组件时，Flutter 引擎会生成渲染指令。嵌入器会将这些指令传递给 FlutterView，后者使用原生图形 API（OpenGL ES/Vulkan）在屏幕上绘制像素。

处理用户输入：当用户触摸屏幕时，Android 系统将事件传递给 FlutterView。嵌入器捕捉到这个原生事件，将其转换为 Flutter 格式，并发送给 Dart 代码。

为什么嵌入器如此重要？
跨平台能力的核心：嵌入器使得 Flutter 的单一套代码可以在不同的操作系统上运行，而无需修改核心引擎。Google、华为和社区正是通过为不同的平台开发定制的嵌入器，才让 Flutter 能够支持 Android、iOS、Web、Windows、macOS、Linux，甚至 HarmonyOS。

灵活性和可扩展性：由于嵌入器是独立的，开发者可以将其嵌入到任何原生应用中，从而实现混合开发。你可以将一个 Flutter 界面嵌入到现有的 Android Activity 或 iOS UIViewController 中。

总之，Flutter 引擎嵌入器是 Flutter 跨平台哲学的具体实现，它负责所有底层的平台通信和管理工作，让开发者可以专注于用 Dart 构建 UI，而不用关心底层平台的复杂性。

## 在 Flutter 中，Dart 与原生语言之间的内存管理

在 Flutter 中，Dart 与原生语言之间的内存管理，主要是通过**垃圾回收（Garbage Collection）和引用计数（Reference Counting）两种机制，在各自的领域内独立进行，并通过接口（channels）**进行数据传递。

Dart 端的内存管理
Dart 是一种拥有垃圾回收机制的语言。

自动管理：你不需要手动分配或释放内存。Dart 的垃圾回收器会自动识别不再被引用的对象，并回收它们占用的内存。

分代垃圾回收：Dart 的垃圾回收器采用了一种高效的分代算法。新创建的对象被分配到“新生代（New Generation）”，当对象存活一段时间后，会被晋升到“老年代（Old Generation）”。这种设计可以更快地清理掉生命周期短的临时对象。

因此，当你创建 Dart 对象（如 String、List、Widget）时，内存由 Dart 虚拟机自动管理，你不需要担心内存泄漏问题。

原生端的内存管理
Android (Java/Kotlin)：与 Dart 类似，Java 和 Kotlin 也是通过垃圾回收来管理内存。Java 虚拟机（JVM）有自己的垃圾回收器，它会独立于 Dart 虚拟机工作。

iOS (Swift/Objective-C)：iOS 主要使用 **ARC（自动引用计数，Automatic Reference Counting）**来管理内存。当一个对象的引用计数变为 0 时，系统会自动释放该对象。这个机制同样独立于 Dart 的垃圾回收器。

桥接时的内存管理
当 Dart 与原生代码通过 Method Channel 等机制进行通信时，内存管理是独立的，但数据传递的方式很巧妙。

数据序列化：当你从 Dart 向原生传递数据（或反之）时，数据会被序列化。这意味着，数据会从一种语言的对象格式（如 Dart 的 Map），转换为一种通用的、可跨语言传输的格式（例如 JSON 或自定义的二进制格式）。

数据复制：序列化后的数据会通过 Flutter 引擎的 二进制信使（BinaryMessenger） 在 Dart 和原生平台之间传输。在这个过程中，数据实际上是复制过去的，而不是共享内存。

独立管理：一旦数据到达目标端，它会被反序列化为目标语言的对象。例如，一个 Dart 的 Map 会变成一个 Swift 的 NSDictionary 或 Kotlin 的 HashMap。此时，这个新创建的对象会由目标语言的内存管理机制独立管理。

这意味着 Dart 和原生代码的内存是隔离的。 它们不会直接共享内存空间，一个平台上的内存管理问题通常不会直接影响另一个平台。这种隔离的设计是 Flutter 架构稳健性的关键，可以有效防止跨语言的内存泄漏。

## 平台通道（Platform Channels）

在 Flutter 中，与原生代码（Android 和 iOS）进行桥接是实现高级功能的核心方式。由于 Flutter 自身不直接访问底层硬件或操作系统 API，因此需要一个机制来调用原生平台的功能。这个机制主要通过**平台通道（Platform Channels）**来实现。

平台通道是 Flutter 引擎提供的一套用于 Dart 和原生代码之间通信的异步消息系统。它主要有三种类型，可以满足不同的通信需求。

1. 方法通道（MethodChannel）
这是最常用、最核心的桥接方式。它允许你在 Dart 和原生代码之间双向地传递命名的方法调用和参数。

工作原理
Dart 端：你创建一个 MethodChannel 实例，并指定一个唯一的通道名称（这个名称在两端必须一致）。然后，你可以调用 invokeMethod() 来发起一个方法调用，并传递方法名和可选的参数。

原生端：你创建一个同样名称的 MethodChannel 实例，并设置一个 MethodCallHandler 来监听来自 Dart 的调用。当收到调用时，处理程序会根据方法名执行相应的原生代码。

结果返回：原生代码执行完毕后，通过 result.success() 或 result.error() 将结果返回给 Dart。在 Dart 端，invokeMethod() 会得到这个结果。

示例：获取电量信息
Dart 代码

Dart

import 'package:flutter/services.dart';

const platform = MethodChannel('com.example.app/battery');

Future<void> getBatteryLevel() async {
  try {
    // 调用原生方法 'getBatteryLevel'
    final int result = await platform.invokeMethod('getBatteryLevel');
    print('Battery level is $result %.');
  } on PlatformException catch (e) {
    print("Failed to get battery level: '${e.message}'.");
  }
}
Android (Kotlin)

Kotlin

// In MainActivity.kt
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

private const val CHANNEL = "com.example.app/battery"

override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
        .setMethodCallHandler { call, result ->
            if (call.method == "getBatteryLevel") {
                val batteryLevel = getBatteryLevel() // 你的原生方法
                if (batteryLevel != -1) {
                    result.success(batteryLevel)
                } else {
                    result.error("UNAVAILABLE", "Battery level not available.", null)
                }
            } else {
                result.notImplemented()
            }
        }
}
iOS (Swift)

Swift

// In AppDelegate.swift
import Flutter

private const val CHANNEL = "com.example.app/battery"

override func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    let controller = window.rootViewController as! FlutterViewController
    let batteryChannel = FlutterMethodChannel(name: CHANNEL, binaryMessenger: controller.binaryMessenger)
    
    batteryChannel.setMethodCallHandler { (call: FlutterMethodCall, result: @escaping FlutterResult) in
        if call.method == "getBatteryLevel" {
            let batteryLevel = getBatteryLevel() // 你的原生方法
            result(batteryLevel)
        } else {
            result(FlutterMethodNotImplemented)
        }
    }
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
}
2. 事件通道（EventChannel）
事件通道用于从原生平台向 Flutter 发送持续的、异步的数据流。这对于需要实时更新的场景非常有用，例如：

传感器数据（加速度计、陀螺仪）

网络连接状态变化

蓝牙设备扫描结果

工作原理
Dart 端：你创建一个 EventChannel 实例，并调用 receiveBroadcastStream() 来获取一个 Stream 对象。

原生端：你创建一个 EventChannel 并设置一个 StreamHandler。

数据流：当原生端有新数据时，它会通过 events.success() 将数据发送到 Dart，这些数据会以 Stream 的形式被 Dart 端的代码接收和处理。

3. 基础消息通道（BasicMessageChannel）
基础消息通道是最低级的平台通道，用于传递非结构化的、原始的二进制数据。它通常用于更复杂的自定义数据交换，或者作为实现其他通道的底层基础。

工作原理
Dart 端：使用 BasicMessageChannel 发送和接收二进制消息。

原生端：实现一个 MessageHandler 来处理收到的消息。

总结与选择
大多数情况：使用 MethodChannel。它是最简单、最常用的选择，适用于大多数调用原生功能的需求。

需要持续的数据流：使用 EventChannel。当你需要从原生向 Flutter 推送实时数据时，它是唯一合适的选择。

需要自定义数据格式或高级用法：使用 BasicMessageChannel。它提供了最大的灵活性，但通常不直接用于应用开发，而是作为插件开发的底层基础。

通过这些平台通道，Flutter 开发者可以充分利用原生平台的丰富功能，同时保持代码的整洁和跨平台特性。

## 学习资料

https://github.com/Solido/awesome-flutter?tab=readme-ov-file#frameworks