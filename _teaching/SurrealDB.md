---
title: "SurrealDB 简介"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/SurrealDB
date: 2026-07-02 14:30:00
---

# 简介

  SurrealDB 在几年前我就关注到他了，但是当时我玩的方向是数仓和数据湖及实时流式计算。 最近ai agent 火的不行， 在看相关技术的时候，有人专门提到SurrealDB， 一个多模式的数据，在AI 能有什么作用。 
不过SurrealDB在业务刚起步的时候，用起来还是很爽的。几年前，SurrealDB很多东西不成熟及集群化好像也是收费， 总之当时用的时候就是各种限制。后续就没有跟了。巴拉巴拉的说了一堆，现在还是要记录下 这个东西。

     https://cocoindex.io/docs/examples/

     https://github.com/surrealdb

rust 相关的东西也记录下吧

    https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html#using-extern-functions-to-call-external-code

    https://docs.rust-embedded.org/book/interoperability/index.html

    https://rust-lang.github.io/rust-bindgen/

    https://michael-f-bryan.github.io/rust-ffi-guide/


# 操作相关

## 处理多表数据查询

1. 利用图关系 (最推荐，最符合 SurrealDB 特性)

如果你在多表之间建立了 RELATE 关系，那么你根本不需要 JOIN。你可以直接像“走迷宫”一样从一个表跳到另一个表。

场景： 查询 user (人) -> cart (购物车) -> product (商品)。

语法：

``` json

-- 从 user 开始，顺着 cart 边，找到关联的 product
SELECT *, ->cart->product AS items FROM person;

```

2. 利用自动关联 (Implicit Fetching / Expansion)
如果你在表字段中存储了 record<other_table>（即外键 ID），SurrealDB 允许你使用 .* 符号自动展开关联数据。

场景： order 表中有一个 user 字段（指向 person 表），还有一个 products 字段（数组，指向 product 表）。

语法：

``` json

-- 自动获取订单的用户详情和所有商品的详情
SELECT 
    id, 
    user.* AS user_info, 
    products.* AS product_details 
FROM order;

```

这种方式在处理简单的多对一或多对多映射时非常高效。

3. 多表联合查询 (UNION / Subqueries)
如果你需要从多张完全无关的表中获取数据，或者需要复杂的过滤，可以使用子查询或 UNION 逻辑。

场景： 统计不同类型的人员总数（假设有 student 和 teacher 两张表）。

语法：

``` json

SELECT 
    (SELECT count() FROM student) AS student_count,
    (SELECT count() FROM teacher) AS teacher_count;

```

## from 里面能放多张表吗？

在 SurrealDB 中，FROM 后面确实可以放置多张表，但这与传统 SQL 的用法不同。在 SurrealDB 中，FROM 后面跟着的是一个 记录集合（Record Set），你可以通过多种方式在一个 FROM 子句中同时指定多张表。

1. 使用逗号分隔多张表 (标准做法)

你可以直接在 FROM 后面列出多个表名或 ID，SurrealDB 会将它们视为一个统一的查询范围。

``` json

-- 同时查询 student 表和 teacher 表
SELECT * FROM student, teacher;

-- 查询特定的多张表及其中的特定 ID
SELECT * FROM product:iphone, order:123;

```

这种方式非常适合当你需要统一从多个来源汇总数据时使用。

2. 使用子查询 (Subquery)
如果你需要对每张表先进行初步处理（例如过滤或聚合），然后再合并在一起，可以在 FROM 中使用子查询：

```json

SELECT * FROM (
    SELECT name, 'student' AS type FROM student WHERE age < 20
    UNION
    SELECT name, 'teacher' AS type FROM teacher WHERE salary > 5000
);

```

3. 利用图路径作为数据源 (最符合图数据库逻辑)
在 SurrealDB 中，更常见的做法不是把两张表“拼在一起”，而是从一张表出发，通过边（Edge）“导航”到另一张表。

如果你想查询“购物车”和“产品”两张表，你不需要写 FROM cart, product，而是直接通过图遍历：

``` json

-- 从人出发，导航到所有相关的购物车，再导航到商品
SELECT ->cart->product.* FROM person;

```


## SurrealQL 语句执行顺序 (逻辑流)

以下是执行引擎处理语句的优先级排序：

1.FROM (定义数据源): 数据库首先确定要操作的记录集合（它是所有后续操作的基础）。如果使用了 ONLY，则会限制扫描范围。

2. WHERE (过滤/行级安全): 这是第一道防线。SurrealDB 会根据条件（以及你定义的 PERMISSIONS）过滤掉不符合条件的记录。

