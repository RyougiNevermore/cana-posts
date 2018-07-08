---
title: Postgres 配置 -- 内存设置
date: 2017-07-09 03:29:48
description:
featured_image:
issue: https://github.com/RyougiNevermore/ryouginevermore.github.io/issues/2
tags:
 - 数据库
 - Postgres
 - 优化
---

# 前言

配置文件postgres.conf中的很多设置都会影响性能。其中关于内存设置是较为关键的。

## shared_buffers

这是最重要的参数，postgresql通过shared_buffers和内核/磁盘打交道。 因此应该尽量大，让更多的数据缓存在shared_buffers中，通常设置为实际RAM的10％是合理的，比如50000(400M) 

## work_mem

在pgsql 8.0之前叫做sort_mem。postgresql在执行排序操作时，
会根据work_mem的大小决定是否将一个大的结果集拆分为几个小的和work_mem查不多大小的临时文件。

显然拆分的结果是降低了排序的速度。因此增加work_mem有助于提高排序的速度。通常设置为实际RAM的2%-4%，根据需要排序结果集的大小而定，比如81920(80M)

## effective_cache_size

是postgresql能够使用的最大缓存，
这个数字对于独立的pgsql服务器而言应该足够大，比如4G的内存，可以设置为3.5G(437500)

## maintence_work_mem

这里定义的内存只是在CREATE INDEX, VACUUM等时用到，因此用到的频率不高，但是往往这些指令消耗比较多的资源，
因此应该尽快让这些指令快速执行完毕：给maintence_work_mem大的内存，比如512M(524288)

## max_connections

通常，max_connections的目的是防止max_connections * work_mem超出了实际内存大小。

比如，如果将work_mem设置为实际内存的2%大小，则在极端情况下，如果有50个查询都有排序要求，而且都使用2％的内存，则会导致swap的产生，系统性能就会大大降低。
当然，如果有4G的内存，同时出现50个如此大的查询的几率应该是很小的。不过，要清楚max_connections和work_mem的关系。