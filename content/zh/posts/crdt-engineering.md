---
title: "CRDT 与实时协作的工程实践"
date: 2025-09-02
draft: false
description: "从 Yjs 到 Automerge，拆解冲突自由数据类型在真实业务中的取舍与落地。"
tags: ["CRDT", "实时协作", "分布式系统"]
category: "技术"
---

## 什么是 CRDT？

CRDT（Conflict-free Replicated Data Type）是一种可以在分布式系统中自动解决冲突的数据结构。

### 核心思想

- 每个副本独立更新
- 合并时总能收敛到一致状态
- 不需要中央协调器

## 主流方案对比

| 特性 | Yjs | Automerge |
|------|-----|-----------|
| 性能 | 优秀 | 良好 |
| API 设计 | 灵活 | 语义化 |
| 生态 | 丰富 | 成长中 |

## 落地经验

在三个生产项目中，我们得到的结论是...

<!-- 更多内容 -->
