---
name: deep-analyze
description: 深度逆向分析。反编译关键函数、追踪调用链、分析数据流、识别算法和加密逻辑。适合已脱壳的二进制文件。
user-invocable: true
argument-hint: [function_name_or_address]
---

对当前 session 的二进制文件执行深度逆向分析。如果指定了函数名/地址，聚焦分析该函数；否则从入口点开始全局分析。

## 执行步骤

### 阶段 1: 全局视图

1. 调用 `idb_meta` 获取基本信息
2. 调用 `entrypoints` 获取入口点
3. 调用 `list_funcs` 获取函数列表（前 100 个），关注：
   - 函数名中包含关键词的（main, init, start, decrypt, encode, key, auth, login, check, verify, license）
   - 非库函数（排除以 `_` 或 `sub_` 开头的标准库函数）
4. 调用 `imports` 分析导入表，按功能分类

### 阶段 2: 关键函数分析

5. 如果用户指定了目标（$ARGUMENTS），用 `lookup_funcs` 定位
6. 否则，从入口点开始，调用 `decompile` 反编译
7. 对关键函数调用 `callers` 和 `callees` 追踪调用链
8. 调用 `xrefs_to` 分析交叉引用

### 阶段 3: 深入分析

9. 对可疑函数调用 `decompile` 获取伪代码
10. 调用 `analyze_funcs` 获取函数详细信息（参数、局部变量、栈帧）
11. 调用 `basic_blocks` 分析控制流
12. 对关键路径调用 `find_paths` 追踪执行路径
13. 调用 `analyze_strings` 查找函数引用的字符串

### 阶段 4: 模式识别

分析过程中关注以下模式：

**加密/编码:**
- XOR 循环（常见混淆手法）
- AES/DES/RC4 常量（S-Box, 轮常量）
- Base64 字符表
- 自定义编码算法

**反调试:**
- IsDebuggerPresent / NtQueryInformationProcess
- 时间检测（rdtsc, GetTickCount）
- 异常处理滥用

**网络通信:**
- C2 地址构造
- 数据编码/加密后发送
- 协议解析逻辑

**持久化:**
- 注册表操作
- 服务创建
- 计划任务
- 文件投放

## 输出格式

```
## 深度分析报告: [文件名]

### 程序结构
- 入口点: 0x...
- 函数总数: ...
- 关键函数: [列表，含地址和简要描述]

### 调用关系
[ASCII 调用图或关键路径描述]

### 核心逻辑
[对每个关键函数的伪代码分析和功能说明]

### 发现
- [编号列出所有有价值的发现]
- [识别到的算法、协议、行为模式]

### 数据流
- 输入: [数据从哪里来]
- 处理: [怎么处理的]
- 输出: [数据到哪里去]
```

## 注意事项

- 如果文件加壳，建议先执行 /unpack
- 分析过程中遇到大量混淆代码，标记出来但不要陷入细节
- 优先分析有明确语义的函数，跳过明显的库函数
