---
title: "Web_Hardware_APIs 简介"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/Web_Hardware_APIs
date: 2026-01-08 15:36:12
---

# Web_Hardware_APIs简介

  Web Hardware APIs 是一系列允许网页程序突破浏览器“沙箱”限制，直接与物理世界硬件设备进行数据交换的技术。
  这些 API 的核心价值在于：无需下载安装任何驱动或桌面软件，只需一个 URL 即可控制硬件。

# API 目前有以下几大类

## 1. 串口通信：Web Serial API

  这是目前在工控、医疗和创客领域应用最广的 API。它模拟了传统的 RS-232 或 TTL 串口通信。

  对接设备： Arduino, ESP32, 扫码枪, 热敏打印机, 称重仪表, PLC。

  物理连接： 通常通过 USB 线（设备端带 USB 转串口芯片，如 CH340 或 CP2102）。

  通信模式： 基于字节流（Streams），可以设置波特率（Baud rate）、停止位等参数。

## 原始 USB 通信：WebUSB API
  
  WebUSB 允许网页直接与 USB 设备的端点（Endpoints）进行对话，绕过操作系统标准驱动。

  对接设备： 3D 打印机、示波器、逻辑分析仪、自定义固件刷写工具（如手机刷机工具）。

  核心优势： 它是“免驱动”的终极方案。硬件厂商只需编写网页逻辑，用户插上设备后，网页就能直接识别并操作。

  限制： 浏览器严禁 WebUSB 访问鼠标、键盘、耳机等受系统内核保护的“标准类”设备（这些设备由 WebHID 负责）。

## 人机交互设备：WebHID API

  HID（Human Interface Device）是一个特殊的协议类，旨在简化低速输入输出设备。

  对接设备： 游戏手柄、具有自定义宏的机械键盘、绘图板、带接听键的电话耳机。

  功能： 支持读取 Input Reports（如按键按下）和发送 Output Reports（如让手柄震动或让键盘灯亮起）。

## 蓝牙低功耗：Web Bluetooth API

  这是目前唯一的官方 Web 无线硬件连接方案，基于 BLE (Bluetooth Low Energy)。

  对接设备： 运动手环、心率计、蓝牙温湿度计、智能灯泡。

  通信架构： 基于 GATT（通用属性配置文件）。网页作为中心设备（Central），通过寻找 Service（服务）和 Characteristic（特征值）来读写数据。

## 近场通信：Web NFC API

  主要针对带有 NFC 模块的移动端浏览器（如 Android 上的 Chrome）。

  对接设备： NTAG 系列标签、智能海报、门禁卡（未加密数据）。

  功能： 支持读取标签序列号（UID）以及读写 NDEF 格式的消息。

# 蓝牙低功耗：Web Bluetooth API

  Web Bluetooth API 允许浏览器通过 蓝牙低功耗 (Bluetooth Low Energy, BLE) 与附近的硬件设备直接通信。它使得开发者可以编写网页来控制智能灯泡、读取心率传感器或配置物联网设备，而无需安装原生应用。

## 核心协议基础：GATT

  Web Bluetooth API 基于 GATT (Generic Attribute Profile) 协议。要理解该 API，必须理解其层级结构：

  Server (设备): 蓝牙外设（如手环），它持有一个包含数据的数据库。

  Service (服务): 功能模块。例如，“电池服务”或“心率服务”。每个服务由一个唯一的 UUID 标识。

  Characteristic (特征值): 最小的数据单元。例如，“当前电量百分比”。它可以支持读、写、订阅（Notification）。

## 交互工作流

  由于安全原因，Web Bluetooth 的流程非常严格，必须由用户交互触发。

### 第一步：设备扫描与请求 (Discovery)

  浏览器弹出系统对话框，让用户选择设备。开发者可以通过过滤器（Filters）限制显示的设备类型。

``` JavaScript

const device = await navigator.bluetooth.requestDevice({
  filters: [{ services: ['heart_rate'] }], // 只显示提供心率服务的设备
  optionalServices: ['battery_service']   // 额外允许访问的服务
});

```

### 建立连接 (Connection)

连接到设备的 GATT 服务器。

``` JavaScript

const server = await device.gatt.connect();

```

### 获取服务与特征值 (Access)

定位到具体的控制点。

``` JavaScript

  const service = await server.getPrimaryService('heart_rate');
const characteristic = await service.getCharacteristic('heart_rate_measurement');

``` 

### 数据操作 (Data Exchange)

  读取： await characteristic.readValue();

  写入： await characteristic.writeValue(new Uint8Array([0x01]));

  通知： 实时监听数据变化（如心率实时跳动）。

## 安全与限制 (必须注意)

  Web Bluetooth 是浏览器中权限最高、限制最严的 API 之一：

  HTTPS 强制： 必须在安全上下文（HTTPS 或 localhost）下运行。

  用户手势触发： 必须由点击、触摸等用户动作调用 requestDevice()，不能在页面加载时自动运行。

  黑名单机制： 浏览器禁用了部分敏感服务（如 HID 键盘、GAP 服务），以防止攻击者通过网页劫持用户的输入。

  平台兼容性：

    支持较好： Android 上的 Chrome, macOS 上的 Chrome/Edge, Windows 10+。

    基本不支持： iOS 上的所有浏览器（受苹果 WebKit 策略限制），以及 Linux 上的 Firefox。

## 典型应用场景

运动健身： 网页端直接同步跑步机、心率带数据。

工业配置： 通过手机浏览器无线调试 PLC 或传感器参数，无需扫码下载 App。

教育硬件： 配合 Web 编程平台（如 Scratch）控制乐高、Arduino 机器人。

## 调试工具
  
Chrome 内部控制台： 在地址栏输入 chrome://bluetooth-internals/，可以查看当前连接的详细 GATT 结构、信号强度（RSSI）和原始报文，这是排查连接问题的核心工具。
