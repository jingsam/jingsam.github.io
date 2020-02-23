---
title: 查询PostgreSQL中的数据类型
tags:
---


pg_type

format_type(typoid, typmod)

postgis_typmod_type(tpymod)

SELECT * FROM pg_attribute WHERE attrelid = 'mytable'::regclass;