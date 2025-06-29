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

    ReBAC基于关系的访问控制，是一种先进的授权模型。它通过分析实体之间存在的各种关系来决定用
    户是否拥有对特定资源的访问权限。相较于传统的 RBAC（基于角色的访问控制）和 ABAC（基于属
    性的访问控制），ReBAC 提供了更高的灵活性和表达能力，尤其适用于拥有复杂、动态实体关系的应
    用场景，例如社交网络、协同办公、以及物联网 (IoT) 平台。说到ReBAC就离不开Google的Google
     Zanzibar论文。

    论文地址：https://www.usenix.org/system/files/atc19-pang.pdf

# Google Zanzibar 概述

    Google Zanzibar 是 Google 内部构建的、用于处理全球分布式授权的一致、大规模的系统。
    它在 2019 年通过一篇同名学术论文首次对外披露，立即在业界引起了广泛关注，
    并成为了 ReBAC (Relationship-Based Access Control) 模型在实际生产环境中应用的典范。

## 基于关系的授权 (ReBAC 的实践)

    核心数据模型： Zanzibar 的权限核心是存储关系元组 (Relationship Tuples)。
    这些元组清晰地定义了实体之间的关系，例如 object#relation@subject。

        示例： document:report123#editor@user:alice (用户 Alice 是文档 report123 的编辑者)

        示例： folder:project_alpha#viewer@group:engineering_team (工程团队是项目文件夹 project_alpha 的查看者)

        示例： group:engineering_team#member@user:bob (用户 Bob 是工程团队的成员)

    图结构表示： 所有这些关系元组在内部被表示为一个巨大的有向图。授权检查本质上是查询这个图，
    寻找从请求者到目标资源的路径或关系，从而推导权限。

    复杂关系推理： Zanzibar 不仅仅是简单的关系匹配，它还能进行复杂的嵌套和推导，
    例如，如果用户 Bob 是工程团队的成员，而工程团队是项目文件夹的查看者，
    那么 Bob 自然也就拥有查看该文件夹内所有内容的权限。

## 强大的数据一致性

    外部一致性： 这是 Zanzibar 最显著的特点之一。它确保授权决策能尊重用户操作的因果顺序。
    这意味着，当用户修改权限（如分享一个文件）后，紧随其后的授权检查
    （如另一个用户尝试访问这个文件）会立即反映最新的权限状态，即使在分布式系统中数据传播存在延迟。
    Spanner 的支撑： Zanzibar 的数据存储层依赖于 Google 的全球分布式数据库 Spanner，
    利用其强大的全球原子钟和 TrueTime 技术，实现了跨区域的强一致性保证。
    “Zookie”机制： 为了在不牺牲性能的情况下保持一致性，Zanzibar 引入了一个概念叫做 "Zookie"。
    这是一个不透明的数据结构，包含了最近一次权限更新的时间戳。客户端在发起授权检查时可以带上这个 Zookie，
    确保授权决策是基于该时间戳或更新后的数据来做出的，从而避免了读取到过期数据。

## Zanzibar的重要性

    Zanzibar 的论文不仅展示了 Google 在构建大规模分布式系统方面的卓越工程能力，
    更重要的是，它为业界提供了一个构建高度一致、
    可扩展且灵活的授权系统的蓝图。它验证了 ReBAC 模型在面对海量用户和复杂关系时的可行性与强大性，
    并成为了许多开源项目（如 OpenFGA、AuthZed SpiceDB）和商业授权服务的设计灵感来源。
    它推动了整个行业对授权系统设计理念的思考和实践。


# ReBAC 建模方式

## 一、选择最重要的功能(权限相关的用户故事)

    故事：
        1. 如果用户是drive的所有者，他们可以在drive中创建文档。
        2. 如果用户是drive的所有者，他们可以在drive中创建文件夹。
        3. 用户如果是文件夹的所有者，可以在文件夹中创建文档。该文件夹是文档的父级。
        4. 用户如果是文件夹的所有者，可以在文件夹中创建子文件夹。已存在的文件夹是新文件夹的父级。
        5. 如果用户是文档的所有者或编辑者，或者用户是文档父级文件夹/drive的所有者，
            用户可以将文档共享给其他用户或组织，并指定为编辑者或查看者。
        6. 如果用户是文件夹的所有者，他们可以将文件夹共享给其他用户或组织，并设置为查看者。
        7. 如果用户是文档的所有者、查看者或编辑者，或者他们是该文档所属文件夹/drive的查看者
            或所有者，则可以查看该文档。
        8. 如果用户是文档的所有者或编辑者，或者他们是文档所在文件夹/drive的所有者，则可以编辑文档。
        9. 如果用户是文档的所有者，他们可以更改文档的所有者。
        10. 用户如果是文件夹的所有者，可以更改文件夹的所有者。
        11. 用户可以是组织的成员。
        12. 用户如果是文件夹的所有者，或者该文件夹的父文件夹或父drive的查看者或所有者，则可以查看该文件夹。

## 二、列出对象类型

    文档，文件夹， 组织， 用户，Drive 

## 三、为这些类型列出关系

### 文档
    parent  父级
    can_share  可共享
    owner  所有者
    editor  编辑器
    can_write  可写
    can_view  可查看
    viewer  查看者
    can_change_owner  可以更改所有者

### 文件夹

    can_create_document  可以创建文档
    owner  所有者
    can_create_folder
    can_view  可查看
    viewer  查看者
    parent  父级

### 组织

    member  成员

### Drive 

    can_create_document  可以创建文档
    owner  所有者
    can_create_folder    

## 四、定义关系

    type user

    type organization
    relations
        define member: [user, organization#member]

    type document
    relations
        define owner: [user, organization#member]
        define editor: [user, organization#member]
        define viewer: [user, organization#member]
        define parent: [folder]
        define can_share: owner or editor or owner from parent
        define can_view: viewer or editor or owner or viewer from parent or editor from parent or owner from parent
        define can_write: editor or owner or owner from parent
        define can_change_owner: owner

## 五、测试模型

### 编写关系元组

    { user:"user:anne", relation: "member", object: "organization:contoso"}
    { user:"user:beth", relation: "member", object: "organization:fabrikam"}
    { user:"user:anne", relation: "owner", object: "document:1"}
    { user:"organization:fabrikam#member", relation: "editor", object: "document:1"}
    { user:"user:beth", relation: "owner", object: "document:2"}
    { user:"organization:contoso#member", relation: "viewer", object: "document:2"}


### 创建断言

    user anne has relation can_share with document:1
    user anne has relation can_write with document:1

    user beth has relation can_share with document:1

## 六、迭代



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