3.SPLIT (拆分记录): 如果你在 SELECT 中使用了 SPLIT，系统会将数组字段拆分为多条记录（这是 SurrealDB 处理嵌套数据的一大特色）。

4.GROUP BY (分组): 在进行聚合计算之前，系统会将数据按指定字段进行分组。

5.SELECT / VALUE (计算与投影): 计算表达式的值、应用别名（AS）和忽略字段（OMIT）。这是数据最终变形的阶段。

6.ORDER BY (排序): 在生成最终结果列表前，对所有记录进行排序。

7.LIMIT & START (分页): 确定返回哪一部分数据。

8.FETCH (深度获取/关联加载): 这是最后一步。它将查询结果中的记录引用（Record IDs）自动展开为完整的对象，以减少 N+1 查询次数。

其他特定语句的执行时机
除了上述核心处理流，还有一些语句在特定的时间点生效：

WITH [NOINDEX | INDEX]：在 FROM 阶段之前生效。它直接告诉引擎在扫描数据源时应该如何使用索引，直接决定了读取数据的速度。

TIMEOUT：贯穿整个过程。它作为一个监控器，一旦在任一阶段超过指定时间，立即停止所有操作。

TEMPFILES：在 GROUP 和 ORDER 阶段生效。如果处理的数据集过大，超出了内存限制，数据库会利用这些临时文件来完成复杂的排序或分组操作。

EXPLAIN [FULL]：在所有阶段之前和之后生效。如果你加上这个参数，引擎不会执行查询，而是输出整个执行计划（执行顺序、索引命中情况等），供你分析优化。

一个核心的误区提醒
很多开发者容易混淆 FETCH 和普通 SELECT。

FETCH 不是 JOIN：它不参与 WHERE 或 GROUP 的逻辑。它发生在数据已经筛选完成之后，仅仅是为了把那些引用 ID（如 person:alice）变成具体的完整用户信息（如 {"name": "Alice", "age": 25}）。

顺序优势：因为 FETCH 在 LIMIT 之后执行，这意味着如果你只获取 10 条数据，它就只会展开这 10 条记录的关联对象，而不会展开整表，这正是 SurrealDB 高性能的原因之一。

## 如果有子查询呢？

在 SurrealQL 中，子查询（Subqueries）通常出现在 FROM、WHERE 或 SELECT 的表达式中。它们的执行逻辑是“嵌套闭环”的——即外层查询在执行到子查询所在的位置时，会暂停当前执行流，转而去完成子查询的计算，将子查询的结果作为数据喂给外层。

子查询的执行优先级（嵌套逻辑）
当查询包含子查询时，执行顺序会变得更精细：

子查询优先执行（Inside-Out）：无论子查询位于哪里，数据库引擎必须先计算出子查询的完整结果集。

结果回传（Injection）：将子查询的结果作为一个“临时表”或“临时值数组”注入到外层查询的对应位置。

外层逻辑继续（Resuming）：外层查询继续按照原有的执行顺序（从 FROM 到 FETCH）进行处理。

不同位置子查询的执行解析
1. 子查询在 FROM 中 (数据源转换)
这是最常见的用法，外层查询完全依赖子查询产生的数据。

执行顺序：FROM (子查询计算完成) -> WHERE -> GROUP -> SELECT ...

场景：你需要先对 student 和 teacher 做某种过滤和合并，再从这个结果集里筛选。

2. 子查询在 WHERE 中 (过滤条件)
子查询作为外层过滤的依据。

执行顺序：外层 FROM (开始) -> (暂停) 子查询执行 -> 返回结果集 -> (恢复) -> 外层 WHERE 使用结果集进行过滤 -> SELECT ...

场景：查找所有买了“苹果”的用户。

3. 子查询在 SELECT 表达式中 (字段计算)
执行顺序：外层查询开始 -> 每一行执行时 -> 调用子查询计算该行的列值。

场景：统计每个人的购物车商品总数。

给你的性能优化建议
尽量避开“相关子查询”：如果你在 SELECT 表达式中使用了相关子查询，尽量将其改写为 JOIN 或“图路径遍历”（->cart），因为图遍历是引擎优化的原生路径，速度远快于子查询。

能写在 FROM 里的不要写在 SELECT 里：如果子查询产生的逻辑可以预处理，先在 FROM 里做一次聚合，这能减少外层查询扫描的次数。

利用 FETCH 代替子查询：如果你在 SELECT 里写子查询是为了获取关联对象详情，请改用 FETCH，因为 FETCH 是在所有过滤和分页完成之后才执行的，开销极小。

## 权限控制

行级权限，列级权限，基于操作层级的权限，基于关系的图遍历权限，基于环境上下文的权限，命名空间与数据库级权限，存储过程中的逻辑权限

