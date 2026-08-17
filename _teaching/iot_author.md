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

# aws IOT 批量设备通信安全处理

  aws fleet provisioning

  AWS IoT Fleet Provisioning（舰队批量配置/自动上线服务） 是 AWS IoT Core 提供的一项核心功能，旨在解决物联网设备在大规模量产和部署时的数字身份安全注入与云端注册难题。  在传统的物联网项目中，为成千上万台设备手动烧录独一无二的 X.509 证书、私钥并在云端注册（Thing、Policy、Groups）是一场工程噩梦。Fleet Provisioning 实现了这一过程的完全自动化。

##  Fleet Provisioning 解决了什么痛点？

      避免产线证书管理混乱： 过去需要在工厂产线为每台设备烧录不同的唯一证书，稍有不慎就会导致证书重复或泄露。
      零接触部署（Zero-Touch Provisioning）： 设备在出厂时只需要烧录一个通用的“认领证书（Claim Certificate / Bootstrap Certificate）”。
      当设备第一次联网时，它会自动向云端申请属于自己的唯一生产证书和设备身份。
      安全与权限解耦： 限制通用的 Claim 证书权限，使其只能访问注册相关的特定 MQTT 主题，防止安全漏洞。  

## 核心工作流程（以“Claim 认领机制”为例）

  当一台全新的设备第一次通电联网时，整个自动注册流程如下：

### 1.引导连接（Bootstrap Connection）：

  设备使用内置的 Claim 证书（一个所有同类设备可共用的临时低权限证书）连接到 AWS IoT Core。

  该 Claim 证书关联的 IoT 策略（Policy）极其严格，只能用于执行 Fleet Provisioning 相关的 API 话题。

### 2.提交参数与凭证申请：

  设备通过 MQTT 向 AWS 发送请求，附带自身的硬件唯一标识（如序列号、MAC 地址、或通过 CSR 密钥签名请求）。

### 3.Lambda 钩子验证（Pre-provisioning Hook）：

    AWS 收到请求后，会触发一个 AWS Lambda 函数。

    你可以在该 Lambda 中编写业务逻辑：去数据库（如 DynamoDB）核对设备的序列号是否合法。如果合法，允许注册；如果是伪造设备，直接拒绝。

### 4.自动创建云端资源（Provisioning Template）：  

      验证通过后，AWS 根据预先定义好的 Provisioning Template（配置模板） 
      自动在云端执行以下操作：为该设备生成或注册一个唯一的 X.509 生产证书。 
      在 AWS IoT 注册表中创建 Thing（物模型实体）。 
      将 Thing 自动分配到指定的 Thing Groups（设备分组）。 
      绑定正式的高权限生产策略（Production Policy）。

### 5. 凭证下发与切换：

  AWS IoT 将新生成的证书、私钥以及“所有权证明令牌”通过 MQTT 下发给设备。

  设备安全地将新证书保存在本地（如 Secure Element / Flash 安全区）。

  设备断开临时连接，随后使用崭新的“唯一生产身份”重新连接 AWS IoT Core，正式进入日常业务运营状态。

## 核心概念组成

  要玩转 Fleet Provisioning，主要涉及三个核心构件：
  
  Claim Certificate（认领证书）： 烧录在批量设备中的“通行证”，生命周期可控，通常权限受限。
  
  Provisioning Template（配置模板）： 一个 JSON 格式的蓝图，规定了设备接入后要在云端自动创建哪些 AWS 资源（Thing、Certificate、Policy）以及采用什么命名规则。 
  
  Pre-provisioning Hook（前置钩子 Lambda）： 决定设备能否入网的“安检门”，用于对接企业自身的 ERP、CRM 或设备白名单数据库进行安全校验。

## 两种主流的 Fleet Provisioning 场景

  1. Provisioning by Claim（认领配置 - 工业/无屏设备常用）：
设备出厂自带通用的引导证书，上电自动联网注册。适合大批量、无交互界面的工业传感器、网关。

2. Provisioning by Trusted User（信任用户/App 配置 - 消费级智能家居常用）：
设备本身没有证书，用户买回家后，通过手机 App（如配网助手）扫描设备的二维码。手机 App 充当“受信任的代理”，在短时间内向 AWS 申请并为该设备注入唯一的身份证书。


  
