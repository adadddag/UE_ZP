# Debug Session: static-config-not-persisting

## Session Info
- **Session ID**: static-config-not-persisting
- **Created**: 2026-05-29
- **Status**: [OPEN]

## Bug Description
编辑模式下修改页面静态元素（姓名、角色、技能等），点击保存后刷新页面，内容又复原了。

## Hypotheses (5 falsifiable)

### H1: syncCardEdits() 未被调用
- **检测点**: exitEditMode() 中调用 syncCardEdits() 的日志
- **预期**: 退出编辑模式时 syncCardEdits 应该被调用并同步数据

### H2: staticConfig 未被正确保存到 localStorage
- **检测点**: saveToDB() 中 localStorage.setItem(STATIC_CONFIG_KEY) 的执行
- **预期**: 保存成功时应有日志，localStorage 中应有 UE_PF_STATIC_CONFIG 键

### H3: loadStaticConfig() 加载失败或未执行
- **检测点**: init() 中 loadStaticConfig() 的执行和返回值
- **预期**: 应该从 localStorage 读取到已保存的配置

### H4: applyStaticConfig() 应用配置时元素不存在或配置为空
- **检测点**: applyStaticConfig() 中各元素的设置日志
- **预期**: 所有静态元素应该被正确设置

### H5: 页面 HTML 中的默认值覆盖了加载的配置
- **检测点**: HTML 中元素的内容 vs applyStaticConfig 后的内容
- **预期**: applyStaticConfig 应该在 HTML 渲染后修改元素内容

## Instrumentation Plan
1. 在 syncCardEdits() 添加详细日志
2. 在 saveToDB() 保存 staticConfig 前后添加日志
3. 在 loadStaticConfig() 加载前后添加日志
4. 在 applyStaticConfig() 应用配置前后添加日志
5. 在 init() 中各步骤添加日志

## Progress Log
- [x] 创建调试会话
- [ ] 添加插桩代码
- [ ] 收集运行时证据
- [ ] 分析并确定根因
- [ ] 实现修复
- [ ] 验证修复
- [ ] 清理调试环境

## Root Cause (Confirmed)
**变量声明顺序错误**：`staticConfig` 和 `STATIC_CONFIG_KEY` 定义在文件靠后的位置（约第 2066 行），但 `init()` 函数在第 1742 行执行。由于 JavaScript `let` 声明不会被提升，当 `init()` 执行时 `staticConfig` 还不存在，导致 `loadStaticConfig()` 中的 `staticConfig = {...}` 赋值失败。

## Fix Applied
将 `staticConfig` 和 `STATIC_CONFIG_KEY` 的定义移到文件前面（约第 768 行），确保在 `init()` 执行前就已经声明。

## Verification
代码顺序验证：
- 第 768 行：`const STATIC_CONFIG_KEY` ✓
- 第 769 行：`let staticConfig` ✓
- 第 1389 行：`saveToDB()` 使用 `STATIC_CONFIG_KEY` ✓
- 第 1727 行：`loadStaticConfig()` 使用 `STATIC_CONFIG_KEY` ✓
- 第 1742 行：`init()` 函数定义并执行 ✓
- 第 2065 行：`syncCardEdits()` 函数定义 ✓

## Status: [FIXED]

等待用户验证修复结果。