### SurrealDB 的身份验证系统 (Authentication) 详解

  SurrealDB 的身份验证系统设计得非常精妙，它将数据库层面的鉴权（Authentication）与业务逻辑层面的授权（Authorization）紧密结合在一起。

其核心在于通过 SCOPE（作用域）和 TOKEN（令牌）将用户的身份信息（Context）注入到数据库的运行上下文中。

1. 核心架构：Scope 与 Token
SurrealDB 不依赖外部中间件来处理权限，而是通过两种方式定义谁可以登录：

SCOPE (登录逻辑): 适用于数据库原生的登录流程（类似传统 Session）。你定义 SIGNIN 时如何查询用户、如何验证密码，以及登录后会话包含哪些数据。

TOKEN (令牌校验): 适用于前后端分离场景（JWT）。你定义如何解析外部颁发的 JWT，并将其中的 Claim 映射为数据库变量。


2. 用户登录流 (SIGNIN)
当你调用 SIGNIN 时，SurrealDB 内部执行以下操作：

验证身份：执行 SIGNIN 中定义的查询（如比对密码 crypto::argon2::compare）。

创建上下文：如果验证成功，数据库会创建一个会话，并生成一个会话 ID 或 Token。

变量注入 ($auth)：这是最关键的一步。数据库将该用户的记录数据加载到内存中，并将其赋值给全局变量 $auth。

比如，如果登录用户是 person:alice，那么所有引用 $auth 的地方都会自动指向 Alice 的记录数据。

3. 定义身份与权限映射
下面是一个完整的 SCOPE 配置示例，展示了如何构建身份上下文：

```text

-- 定义一个用于用户登录的作用域
DEFINE SCOPE account SESSION 24h
    -- 登录时执行的 SQL，用于校验密码
    SIGNIN (
        SELECT * FROM person 
        WHERE email = $email 
        AND crypto::argon2::compare(password, $pass)
    )
    -- 登录成功后，用户记录会被注入到 $auth
    -- $auth 的内容直接决定了后续 PERMISSIONS 的过滤效果
    SESSION (
        RETURN $auth;
    );

```

4. 权限与身份的联动 (The Magic of $auth)
   
一旦用户通过上述 SCOPE 登录，他们在执行任何 SELECT, UPDATE 等操作时，SurrealDB 都会自动读取当前连接的 $auth 变量。

这就实现了“一行代码实现所有权限”的奇迹：

``` text

DEFINE TABLE document SCHEMAFULL
    PERMISSIONS
        -- 这里的 $auth.tenant 和 $auth.id 是由 SIGNIN 阶段自动注入的
        FOR select WHERE tenant = $auth.tenant;

```

关键点：如何使用 JWT 实现外部鉴权？
如果你的系统使用了 Auth0、Firebase 或自建的 Node.js/Go 服务颁发 JWT，你可以直接在 SurrealDB 中配置 DEFINE TOKEN：

``` text

DEFINE TOKEN login_token ON NAMESPACE TYPE HS256 VALUE "my-secret-key" 
    -- 解析 JWT 中的声明（Claim）并赋予 $auth
    CLAIM "tenant" AS tenant;

```


这样，即使你的用户不是在数据库内登录的，只要带着合法的 JWT 过来，SurrealDB 就能提取出 tenant 放入 $auth.tenant，实现 RLS 过滤。





### 行级权限

  当你定义 FOR update WHERE $auth.role = 'admin' 或 FOR delete WHERE created_by = $auth.id ... 时，你不是在控制“谁能访问 product 表”，而是在控制“谁能对表中的哪一行记录执行什么操作”。

为什么说这是记录级的？
动态求值：当用户尝试对某一行 product:123 执行 DELETE 时，SurrealDB 引擎会取出该记录的 created_by 字段，并与当前用户 $auth.id 进行动态比较。

查询过滤 (SELECT)：如果你有 FOR select WHERE tenant = $auth.tenant，那么当用户查询该表时，数据库会自动给查询加上一个“隐形过滤器”。如果用户没有权限看到某一行，那一行数据根本不会出现在查询结果中——就像它在数据库中不存在一样。

内核强制性：这种权限逻辑嵌入在数据访问层，无论你通过何种方式（API、控制台、代码）调用，只要涉及该表，规则就会被触发。

#### RLS 怎么建表

在 SurrealDB 中，构建支持记录级权限（RLS）的表结构，核心在于将权限逻辑内嵌到 DEFINE TABLE 语句中。

以下是实现 RLS 的标准化步骤和最佳实践代码。

1. 建立带有 RLS 的表结构
在定义表时，你可以通过 PERMISSIONS 子句明确定义不同操作的访问规则。

