---
title: "ontology(本体论)"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/ontology
date: 2026-08-17 02:01:00
---

# 简介

  以前使用图建模来设计东西，发现会很方便很爽， 几年前， 被问的最多的是， 微信公众号，微博，怎么设计表，因为有人之间的点评，点赞，关联等一系列反关系型数据建模，当时一群逗逼就往这个方向设计，
  我当时用图建模来设计，分分钟解决，效率和速度很快，那个时候他们听过图数据库的很少，感觉关系型就是一切，后面有redis了。又有一种新型的了，哈哈。 那个时候和某推送的技术负责人聊这个， 他只知道
  表越设计越难维护，没有换思路解决。哈哈， 不吐槽了， 每个人有自己的认知， 业务赚钱并不是技术有多厉害，可能技术很low。

# 使用经历

## 最近玩的ReBAC 权限管理的实现

## 金融风控，助贷方向

## ai语音陪聊记忆的实现

## 知识图谱的构建

## 零信任网路

# ontology 简介

  核心定义的四个关键词
  
    这个经典定义（由信息学家 Tom Gruber 提出，后经 R. Studer 等人完善）包含了四个关键支柱：

    概念模型（Conceptualization）：

      指的是对客观世界中某种现象的抽象简化视角。它通过识别该领域中的对象、概念以及它们之间的关系，来决定“在这个世界里什么东西是存在的”。

    明确的（Explicit）：

      本体中所包含的概念、属性、约束和限制条件，都必须用明确的类型和定义显式地表达出来，不能含糊其辞或依赖人类的隐性默契。

    形式化的（Formal）：

      这是计算机能够理解本体的基石。形式化意味着它必须采用数学和逻辑语言（如描述逻辑 Description Logics）来表达，从而让计算机能够进行无歧义的解析和自动化推理（Reasoning）。

    共享的（Shared）：

      本体不是某一个人的私有定义，而是反映了一个特定领域或社区（Community）内共识的知识体系。

## Ontology 在现代计算系统中的三层内涵

  在实际技术落地中，一个完整的 Ontology 通常承载了以下三个层次的定义：
  
    1. 词汇表与分类体系（Taxonomy & Vocabulary）：规定了领域内有哪些“类”（Classes，例如：员工、部门、项目），以及类与类之间的层级关系（例如：程序员 是 员工 的子类）。
    2. 属性与实例约束（Attributes & Relations）：定义了类拥有什么属性（例如：员工 具有 入职时间、薪资 属性），以及类与类之间的关联（例如：员工 “属于” 部门）。
    3. 逻辑公理与约束规则（Axioms & Rules）：这是本体比普通数据库 Schema 或表格高级的地方。它通过逻辑规则限制数据的合法性并推导出新知识。例如：互斥规则： “全职员工”和“实习生”两个类不能有交集。推理规则： 如果 $A$ 是 $B$ 的主管，且 $B$ 是 $C$ 的主管，那么可以通过推理得出 $A$ 是 $C$ 的更高阶管理者。

## Ontology 企业的ReBAC 的具体例子

  企业级应用中，将 Ontology（本体论） 与 ReBAC（基于关系的访问控制） 结合，能够优雅地解决那些传统 RBAC（角色权限）或 ABAC（属性权限）根本无法应对的复杂、多跳、动态的权限控制难题。

以下通过一个典型的大型跨国企业“多级研发与知识产权（IP）资产管理系统”，来展示 Ontology 驱动的 ReBAC 具体是如何运作的。

### 业务背景：为什么传统权限方案失效了？

  在一个拥有数万人的跨国科技公司中：

如果用 RBAC（基于角色）：公司有上百个项目，每个项目都要配一堆“项目A经理”、“项目B研发”、“项目C审计”等角色，角色爆炸，根本无法管理。

如果用 ABAC（基于属性）：写一堆诸如 if user.department == doc.department and user.level >= doc.secret_level 的硬编码逻辑，一旦组织架构调整或引入“借调”、“外部专家顾问”等业务场景，代码就会陷入无限的 if-else 地狱。

而采用 Ontology + ReBAC 后，系统会将企业的组织架构、业务项目、资产文件抽象为一张统一的语义本体图谱，权限判断变成了“在图上寻找一条合法的有向路径”。

