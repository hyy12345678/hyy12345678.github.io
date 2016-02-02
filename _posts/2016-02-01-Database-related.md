---
layout: post
title:  数据库相关
date:   2016-02-01 15:05:32
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---

# Database related

－ 解锁oracle表
  dba用户登录
  sqlplus / sysdba
  
  锁表查询SQL:
  ```
  SELECT object_name, machine, s.sid, s.serial
  FROM gv$locked_object l, dba_objects o, gv$session s 
  WHERE l.object_id　= o.object_id 
  AND l.session_id = s.sid; 
  ```
  
  释放SESSION SQL:
  ```
  alter system kill session 'sid, serial'; 
  ALTER system kill session '23, 1647'; 
  ```






