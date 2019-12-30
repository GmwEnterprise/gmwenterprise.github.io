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

### Main modules 主要模块

- Spring Data Commons - Core Spring concepts underpinning every Spring Data module.
- Spring Data JDBC - Spring Data repository support for JDBC.
- Spring Data JDBC Ext - Support for database specific extensions to standard JDBC including support for Oracle RAC fast connection failover, AQ JMS support and support for using advanced data types.
- Spring Data JPA - Spring Data repository support for JPA.
- Spring Data KeyValue - `Map` based repositories and SPIs to easily build a Spring Data module for key_value stores.
- Spring Data LDAP - Spring Data repository support for Spring LDAP.
- Spring Data MongoDB - Spring based, object-document support and repositories for MongoDB.
- Spring Data Redis - Easy configuration and access to Redis from Spring applications.
- Spring Data REST - Exports Spring Data repositories as hypermedia-driven RESTful resources.
- Spring Data for Apache Cassandra - Easy configuration and access to Apache Cassandra or large scale, highly available, data oriented Spring applications.
- Spring Data for Apache Geode - Easy configuration and access to Apache Geode for highly consistent, low latency, data oriented Spring applications.
- Spring Data for Apache Solr - Easy configuration and access to Apache Solr for your search oriented Spring applications.
- Spring Data for Pivotal GemFire - Easy configuration and access to Pivotal GemFire for your high consistent low latency/high through-put, data oriented Spring applications.

> - Spring Data Commons - 支撑着每一个Spring Data模块的核心概念
> - Spring Data JDBC - Spring Data 库对JDBC的支持
> - Spring Data JDBC Ext - 支持标准JDBC的数据库特定扩展，包括了对Oracle快速连接故障转移的支持，AQ JMS支持以及高级数据类型使用的支持
> - Spring Data JPA - Spring Data库对JPA的支持
> - Spring Data KeyValue - 轻松构建基于Map以及SPI的用于键值存储的Spring Data模块
> - Spring Data LDAP - Spring Data 库对Spring LDAP的支持
> - Spring Data MongoDB - 基于Spring的文档对象模型的支持以及适用MongoDB的库
> - Spring Data Redis - 轻松配置且易于从Spring应用中访问Redis
> - Spring Data REST - 将Spring Data存储库导出为超媒体驱动的RESTful资源
> - Spring Data for Apache Cassandra - 轻松配置并访问Apache Cassandra或大规模、高可用，面向数据的Spring应用
> - Spring Data for Apache Geode - 易于配置和访问Apache Geode，以实现高度一致、低延迟、面向数据的Spring应用
> - Spring Data for Apache Solr - 易于配置，并可访问面向搜索的Spring应用的Apache Solr
> - Spring Data for Pivotal GemFire -  轻松配置和访问Pivotal GemFire，以实现高度一致，低延迟/高吞吐量的面向数据的Spring应用程序