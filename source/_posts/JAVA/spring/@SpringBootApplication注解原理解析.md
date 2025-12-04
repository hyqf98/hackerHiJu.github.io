---
title: "@SpringBootApplication注解原理解析"
date: 2024-12-30 19:07:05
updated: 2024-12-30 19:07:05
tags:
  - Java
  - 源码
comments: true
categories:
  - Java
  - 源码
  - Spring
thumbnail: https://cdn2.zzzmh.cn/wallpaper/origin/4e952013c980414da49ee18e7fb4199b.jpg/fhd?auth_key=1749052800-a9d6db4059f59d93c3c9dff4b20c7e6a2df84469-0-6a2ffb5ffc3b4a6f9912f1b6c7d68a36
published: false
---
# @SpringBootApplication

SpringBoot的启动注解，是一个组合注解，其中包括了以下几个注解：

- SpringBootConfiguration：用于标注启动类是一个配置类
  - Configuration：配置类注解
  - Indexed：用于提高类的解析效率，项目编译打包时会自动生成 **META-INF/spring.components** 文件，会被 **CandidateComponentsIndexLoader** 读取并且加载，转换为 **CandidateComponentsIndex** 对象
- EnableAutoConfiguration：springboot用于自动配置的重点注解
  - AutoConfigurationPackage：导入了 **AutoConfigurationPackages.Registrar**
  - Import：导入了 **AutoConfigurationImportSelector**
- ComponentScan：扫描组件
  - TypeExcludeFilter：扫描自定义的数据
  - AutoConfigurationExcludeFilter：扫描出配置类以及自动配置类



## 1. EnableAutoConfiguration

自动配置类注解，其中导入了 **AutoConfigurationImportSelector** 

