---
layout: post
title: "Software Construction Summary 2"
subtitle: ""
date: 2019-03-16
author: Aether
category: coding
tags: CODE ANDROID JAVA
finished: true
---

## Test-First Programming
### Testing levels
- Unit testing 单元测试
  - 等价类划分 partitioning
     • 全覆盖 Full cartestian produck
     • 覆盖每个取值 Cover each part
  - 边界值分析 BVA (binary value analysis)
- Integration testing 集成测试
- System testing 系统测试

设计spec —>设计test cases —> code
### Code coverage
- Fuction coverage
- Statement coverage
- Branch coverage
- Condition coverage
- Path coverage
- path > branch > statement

## ADT
### Primitive Types/Reference Types
- primitive types
  - stack, exist only when in use
  - immutable
  - no ID, only value
  - dirt cheap
  - int, long, byte, short, char, float, double, boolean
- Object Reference Types
  - heap, garbage collected
  - some mutable, some not
  - ID + value
  - more costly
  - Classes, interfaces, arrays, enums, annotations

### Static Types/Dynamic Types
- static types
  - 在编译阶段进行类型检查 -- type
- dynamic data types
  - 在运行阶段进行类型检查 -- value
  
### Immutable/Mutable
- immutability - final
  - references / value cannot be changed
  - final class cannot be inherited
  - final method cannot be overridden by subclasses
- String —— immutable
- StringBuilder —— mutable (never use Date)
  - 防御式拷贝 defensive copy
      - 避免指向同一内存地址 避免被修改的潜在风险
      - 可能造成内存浪费
