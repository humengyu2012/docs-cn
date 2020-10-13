---
title: 下推计算结果缓存
aliases: ['/docs-cn/stable/coprocessor-cache/','/docs-cn/v4.0/coprocessor-cache/']
---

# 下推计算结果缓存

TiDB 从 4.0 起支持下推计算结果缓存（即 Coprocessor Cache 功能）。开启该功能后，将在 TiDB 实例侧缓存下推给 TiKV 计算的结果，在部分场景下起到加速效果。

## 配置

Coprocessor Cache 的配置均位于 TiDB 的 `tikv-client.copr-cache` 配置项中。Coprocessor 的具体开启和配置方法，见 [TiDB 配置文件描述](/tidb-configuration-file.md#tikv-clientcopr-cache-从-v400-版本开始引入)。

## 特性说明

+ 所有 SQL 在单个 TiDB 实例上的首次执行都不会被缓存。
+ 缓存仅存储在 TiDB 内存中，TiDB 重启后缓存会失效。
+ 不同 TiDB 实例之间不共享缓存。
+ 缓存的是下推计算结果，即使缓存命中，后续仍有 TiDB 计算。
+ 缓存以 Region 为单位。对 Region 的写入会导致涉及该 Region 的缓存失效。基于此原因，该功能主要会对很少变更的数据有效果。
+ 下推计算请求相同时，缓存会被命中。通常在以下场景下，下推计算的请求是相同或部分相同的：
    - SQL 语句完全一致，例如重复执行相同的 SQL 语句。

        该场景下所有下推计算的请求都是一致的，所有请求都能利用上下推计算缓存。

    - SQL 语句包含一个变化的条件，其他部分一致，变化的条件是表主键或分区主键。

        该场景下一部分下推计算的请求会与之前出现过的一致，部分请求能利用上下推计算结果缓存。

    - SQL 语句包含多个变化的条件，其他部分一致，变化的条件完全匹配一个复合索引列。

        该场景下一部分下推计算的请求会与之前出现过的一致，部分请求能利用上下推计算结果缓存。

+ 该功能对用户透明，开启和关闭都不影响计算结果，仅影响 SQL 执行时间。

## 检验缓存效果

目前 Coprocessor Cache 尚为实验性功能，用户无法判断一个 SQL 语句中有多少下推请求命中了缓存，也无法了解整体的缓存命中情况。后续版本中将引入相应的监控和检查方法。