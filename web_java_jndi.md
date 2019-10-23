> https://docs.oracle.com/javase/tutorial/jndi/overview/index.html

> https://www.blackhat.com/docs/us-16/materials/us-16-Munoz-A-Journey-From-JNDI-LDAP-Manipulation-To-RCE.pdf



#### JNDI简介

JNDI(Java Naming and Directory Interface)

* Common interface to interact with Naming and Directory Services
* Naming Service
  * A Naming Service is an entity that associates names with values, also
known as “bindings”.
  * It provides a facility to find an object based on a name that is known as
“lookup” or “search” operation.
* Directory Service
  * Special type of Naming Service that allows storing and finding of
“directory objects.”
  * A directory object differs from generic objects in that it's possible to
associate attributes to the object.
  * A Directory Service, therefore offers extended functionality to operate on
the object attributes. 


什么是JNDI 简单概况如下

* JNDI是"通用接口"(Common interface)，用来和Naming Service和Directory Services交互
* 命名服务(Naming Service)
  * 一个命名服务(Naming Service)是一个将名称(names)和值(values)关联起来的实体，也被称为“绑定”(bindings)
  * 命名服务(Naming Service)提供了一种功能，可以根据一个名称(name)找到一个对象(object)，也被称为“查找”(lookup) 或 “搜索”(search) 操作
* 目录服务(Directory Service)
  * 目录服务(Directory Service)是一种特殊类型的命名服务(Naming Service)——它允许存储和查找“目录对象”(directory objects)
  * “目录对象”与“一般对象”的不同之处在于，“目录对象”可以将属性(attributes)关联到对象。
  * 因此，目录服务(Directory Service)提供了扩展功能——对“对象属性”进行操作


#### JNDI架构

The JNDI architecture consists of an API and a service provider interface (SPI).

* JNDI架构的组成
  * JNDI API - Java应用程序使用"JNDI API"访问各种命名服务(Naming Service)和目录服务(Directory Service)
  * SPI接口 -SPI接口允许各种命名服务(Naming Service)和目录服务(Directory Service)透明地被插入(plugged in)，从而实现了Java应用程序只需使用JNDI API即可访问这些服务

<div align=center><img src="./imgs/jndi_arch.gif"/></div>

* 小结JNDI架构
  * JNDI提供了一个通用接口(a common interface)，用来与不同类型的服务(services)进行交互(interact)
  * 命名管理器(Naming Manager)包含了一些静态方法，这些静态方法用于创建“上下文对象”(context objects)和“对象”(objects).引用位置信息(location information)
  * service provider interface (SPI) -  本接口允许JNDI管理各种不同的服务(services),即各种不同的服务被JNDI管理



