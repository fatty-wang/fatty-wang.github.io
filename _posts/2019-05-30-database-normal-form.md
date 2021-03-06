---
layout: post
title: "数据库设计范式总结"
date: 2019-05-30 
author: "WangX"
catalog: true
header-style: text
tags:
    - database
    - basic
---


----


## 基本概念

> 范式是符号某一种级别的关系模式的集合，表示一个关系内部个属性之间的联系的合理化陈程度


* 实体： 客观存在的事物，具体的或抽象的，如 学生 或 学生与班级的关系
* 属性： 属性时实体所具有的某一特性，是个逻辑的概念，但在关系数据库中，属性也是一个物理概念，属性对应表中的一列。
* 元组： 表中的一行就是一个元组。
* 分量： 元组的某个属性值。
* 码：表中可以唯一确定一个元组的某个属性（或者属性组），如果这样的码有多个，这些码均称为**候选码**。
* 主码：从候选码中选出的一个。
* 全码：包含所有属性的码。
* 主属性：一个属性只要在任何一个候选码中出现过，就是主属性。
* 非主属性：没有在任何候选码中出现过。
* 码：一个属性（或属性组）不是这个表的码，但它是别的表的码，就称之为外码。
* 函数依赖：设X,Y是关系**R**的两个属性集合，当任何时刻**R**中的任意两个元组中的X属性值相同时，则它们的Y属性值也相同，则称X函数决定Y，或Y函数依赖于X。
* 完全函数依赖：设X,Y是关系**R**的两个属性集合，X’是X的真子集，存在 ***X->Y***，但对每一个X’都有 ***X’!->Y***，则称Y完全函数依赖于X。
* 部分函数依赖：设X,Y是关系**R**的两个属性集合，存在 ***X->Y***，若X’是X的真子集，存在 ***X’→Y***，则称Y部分函数依赖于X。
* 传递函数依赖：设X,Y,Z是关系R中互不相同的属性集合，存在***X→Y (Y !→X)，Y→Z***，则称Z传递函数依赖于X。

## 1NF 属性不可再分
* 第一范式是最基本的范式。符合1NF的关系中的每个属性都不可再分。不符合第一范式则不能称之为关系数据库。但能不能再分要看具体的业务场景。

## 2NF
* 2NF 在1NF的基础上，消除了非主属性对码的部分函数依赖。或者说非主属性完全函数依赖于码。
* 表中的属性必须完全依赖于全部主键，而不是部分主键.所以只有一个主键的表如果符合第一范式，那一定是满足第二范式。

## 3NF
* 3NF在2NF的基础之上，消除了非主属性对于码的传递函数依赖。
* 非主属性外的所有字段必须互不依赖
* 

## BCNF 
* 在3NF的基础上消除主属性对于码的部分与传递函数依赖，若一个关系达到了第三范式，并且它只有一个候选码，或者它的每个候选码都是单属性，则该关系自然达到BC范式。