### 第一步：定义企业级 ReBAC 本体（Schema & 关系）

  在本体系统中，我们定义以下核心实体（Classes）和边（Relationships）：

  实体类（Classes）：

    Employee（员工）

    Department（部门）

    Project（项目）

    Document（敏感设计文档/源码资产）

    ExternalPartner（外部合作伙伴/顾问）

    核心业务关系（Object Properties）：

    belongsTo：Employee $\rightarrow$ Department（员工属于某部门）
    
    manages：Employee $\rightarrow$ Department（员工管理某部门）
    
    assignTo：Employee $\rightarrow$ Project（员工被分配到某项目）
    
    produces：Project $\rightarrow$ Document（项目产出了某文档）
    
    sharesWith：Project $\rightarrow$ ExternalPartner（项目将资料共享给外部伙伴）
    
    confidentialityLevel：Document 的属性（如：Public, Internal, TopSecret）

### 第二步：定义 ReBAC 访问控制规则（Ontology Axioms / Rules）

  在本体中，我们编写通用的业务逻辑规则（可以用 SWRL、图查询逻辑或策略规则表达），而不是针对具体的某个人写权限：

  规则 1：项目团队成员访问规则（基础路径）

    “如果员工 $E$ 被分配到了项目 $P$（assignTo），且文档 $D$ 是由项目 $P$ 产出的（produces），那么员工 $E$ 拥有对该文档的 Read（读取）权限。”

  规则 2：管理链继承规则（多跳推理）

    “如果经理 $M$ 管理（manages）部门 $D_1$，而员工 $E$ 属于（belongsTo）部门 $D_1$（或其子部门），且该员工拥有某文档的权限，则经理 $M$ 自动继承对该文档的 Review（审核）权限。”

  规则 3：跨界借调/外部顾问动态规则

    “如果项目 $P$ 与外部合伙人 $EP$ 建立了共享关系（sharesWith），且该文档的机密等级不属于 TopSecret，则该外部合伙人临时获得 Read 权限。”

### 第三步：具体场景与动态裁决（Execution）

  假设发生了一个真实的访问请求：

  请求主体： 工程师“张三”试图访问核心资产文档 Doc-007。
  传统数据库/代码的做法： 需要去查张三属于哪个项目、Doc-007 属于哪个项目、中间有没有特批白名单、是不是上级领导……写一堆复杂的 SQL JOIN。
  Ontology ReBAC 的做法：系统将请求丢给本体引擎（或通过图数据库路径匹配），引擎在图谱中检索从 Employee:张三 到 Document:Doc-007 之间是否存在符合规则的通路：
  引擎发现：张三 (assignTo) $\rightarrow$ Project-X (produces) $\rightarrow$ Doc-007。
  路径打通！规则 1 匹配成功。
  动态裁决结果：允许访问（Allow）。

### 这种模式的降维打击优势在哪里？

组织架构大调整时，代码零修改：
  如果张三被调到了另一个部门，或者项目负责人换了，底层数据图谱中的 belongsTo 或 assignTo 边发生了变化。权限系统会自动生效，因为权限是随着“关系”实时计算出来的，根本不需要去改权限控制代码。

完美支持“动态临时授权”：
如果某天需要让外部专家临时看一眼代码，只需要在图谱里临时建立一条 sharesWith 的边，访问控制立刻开通，到期删掉边权限自动回收。

消除大模型（AI Agent）的安全越权：
在现代企业引入 AI Agent（如帮员工查找公司内部资料的 Copilot）时，Ontology ReBAC 是最强力的安全护栏。
Agent 在调用知识库检索（GraphRAG）前，必须先通过本体 ReBAC 过滤：“这个 Agent 代表的用户在 ontology 图谱里有没有到达目标文档的合法路径？” 从而彻底防止 AI 泄露机密数据。


### 基于surrealDB 实现上述的 Ontology 企业的ReBAC 的具体例子

  基于surrealDB 实现上述的 Ontology 企业的ReBAC 的具体例子

#### 第一步：定义本体架构与关系（Schema & Ontology）

在 SurrealDB 中，我们直接用 SCHEMAFULL 模式定义类（Tables）和带类型的语义边（Relations）：  

``` text

  -- 1. 定义核心实体类 (Classes)
DEFINE TABLE employee SCHEMAFULL;
DEFINE FIELD name ON employee TYPE string;

DEFINE TABLE department SCHEMAFULL;
DEFINE FIELD name ON department TYPE string;

DEFINE TABLE project SCHEMAFULL;
DEFINE FIELD name ON project TYPE string;

DEFINE TABLE document SCHEMAFULL;
DEFINE FIELD title ON document TYPE string;
DEFINE FIELD confidentiality_level ON document TYPE string 
    ASSERT $value INSIDE ['Public', 'Internal', 'TopSecret'];

DEFINE TABLE external_partner SCHEMAFULL;
DEFINE FIELD name ON external_partner TYPE string;


-- 2. 定义具有业务语义的图关系边 (Relations)
-- 员工属于部门
DEFINE TABLE belongs_to SCHEMAFULL TYPE RELATION FROM employee TO department;

-- 员工管理部门
DEFINE TABLE manages SCHEMAFULL TYPE RELATION FROM employee TO department;

-- 员工被分配到项目
DEFINE TABLE assign_to SCHEMAFULL TYPE RELATION FROM employee TO project;

-- 项目产出文档
DEFINE TABLE produces SCHEMAFULL TYPE RELATION FROM project TO document;

-- 项目与外部伙伴共享
DEFINE TABLE shares_with SCHEMAFULL TYPE RELATION FROM project TO external_partner;

```


