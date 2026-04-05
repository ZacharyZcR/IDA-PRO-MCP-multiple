---
name: triage
description: 二进制文件快速分类。识别文件类型、壳/保护、编译器、熵值，给出初步判断。用于拿到一个未知样本时的第一步分析。
user-invocable: true
argument-hint: [session_id]
---

对当前 session（或指定 session）的二进制文件执行快速分类分析。

## 执行步骤

1. **切换 session**（如果提供了 $ARGUMENTS）：调用 `session_switch` 切换到目标 session

2. **基础信息采集**：并行调用以下工具：
   - `idb_meta` — 获取文件元数据（路径、MD5、SHA256、大小、基址）
   - `detect_packer` — 识别壳/编译器/链接器
   - `detect_entropy` — 熵值分析
   - `segments` — 获取段/节信息
   - `entrypoints` — 入口点
   - `imports` — 导入函数

3. **字符串采样**：调用 `strings` 获取前 50 条字符串，关注：
   - 可疑 URL/IP/域名
   - 文件路径
   - 注册表键
   - 调试/错误信息
   - 加密相关字符串

4. **综合研判**：根据采集到的信息输出分类报告

## 输出格式

用中文输出结构化报告，包含：

```
## 分类报告: [文件名]

### 基本信息
- 文件类型: PE32/PE64/ELF32/ELF64/Mach-O
- MD5/SHA256: ...
- 文件大小: ...
- 编译器: ...
- 架构: ...

### 保护检测
- 壳/保护: [检测结果]
- 熵值: [数值] ([packed/not packed])
- 可疑高熵段: [列出]

### 导入分析
- 导入函数总数: ...
- 高风险 API: [如 VirtualAlloc, CreateRemoteThread, WriteProcessMemory 等]
- 网络相关: [socket, connect, send, recv, WinHTTP 等]
- 文件操作: [CreateFile, WriteFile, DeleteFile 等]
- 注册表: [RegSetValue, RegCreateKey 等]

### 字符串线索
- [列出有价值的字符串]

### 初步判断
- 分类: [正常软件 / 可疑样本 / 高度可疑 / 已加壳需脱壳]
- 建议下一步: [脱壳 → /unpack | 深度分析 → /deep-analyze | 恶意软件分析 → /malware-triage]
```
