---
title: "使用aws时遇到的问题"
collection: talks
type: "talks"
permalink: /talks/aws_problem
date: 2025-06-28
toc: true # 启用目录
toc_label: "aws 问题及解决方案" # 目录标题
toc_sticky: true # 目录是否固定在侧边 (可选)
---


# aws iot core 遇到的问题

## 创建aws的AWS Signer

### 查看支持的平台类型

    aws signer list-signing-platforms --region  us-east-1
    
### 创建平台ID

    aws signer put-signing-profile --profile-name otaSign --platform-id AWSIoTDeviceManagement-SHA256-ECDSA --signing-material certificateArn=arn:aws:acm:us-east-1:561470034375:certificate/e20e0e41-8688-4801-8aa6-0406afec266e --region us-east-1
    注意 要选用第二种，AmazonFreeRTOS-Default的方式
    aws signer put-signing-profile --profile-name otaFreeRTOS --platform-id AmazonFreeRTOS-Default --signing-material certificateArn=arn:aws:acm:us-east-1:561470034375:certificate/e20e0e41-8688-4801-8aa6-0406afec266e --signing-parameters certname=/ --region us-east-1 

### 查看平台Id
 
    aws signer list-signing-profiles --region us-east-1
    aws signer get-signing-profile --profile-name otaSign --region us-east-1
    aws signer list-signing-profiles --region us-east-2
    aws signer list-signing-jobs --region us-east-1
    aws iot list-streams --max-results 10 --region us-east-1


## 自定义job实现OTA一样的功能

    我们来详细说明如何使用 AWS IoT 的自定义 Job 功能，并结合 MQTT 文件流来下发固件到设备，并提供相应的 Python 代码。

### 自定义 OTA Job 涉及的关键模块和流程
    当您选择使用自定义 Job 并通过 MQTT 文件流下发固件时，整个过程将比使用 create_ota_update 更灵活，但也需要您自行管理更多细节。

#### 涉及的关键模块：

##### 固件文件管理 (Amazon S3)：

    您的固件二进制文件将存储在 S3 桶中。这是 AWS IoT Stream 的数据源。
    S3 权限：需要确保您的 Job 角色有权限从 S3 桶中读取文件。

##### AWS IoT Stream (文件流服务)：
    
    这是 MQTT 文件传输的核心。您需要创建一个 AWS IoT Stream，它将 S3 中的固件文件“包装”起来，并通过 MQTT 提供给设备。
    权限：您的 Job 角色需要有 iot:CreateStream、iot:GetStream 等权限，并且 AWS IoT Stream 服务本身也需要有权限读取您 S3 桶中的固件文件。

##### AWS Code Signing (可选但强烈推荐)：
    
    为了确保固件的完整性和真实性，您应该对固件进行数字签名。
    AWS Code Signing (Signer) 可以为您完成此操作，生成签名数据，这些数据将包含在您的自定义 Job 文档中。
    权限：执行签名操作的用户或角色需要有 signer:StartSigningJob 等权限。

##### AWS IoT Jobs (任务管理)：
    
    这是协调整个 OTA 过程的服务。您将创建一个自定义 Job，并指定一个 S3 路径，该路径指向您自己编写的 Job 文档。
    Job 文档：这是一个 JSON 文件，它将包含 Stream ID、文件 ID、固件大小、校验和、签名信息以及任何您想传递的自定义参数。
    Job 角色 (IAM Role)：一个 IAM 角色，允许 AWS IoT 服务代入该角色来执行任务（例如：读取 Job 文档、创建 Stream、访问 S3）。这个角色的信任策略需要允许 iot.amazonaws.com 服务代入。

##### 设备端 OTA Agent (固件逻辑)：

    这是运行在设备上的软件，负责接收 Job 通知、解析 Job 文档、通过 MQTT 请求文件流数据块、重组文件、校验固件、执行更新和报告 Job 状态。
    MQTT 客户端：设备需要一个能够连接到 AWS IoT Core 并处理 MQTT 消息的客户端。
    OTA 逻辑：下载、验证、烧写固件的逻辑。
    
##### 流程概述

    准备固件：将固件文件上传到 S3。
    创建 Stream：使用 S3 中的固件文件作为源，创建 AWS IoT Stream。
    签名固件：使用 AWS Code Signing 对固件进行签名（可选）。
    构建自定义 Job 文档：创建一个 JSON 文档，其中包含 Stream ID、文件 ID、签名信息以及其他自定义参数。将此文档上传到 S3。
    创建自定义 Job：使用 boto3.client('iot').create_job() API 创建一个 Job，并将其 documentSource 参数指向您在 S3 上的自定义 Job 文档。
    设备执行：设备通过 MQTT 接收 Job 通知，请求 Job 文档，解析文档以获取 Stream 信息，然后通过 MQTT 文件流主题请求并接收固件数据块，最后执行更新。

## iot-core 证书权限策略设置注意问题

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:*",
      "Resource": [
        "arn:aws:iot:us-east-1:561470034375:thing/DH0050P000003",
        "arn:aws:iot:us-east-1:561470034375:client/DH0050P000003",
        "arn:aws:iot:us-east-1:561470034375:topic/$aws/things/DH0050P000003/shadow/*",
        "arn:aws:iot:us-east-1:561470034375:topic/xsense/things/DH0050P000003/shadow/*",
        "arn:aws:iot:us-east-1:561470034375:topic/xsense/things/DH0050P000003/event/name/aaa",
        "arn:aws:iot:us-east-1:561470034375:topic/$aws/things/DH0050P000003/jobs/*",
        "arn:aws:iot:us-east-1:561470034375:topic/$aws/things/DH0050P000003/streams/*",
        "arn:aws:iot:us-east-1:561470034375:topicfilter/$aws/things/DH0050P000003/shadow/*",
        "arn:aws:iot:us-east-1:561470034375:topicfilter/xsense/things/DH0050P000003/shadow/*",
        "arn:aws:iot:us-east-1:561470034375:topicfilter/xsense/things/DH0050P000003/event/name/aaa",
        "arn:aws:iot:us-east-1:561470034375:topicfilter/$aws/things/DH0050P000003/jobs/*",
        "arn:aws:iot:us-east-1:561470034375:topicfilter/$aws/things/DH0050P000003/streams/*"
      ]
    }
  ]
}