---
title: "rebac权限系统"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/rebac
date: 2025-06-28
toc: true # 启用目录
toc_label: "aws 问题及解决方案" # 目录标题
toc_sticky: true # 目录是否固定在侧边 (可选)
---

# ReBAC 概述

    ReBAC基于关系的访问控制，是一种先进的授权模型。它通过分析实体之间存在的各种关系来决定用户是否拥有对特定资源的访问权限。相较于传统的 RBAC（基于角色的访问控制）和 ABAC（基于属性的访问控制），ReBAC 提供了更高的灵活性和表达能力，尤其适用于拥有复杂、动态实体关系的应用场景，例如社交网络、协同办公、以及物联网 (IoT) 平台。说到ReBAC就离不开Google的Google Zanzibar论文。

# Google Zanzibar 概述

    Google Zanzibar 是 Google 内部构建的、用于处理全球分布式授权的一致、大规模的系统。它在 2019 年通过一篇同名学术论文首次对外披露，立即在业界引起了广泛关注，并成为了 ReBAC (Relationship-Based Access Control) 模型在实际生产环境中应用的典范。

## 基于关系的授权 (ReBAC 的实践)

    核心数据模型： Zanzibar 的权限核心是存储关系元组 (Relationship Tuples)。这些元组清晰地定义了实体之间的关系，例如 object#relation@subject。

        示例： document:report123#editor@user:alice (用户 Alice 是文档 report123 的编辑者)

        示例： folder:project_alpha#viewer@group:engineering_team (工程团队是项目文件夹 project_alpha 的查看者)

        示例： group:engineering_team#member@user:bob (用户 Bob 是工程团队的成员)

    图结构表示： 所有这些关系元组在内部被表示为一个巨大的有向图。授权检查本质上是查询这个图，寻找从请求者到目标资源的路径或关系，从而推导权限。

    复杂关系推理： Zanzibar 不仅仅是简单的关系匹配，它还能进行复杂的嵌套和推导，例如，如果用户 Bob 是工程团队的成员，而工程团队是项目文件夹的查看者，那么 Bob 自然也就拥有查看该文件夹内所有内容的权限。

## 强大的数据一致性

    外部一致性： 这是 Zanzibar 最显著的特点之一。它确保授权决策能尊重用户操作的因果顺序。这意味着，当用户修改权限（如分享一个文件）后，紧随其后的授权检查（如另一个用户尝试访问这个文件）会立即反映最新的权限状态，即使在分布式系统中数据传播存在延迟。
    Spanner 的支撑： Zanzibar 的数据存储层依赖于 Google 的全球分布式数据库 Spanner，利用其强大的全球原子钟和 TrueTime 技术，实现了跨区域的强一致性保证。
    “Zookie”机制： 为了在不牺牲性能的情况下保持一致性，Zanzibar 引入了一个概念叫做 "Zookie"。这是一个不透明的数据结构，包含了最近一次权限更新的时间戳。客户端在发起授权检查时可以带上这个 Zookie，确保授权决策是基于该时间戳或更新后的数据来做出的，从而避免了读取到过期数据。

## Zanzibar的重要性

    Zanzibar 的论文不仅展示了 Google 在构建大规模分布式系统方面的卓越工程能力，更重要的是，它为业界提供了一个构建高度一致、
    可扩展且灵活的授权系统的蓝图。它验证了 ReBAC 模型在面对海量用户和复杂关系时的可行性与强大性，并成为了许多开源项目（如 OpenFGA、AuthZed SpiceDB）和商业授权服务的设计灵感来源。它推动了整个行业对授权系统设计理念的思考和实践。



# ReBAC具体实现

## aws 相关描述

https://aws.amazon.com/es/blogs/database/graph-powered-authorization-relationship-based-access-control-for-access-management/

aws的 AVP服务即实现了ReBAC


## 具体开源实现

https://github.com/authzed/spicedb

https://openfga.dev/

https://github.com/ory/keto

https://authzed.com/

https://github.com/Permify/permify