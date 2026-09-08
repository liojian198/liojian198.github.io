---
title: "行为树"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/behavior_tree
date: 2026-09-08 16:00:00
---

# 简介

  以前做游戏的时候，行为树经常用，最近在看机器人的时候， 也看到这个算法在被使用。

  参考： https://robohub.org/introduction-to-behavior-trees/

# 说明

## 什么是行为树(behavior tree)？

  行为树(behavior tree)是用来实现非人工角色复杂行为的工具，它具有下面这些特征：

  行为树是树：执行时从根结点开始按照指定的顺序遍历，直到到达终结状态。
  叶子结点都是可执行的行为：叶子结点会进行具体的操作，可以是一个简单的检测操作，也可以是一个更复杂的操作，结点会返回状态信息(成功，失败，运行中)。
  内部结点控制树的遍历：内部结点会根据孩子结点返回的状态信息，按照特定的规则确定下一个执行的结点。
  游戏行业使用行为树(behavior tree)来定义非玩家角色的行为：虚幻引擎和Unity引擎都提供了专门的工具来帮助用户定义行为树。

  在机器人行业，行为树也被用来让机器人实现复杂任务。如果读者对此感兴趣，可以阅读Behavior Trees in Robotics and AI: An Introduction。


  本文使用的术语遵循Behavior Trees in Robotics and AI: An Introduction中的定义。
  
  下图给出了行为树的不同结点类型及其对应的图示：

  <img width="825" height="322" alt="image" src="https://github.com/user-attachments/assets/45240bed-7c1e-4a29-8a8e-d09f19c0775c" />


行为树结点的一次触发称为一次tick，会返回成功(success)，失败(failure)，运行中(running)的状态信息给它的父结点。

执行结点(execution node)：行为树的叶子结点，可以是动作结点(action node)或条件结点(condition node)。对于条件结点(condition node)会在一次tick后立马返回成功或失败的状态信息。对于动作结点(action node)则可以跨越多个tick执行，直到到达它的终结状态。一般来说，条件结点用于简单的判断(比如钳子是否打开?)，动作结点用于表示复杂的行为(比如打开房门)。

控制结点(control node)：控制结点是行为树的内部结点，它们定义了遍历其孩子结点的方式。控制结点的孩子可以是执行结点，也可以是控制结点。顺序(Sequence)，备选(Fallback)，并行(Parallel)这3种类型的控制结点可以有任意数量的孩子结点，它们的区别在于对其孩子结点的处理方式。而装饰(Decorator)结点只能有一个孩子结点，用来对孩子结点的行为进行自定义修改。

下面我们通过图示分别介绍这些不同类型的控制结点：

  <img width="385" height="228" alt="image" src="https://github.com/user-attachments/assets/ac592c6d-b135-4021-8859-a83012fca501" />

顺序结点：按顺序执行孩子结点直到其中一个孩子结点返回失败状态或所有孩子结点返回成功状态。


<img width="383" height="229" alt="image" src="https://github.com/user-attachments/assets/145cfd6c-0ccf-46f4-bfba-03797e2195d0" />

备选结点：按顺序执行孩子结点直到其中一个孩子结点返回成功状态或所有孩子结点返回失败状态。一般用来实现角色的备选行为。


<img width="379" height="229" alt="image" src="https://github.com/user-attachments/assets/a6c715bf-1980-4f2f-b406-988cc2b33b62" />

并行结点：“并行执行”所有孩子结点。直到至少M个孩子(M的值在1到N之间)结点返回成功状态或所有孩子结点返回失败状态。


<img width="418" height="225" alt="image" src="https://github.com/user-attachments/assets/b039ab59-b18f-4b1a-bb98-c63dfaea30d7" />

装饰结点：以自定义的方式修改孩子结点的行为。比如Invert类型的装饰结点，可以反转其孩子结点返回的状态信息。为了方便他人理解，应该尽可能使用比较常见的装饰结点。





