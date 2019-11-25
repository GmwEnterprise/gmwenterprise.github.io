---
title: Translate English offical document 'Spring Data Overview'
date: 2019-11-24 02:56:00
tags:
  - English
  - Spring Data
---

> Original link: [https://spring.io/projects/spring-data#overview](https://spring.io/projects/spring-data#overview)

Spring Data’s mission is to provide a familiar and consistent, Spring-based programming model for data access while still retaining the special traits of the underlying data store.

> Spring Data的目的是为数据访问提供一个熟悉且一致的，基于Spring的编程模型，同时仍保留了基础数据存储的特质。
> *-- data access: 数据访问*
> *-- the special traits: 特质*
> *-- underlying data store: 基础数据存储*
> *-- retain: 保留*

It makes it easy to use data access technologies, relational and non-relational databases, map-reduce frameworks, and cloud-based data services. This is an umbrella project which contains many subprojects that are specific to a given database. The projects are developed by working together with many of the companies and developers that are behind these exciting technologies.

> 他（Spring Data）使得数据访问技术，关系型和非关系型数据库，mapReduce框架，还有基于云的数据服务更易于使用。这是一个总括项目，其包含了众多的为给定数据库开发的子项目。这些项目是通过与这些令人激动的技术背后的许多公司和开发人员共同开发的。
> *-- map-reduce: 由Google提出的一个软件架构，用于大规模数据集（大于1TB）的并行运算*
> *-- umbrella project: 总括项目，可以理解为聚合项目*
> *-- cloud-based data services: 基于云的数据服务*

### Features 特性

- Powerful repository and custom object-mapping abstractions
- Dynamic query derivation from repository method names
- Implementation domain base classes providing basic properties
- Support for transparent auditing (created, last changed)
- Possibility to integrate custom repository code
- Easy Spring integration via JavaConfig and custom XML namespaces
- Advanced integration with Spring MVC controllers
- Experimental support for cross-store persistence

> - 强大的存储库和自定义对象映射抽象
> - 从存储库的方法名称来进行动态的查询推导
> - 支持数据变动可见（透明化）
> - 可以集成自定义的存储库代码
> - 可以通过JavaConfig或自定义的XML命名空间来集成Spring
> - 与Spring MVC 控制器的高级集成
> - 跨库持久化（实验性支持）
> *-- custom object-mapping abstractions: 自定义对象映射抽象*
> *-- dynamic query derivation: 动态查询推导（Spring Data 可以通过Repo接口的方法名称来推导生成对应的sql查询语句）*
> *-- transparent auditing: 透明审计（这里我翻译为了数据变动可见）*
> *-- integrate: 集成（动词）*
> *-- integration: 集成（名词）*
> *-- experimental: 实验性的*
> *-- cross-store persistence: 跨存储持久性*
