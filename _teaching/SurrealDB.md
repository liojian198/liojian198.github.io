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