#### 第二步：注入本体实例数据（Instantiation）

模拟企业真实世界中的组织、项目、文档及其拓扑关系：

``` text

-- 创建员工
CREATE employee:zhang_san SET name = "张三 (研发工程师)";
CREATE employee:li_si SET name = "李四 (部门总监)";
CREATE employee:partner_bob SET name = "Bob (外部顾问)";

-- 创建部门与项目
CREATE department:ai_dept SET name = "AI 研发部";
CREATE project:project_x SET name = "大模型底座项目";

-- 创建敏感文档
CREATE document:doc_007 SET title = "核心架构源码与权重文件", confidentiality_level = "TopSecret";
CREATE document:doc_008 SET title = "Q3 项目推进报告", confidentiality_level = "Internal";

-- 建立业务图谱连接 (Ontology Edges)
RELATE employee:zhang_san->belongs_to->department:ai_dept;
RELATE employee:li_si->manages->department:ai_dept;
RELATE employee:zhang_san->assign_to->project:project_x;
RELATE project:project_x->produces->document:doc_007;
RELATE project:project_x->produces->document:doc_008;

```

#### 第三步：实现 ReBAC 动态权限裁决查询

  当用户（如张三或外部顾问）尝试访问某篇文档时，我们不需要写复杂的业务判断代码，只需在 SurrealDB 中执行一段图遍历与规则裁决查询。

场景 A：基础项目成员访问规则验证
规则：如果员工被分配到某项目，且该项目产出了目标文档，则允许读取。  

``` text

  -- 参数化查询：检查张三 (employee:zhang_san) 是否能访问 文档 (document:doc_007)
SELECT * FROM document:doc_007 
WHERE id IN (
    -- 通过图遍历：从指定员工出发 -> 通过 assign_to 找到项目 -> 通过 produces 找到文档
    SELECT VALUE ->produces->document FROM (
        SELECT VALUE ->assign_to->project FROM employee:zhang_san
    )
);

```

执行结果： 能够成功返回 document:doc_007，说明权限校验通过（Allow）。


场景 B：高级多跳继承与合规规则（结合管理链与机密等级）
编写一个综合的 ReBAC 裁决函数或查询。例如：“总监李四虽然没有直接加入 Project-X，但他管理着张三所在的部门，且该文档不是绝密（TopSecret），或者他拥有管理特权”。

在 SurrealDB 中，可以写一个集成了图路径搜索和属性校验的复合查询：

``` text

-- 动态裁决：判断某个身份对目标文档是否有读取权限
LET $target_user = employee:li_si;
LET $target_doc = document:doc_008;

RETURN (
    SELECT 
        id, 
        title,
        -- 判定逻辑 1：是否是项目直接成员或其管理者
        (count(->produces<-project<-assign_to WHERE in = $target_user) > 0) AS is_direct_project_member,
        -- 判定逻辑 2：是否是上级主管路径覆盖
        (count(->produces<-project<-assign_to->employee->belongs_to<-manages WHERE in = $target_user) > 0) AS is_manager_chain_valid,
        -- 判定逻辑 3：机密等级过滤规则
        (confidentiality_level != 'TopSecret') AS level_permitted
    FROM $target_doc
);

```

### 为什么用 SurrealDB 实现 Ontology ReBAC 体验极佳？

图文合一（Graph-Native）： 权限本质上是图的连通性问题。SurrealDB 的 ->relation-> 语法天然支持多跳图遍历，代码极其直观。

Schema 即本体边界： 通过 DEFINE FIELD ... ASSERT 锁死了文档机密等级等元数据，从源头防止非法数据注入。

极高的工程落地效率： 相比于维护一套繁重的 OWL 语义网推理机，SurrealDB 兼顾了关系型数据库的高效查询、图数据库的遍历能力以及文档数据库的灵活性，非常适合直接支撑现代企业级微服务或 AI Agent 的权限护栏（Guardrail）。




