---
title: neo4j实战入门
date: 2025-08-15 11:13:37
updated: 2025-08-15 11:13:37
tags: 
comments: false
categories: 
thumbnail: https://cdn2.zzzmh.cn/wallpaper/origin/5f1d84f8880d11ebb6edd017c2d2eca2.jpg/fhd?auth_key=1757001600-6433d97265c9611e0f84fd4a1aff8c64edbd4d38-0-612a4f849237131b05f4d2d20ea011d7
published: false
---
# 1. 环境安装

## 1.1 docker安装

```bash
docker run \
--name neo4j \
-p7474:7474 -p7687:7687 \
-e NEO4J_AUTH=neo4j/yourpassword \
-d \
neo4j:latest
```

> http://localhost:7474 访问neo4j浏览器


# 2. CQL使用

## 1. Cypher语言

使用Cypher语法来描述关系

- 其中箭头表示关系方向
- knows表示关系标签

![社交图](images/社交图.svg)

> (王五)<-[:knows]-(李华)-[:knows]->(张三) -[:knows]->(王五)

所描述的信息为 李华认识张三和王五，而张三则认识王五
## 2. 常用命令
