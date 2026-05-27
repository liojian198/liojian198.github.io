---
title: "iot 鉴权，通讯加密"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/iot_author
date: 2026-05-27 09:50:00
---

# 简介

  设备连接后鉴权及权限控制，这个iot领域很常见的，有x.509,一机一密等方式。

# 一机一密

## 传输过程中带密钥的方式（不使用）

## 传输过程中不带密钥的方式

  在物联网（IoT）场景中，利用 HMAC-SHA256 实现“一机一密”的设备鉴权，是一种非常经典、轻量级且对 MCU 极其友好的方案。它不需要设备跑复杂的非对称加密算法（如 RSA/ECC），也不需要消耗大量内存去维护证书链。

  其核心思想是：云端掌握一个全局的“根密钥（ProductSecret）”，每台设备只保存通过自身唯一标识（DeviceName）与根密钥计算出来的“设备密钥（DeviceSecret）”。设备登录时，不发送密钥本身，而是发送用该密钥计算出的 HMAC-SHA256 签名。
  
  下面是完整的方案设计和交互流程：

### 1. 核心三要素

  在“一机一密”的 HMAC 鉴权体系中，每台设备需要具备三个参数：

  ProductKey（产品唯一标识，全系统唯一）
  
  DeviceName（设备唯一标识，通常是芯片的 Mac 地址或 Chip ID）
  
  DeviceSecret（设备密钥，每台设备各不相同，绝对保密）

### 2. 密钥派生与烧录（产线阶段）
  
  “一机一密”的密码不能由人工随机瞎编，而是通过统一的算法派生出来的。

  云端/产线生成：云端持有一个高强度的 ProductSecret（产品根密钥）。
  
  当工厂要生产一批设备时，产线系统通过算法为每台设备计算出专属密钥：$$DeviceSecret = \text{HMAC-SHA256}(\text{ProductSecret}, \text{DeviceName})$$

  烧录：工厂将 ProductKey、DeviceName 和计算出来的 DeviceSecret 烧录进设备的安全存储区（如 Flash 易失性加密区或 OTP 区域）。

### 3. 设备登录鉴权流程（运行阶段）

  当设备上电连接服务器（例如通过 MQTT 或 HTTP 登录）时，鉴权流程如下：

  第一步：设备端生成签名 (Sign)
  
  设备不能直接把 DeviceSecret 发给服务器（防止网络抓包泄露）。它需要利用当前的时间戳或随机数来生成一个动态签名。

  设备获取当前时间戳 Timestamp（例如：1716768000）或生成一个随机数 ClientNonce。

  设备将需要提交的信息拼接成一个明文字符串（Content），例如：

``` text

  clientId=mac123456&deviceName=device_001&productKey=pk_prod&timestamp=1716768000

```

  设备使用本地的 DeviceSecret 作为 Key，对该字符串计算 HMAC-SHA256，
  得到一个 64位的十六进制签名字符串（Signature）：$$\text{Signature} = \text{HMAC-SHA256}(DeviceSecret, \text{Content})$$

### 第二步：设备发送登录请求

  设备通过网络向服务器发送登录报文，报文中包含：

  明文信息：ProductKey、DeviceName、Timestamp

  签名密文：Signature

### 服务端验签与鉴权

  服务端收到请求后，执行以下校验流程：

  1. 时钟校验（可选但强烈推荐）：对比收到的 Timestamp 与服务器当前时间，如果相差超过 5 分钟，直接拒绝（防止重放攻击）。

  2. 查找/计算 DeviceSecret：

  方法 A（查库）：服务端根据明文 ProductKey 和 DeviceName，从数据库或 Redis 中直接查出该设备的 DeviceSecret。

  方法 B（实时派生，无状态）：服务端内存中常驻 ProductSecret，直接现场用同样的算法算一下：$\text{HMAC-SHA256}(ProductSecret, DeviceName)$，
        瞬间得到该设备的 DeviceSecret。（这种方法极具扩展性，服务器不需要存几百万台设备的密钥）。
  
  3. 本地计算签名：服务端用拿到的 DeviceSecret 和设备传过来的明文参数（Timestamp等），按照设备端一模一样的格式拼接，在本地也算一遍 HMAC-SHA256。

  4.比对结果：对比服务端算出来的签名和设备传过来的 Signature。
    
    完全一致 $\rightarrow$ 鉴权通过，允许登录，建立 Session。

    不一致 $\rightarrow$ 鉴权失败，断开连接。

### 4. 为什么这个方案足够安全？

  密钥没有网上传输：网络中只有明文的设备名、时间戳和一串签名。黑客截获了签名，由于 HMAC-SHA256 的不可逆性，他也无法反推出 DeviceSecret。

  防重放攻击（Replay Attack）：因为签名中绑定了 Timestamp（或随机数）。黑客如果录制了这次的登录包，5分钟后再原封不动地发给服务器，服务器会因为时间戳过期而直接拒绝。

  一机一密，隔离风险：每台设备的 DeviceSecret 都是独立的。如果黑客通过物理手段锯开芯片窃取了其中一台设备的 DeviceSecret，他也只能仿冒这一台设备，无法影响其他任何设备，更无法逆推出云端的 ProductSecret。

### 设备端伪代码

``` C

// 1. 准备基础数据
char* device_secret = "设备专属的Secret_xxxxxx"; // 从安全Flash中读取
char* content = "clientId=001&deviceName=dev_01&timestamp=1716768000";

// 2. 调用硬件或 MbedTLS 库计算 HMAC
uint8_t hmac_result[32];
mbedtls_md_hmac(
    mbedtls_md_info_from_type(MBEDTLS_MD_SHA256), 
    (const unsigned char*)device_secret, strlen(device_secret), 
    (const unsigned char*)content, strlen(content), 
    hmac_result
);

// 3. 转成 Hex 字符串作为 Password 发送登录
char sign_str[65];
for(int i = 0; i < 32; i++) {
    sprintf(&sign_str[i*2], "%02x", hmac_result[i]);
}
// 最终将 sign_str 作为 MQTT 的 Password，DeviceName 作为 Username 发起连接

```