```text

  -- 1. 定义资源表 (以 product 为例)
DEFINE TABLE product SCHEMAFULL
    -- 2. 设置权限策略
    PERMISSIONS
        -- 所有人都可以查询该表（或根据需要添加 WHERE 条件）
        FOR select WHERE true
        
        -- 只有记录的创建者或具备 'admin' 角色的用户才能执行更新
        FOR update WHERE created_by = $auth.id OR $auth.role = 'admin'
        
        -- 只有管理员可以删除
        FOR delete WHERE $auth.role = 'admin'
        
        -- 只有通过身份验证的用户才能创建记录
        FOR create WHERE $auth.id != NONE;

-- 3. 定义字段，特别是关联用户的标识符
DEFINE FIELD name ON TABLE product TYPE string;
DEFINE FIELD created_by ON TABLE product TYPE record<person> DEFAULT $auth.id;
DEFINE FIELD tenant ON TABLE product TYPE record<tenant>;

```

2. 多租户隔离 (更高级的 RLS)
在企业级应用中，最常见的需求是“租户隔离”，即用户只能看到属于自己公司（tenant）的数据。

```text

  DEFINE TABLE order SCHEMAFULL
    PERMISSIONS
        -- 强制过滤：任何操作都必须匹配用户的 tenant
        FOR select, update, delete WHERE tenant = $auth.tenant;

DEFINE FIELD tenant ON TABLE order TYPE record<tenant>;

```


3. 如何设置 RLS 的测试环境？
RLS 只有在用户登录（SIGNIN）并拥有 $auth 上下文时才会生效。

第一步：定义权限访问范围
你需要确保用户在登录时，Token 中包含了正确的属性（如 role, tenant）。

第二步：在查询中测试
当你在控制台或代码中操作时，通过 SIGNIN 模拟不同用户：

```text

-- 模拟用户登录
SIGNIN person:alice PASSWORD 'password123' SCOPE user;

-- 此时执行 SELECT，数据库会自动根据定义的 WHERE 条件过滤数据
SELECT * FROM product;

```

4. RLS 设计的 3 个关键原则
使用 $auth 和 $this：

$auth：指代当前登录的用户对象。

$this：指代当前正在被扫描的那一行记录。

通过 WHERE created_by = $auth.id 这种逻辑，你实际上是在给每一行记录“贴上标签”。

确保性能：由于 RLS 会在每次访问时触发，请务必为你用于权限过滤的字段建立索引。


谨慎使用 FULL 权限：在开发阶段可以使用 PERMISSIONS FULL，但生产环境请务必将权限细化到每一类操作（select, create, update, delete）。


#### $auth 怎么获得？

  这个过程通常分为三个步骤：

  1. 定义 Scope (登录作用域)
在定义 SCOPE 时，你可以指定登录成功后 SESSION 中包含什么内容。

```text

DEFINE SCOPE user SESSION 30m
    -- 在登录时进行校验
    SIGNIN (
        SELECT * FROM person 
        WHERE email = $email AND crypto::argon2::compare(password, $pass)
    )
    -- 关键点：这里定义了 $auth 变量的内容
    SESSION (
        -- 这里你可以手动映射用户信息到 session
        -- 数据库会自动将当前登录的 person 记录信息放入 $auth
        RETURN $auth; 
    );

```

2. 用户登录 (SIGNIN)
当你调用 SIGNIN 时，数据库会通过 SCOPE 的逻辑验证用户。一旦通过，数据库会创建一个会话，并自动将该用户记录的所有字段（包括 tenant 字段）加载到 $auth 变量中。

```text

-- 登录后，Alice 的记录信息（包含 tenant 字段）会被存入当前连接的上下文中
SIGNIN SCOPE user {
    email: "alice@example.com",
    pass: "password123"
};

```


3. 变量注入
登录成功后，无论你执行什么查询，SurrealDB 引擎内部都会自动将该会话中的用户信息映射为 $auth 对象。

如果你的 person 表里有一个字段叫 tenant，那么登录后：

$auth.tenant 就等于该用户的 tenant 字段值（例如 tenant:company_a）。

4. 如何确保 $auth.tenant 始终可用？
   
为了保证权限逻辑不出错，你需要遵循以下最佳实践：

1. 确保用户记录包含租户信息：在 person 表中，必须有一个 tenant 字段。
2. 通过 JWT 或 Session 传递：如果你使用外部的 JWT，你需要在数据库的 TOKEN 配置中解析出 tenant 声明，并将其赋值给 $auth。
3.调试方式：你可以在控制台登录后，直接运行以下语句查看 $auth 里到底存了什么：如果返回的 JSON 中不包含 tenant，那么 $auth.tenant 自然就是 NONE，导致权限过滤失败。



























