+++
date = '2026-01-05T14:18:16+08:00'
draft = true
title = '实验范式知识图谱设计方案'
categories = ['心理学知识图谱', '实验范式']
description = 'Exploring the intersection of psychology and graph theory to understand human behavior and decision-making.'

+++


## 前言

## 核心节点类型设计

```
// 主要节点
(:Task)              - 实验任务 (1064个)
(:Category)          - 任务类别 (IAT, Stroop, N-Back等)
(:Concept)           - 心理学概念 (注意力、记忆、抑制控制等)
(:Parameter)         - 实验参数
(:DataField)         - 数据字段
(:Metric)            - 性能指标
(:Publication)       - 参考文献
```

##  关系类型设计

```
// 核心关系
(:Task)-[:BELONGS_TO]->(:Category)
(:Task)-[:MEASURES]->(:Concept)
(:Task)-[:HAS_PARAMETER]->(:Parameter)
(:Task)-[:OUTPUTS]->(:DataField)
(:Task)-[:TRACKS]->(:Metric)
(:Task)-[:CITES]->(:Publication)
(:Task)-[:VARIANT_OF]->(:Task)         // 任务变体
(:Parameter)-[:AFFECTS]->(:Metric)
(:Metric)-[:INDICATES]->(:Concept)
```

## 数据收集与导入

1. **数据来源**：爬取的milisecond实验manuals

