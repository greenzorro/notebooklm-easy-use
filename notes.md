# notebooklm-easy-use 项目备忘录

## 1. 项目概述

### 1.1 项目目的

本文档旨在详细记录 `notebooklm-easy-use` Tampermonkey 用户脚本的技术架构、实现细节和开发维护指南。

**项目定位**: 增强 Google NotebookLM 使用体验的用户脚本，通过自动化批量更新 Google Drive 资料来源提升用户生产力。

**重要提示**: 每次修改功能或 Google 更新前端代码后，请及时更新此备忘录。

### 1.2 核心价值

Google NotebookLM 官方客户端存在以下痛点：
- 需要手动逐个点击更新 Google Drive 来源（Docs/Sheets/Slides）
- 无法批量检测和更新已过期的资料
- 更新状态缺乏明确反馈，难以确认是否成功

本脚本通过自动化批量操作解决上述问题。

---

## 2. 技术架构

### 2.1 整体架构

脚本采用**函数式模块架构**，代码组织清晰分层：

```
Configuration
    │
    ├─ CONFIG          Timing and feature flags
    ├─ SELECTORS       DOM selectors
    ├─ SYNC_RESULT     Return value constants
    │
State Management
    │
    └─ isRunning       Execution state flag
    │
Utility Functions
    │
    ├─ log()           Logging wrapper
    ├─ wait()          Promise-based delay
    ├─ waitForElement() Element detection with MutationObserver
    ├─ clickElement()  Enhanced click simulation
    ├─ highlightElement() Visual debugging with box-shadow
    └─ isOnSourceListPage() Page state detection
    │
Core Functions
    │
    ├─ Data Extraction
    │   └─ getGoogleDriveSources()
    │
    ├─ Navigation
    │   └─ goBack()
    │
    └─ Sync Operations
        ├─ findSourceByTitle()
        ├─ openSourceDetail()
        ├─ waitForSyncButton()
        ├─ clickSyncButton()
        └─ syncSource()
    │
Orchestration
    │
    └─ autoSync()      Main controller
    │
UI Components
    │
    └─ addManualButton()
```

### 2.2 核心配置

#### 2.2.1 时间与重试配置

```javascript
const CONFIG = {
    PAGE_LOAD_DELAY: 2000,        // Wait for Angular rendering after navigation
    POLL_INTERVAL: 1000,          // Interval for checking element state changes
    OPERATION_TIMEOUT: 10000,     // Maximum wait for any operation
    MAX_RETRY_ATTEMPTS: 5,        // Maximum attempts for polling operations
    HIGHLIGHT_DURATION: 1000,     // How long to show element highlights
    BETWEEN_ITEMS_DELAY: 300,     // Delay between processing multiple items
    DEBUG_MODE: true,             // Show visual debugging info
};
```

#### 2.2.2 选择器配置

```javascript
const SELECTORS = {
    SOURCE_CONTAINER: '.single-source-container',
    SOURCE_ICON: 'mat-icon.source-item-source-icon',
    SOURCE_TITLE: '.source-title-column',
    DETAIL_CONTAINER: '.source-panel',
    SYNC_BUTTON: '.source-refresh',
    SYNC_SUCCESS: '.source-refresh--success',
    BACK_BUTTON: '.source-panel .panel-header button',
};
```

**图标类型识别**:
- `article`: Google Docs
- `drive_spreadsheet`: Google Sheets
- `drive_presentation`: Google Slides

#### 2.2.3 返回值常量

```javascript
const SYNC_RESULT = {
    UPDATED: 'updated',
    SKIPPED: 'skipped',
    FAILED: 'failed'
};
```

---

## 3. 核心功能实现

### 3.1 资料来源识别

**函数**: `getGoogleDriveSources()`

**逻辑**:
1. 遍历所有资料卡片容器
2. 检查每个容器的图标元素文本内容
3. 判断是否为 Google Drive 来源
4. 记录资料在列表中的索引位置和基本信息

**返回数据结构**:
```javascript
{
    originalIndex: number,       // 资料在列表中的位置索引
    title: '资料标题',
    type: 'Docs' | 'Sheets' | 'Slides'
}
```

### 3.2 单个资料同步

**主函数**: `syncSource(source)`

**子函数拆分**:

| 函数 | 职责 |
|---|---|
| `findSourceByIndex(index)` | 通过列表索引定位 DOM 元素 |
| `openSourceDetail(container, titleElement, sourceTitle)` | 打开资料详情页并验证导航成功 |
| `waitForSyncButton(detailPanel)` | 轮询检测同步按钮是否出现 |
| `clickSyncButton(detailPanel)` | 点击同步按钮并验证成功状态 |

**执行流程**:
```
1. findSourceByIndex()       → 通过索引定位 DOM 元素（适应 DOM 变化）
2. openSourceDetail()         → 打开详情页
3. waitForSyncButton()        → 检测同步按钮
4. clickSyncButton()          → 点击同步并验证结果
5. goBack()                   → 返回列表页
```

**返回状态** (使用 `SYNC_RESULT` 常量):
- `SYNC_RESULT.UPDATED`: 成功更新
- `SYNC_RESULT.SKIPPED`: 已是最新版本
- `SYNC_RESULT.FAILED`: 同步失败

### 3.3 批量同步控制

**函数**: `autoSync()`

**执行流程**:
```
1. 页面状态验证：确保在资料列表页
   - 如不在列表页 → 尝试返回
   - 如无法返回 → 弹出错误提示并终止

2. 获取所有 Google Drive 资料来源

3. 遍历处理每个资料
   - 调用 syncSource() 处理单个资料
   - 统计三种状态数量
   - 输出处理进度

4. 输出最终统计
```

**日志输出示例**:
```
Processing 5 sources...
[1/5] Document A
[2/5] Spreadsheet B
[3/5] Presentation C
[4/5] Document D
[5/5] Document E
Done: 3 updated, 1 skipped, 1 failed
```

### 3.4 手动触发按钮

**函数**: `addManualButton()`

**按钮位置**: 固定定位 `left: 110px, top: 79px`

**样式特征**:
- 链接风格（蓝色文字 `#1a73e8`）
- 透明背景，无下划线
- 悬停时颜色变深（`#1557b0`）
- emoji 图标：🔄

**按钮文本**: `Sync docs/sheets/slides`

**防重复机制**: 检查 `#nlm-auto-sync-btn` 是否已存在

---

## 4. 关键技术要点

### 4.1 DOM 引用管理

**实现方式**: 存储位置索引，每次使用前动态查找 DOM 元素

```javascript
function getGoogleDriveSources() {
    containers.forEach((container, index) => {
        driveSources.push({
            originalIndex: index,    // 存储位置索引
            title: ...,
            type: ...
        });
    });
}

function findSourceByIndex(originalIndex) {
    const allContainers = document.querySelectorAll(SELECTORS.SOURCE_CONTAINER);
    // 通过索引定位对应的 DOM 元素
    return {
        container: allContainers[originalIndex],
        titleElement: allContainers[originalIndex].querySelector(SELECTORS.SOURCE_TITLE)
    };
}
```

**特点**:
- 每次使用前通过索引重新定位，确保获取最新 DOM
- 支持重复标题的资料
- 适应 Angular 动态渲染

### 4.2 点击目标选择

**关键发现**: 点击整个资料卡片 (`.single-source-container`) 无法触发导航。

**正确做法**: 必须点击标题元素 (`.source-title-column`)

### 4.3 增强点击模拟

**函数**: `clickElement(element)`

**实现策略**: 组合多种事件模拟真实用户操作

```javascript
function clickElement(element) {
    element.focus();
    element.dispatchEvent(new MouseEvent('mousedown', { bubbles: true, cancelable: true }));
    element.dispatchEvent(new MouseEvent('mouseup', { bubbles: true, cancelable: true }));
    element.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true }));
    element.dispatchEvent(new KeyboardEvent('keydown', { bubbles: true, cancelable: true, key: 'Enter', keyCode: 13 }));
}
```

### 4.4 SPA 导航检测

**问题**: NotebookLM 是单页应用，URL 不变化，通过 DOM 变化切换页面。

**检测方法**: 对比详情面板的 className 变化

```javascript
const panelBefore = document.querySelector(SELECTORS.DETAIL_CONTAINER);
const panelClassBefore = panelBefore ? panelBefore.className : 'no panel';

// 执行点击...

const panelAfter = document.querySelector(SELECTORS.DETAIL_CONTAINER);
const panelClassAfter = panelAfter ? panelAfter.className : 'no panel';

if (panelClassBefore === panelClassAfter) {
    // 导航失败
    return null;
}
```

### 4.5 视觉调试实现

**函数**: `highlightElement(element, duration)`

**实现方式**: 使用 `box-shadow` 而非 `border`

```javascript
function highlightElement(element, duration) {
    if (!CONFIG.DEBUG_MODE) return;

    const originalBoxShadow = element.style.boxShadow;
    element.style.boxShadow = '0 0 0 4px rgba(255, 0, 0, 0.5), 0 0 0 2px rgba(255, 0, 0, 0.8)';
    element.style.transition = 'box-shadow 0.3s';

    setTimeout(() => {
        element.style.boxShadow = originalBoxShadow;
    }, duration);
}
```

**优势**: `box-shadow` 不影响元素布局尺寸，避免页面抖动

### 4.6 日志系统

**日志函数**:

```javascript
function log(message, ...args)      // Normal logs
function logWarn(message, ...args)  // Warning logs
function logError(message, ...args) // Error logs
```

**日志策略**: 只记录关键信息，避免冗余输出
- 进度跟踪: `"Processing N sources..."`, `"[1/N] title"`
- 错误提示: `"Source not found"`, `"Click failed"`, `"Sync failed"`
- 状态确认: `"Sync confirmed"`, `"Done: X updated..."`

---

## 5. 维护指南

### 5.1 常见问题处理

| 问题 | 可能原因 | 解决方案 |
|---|---|---|
| 未找到资料 | 选择器失效 | 检查 `SOURCE_CONTAINER` 和 `SOURCE_TITLE` |
| 无法进入详情页 | 点击目标错误 | 确保点击的是标题元素，不是整个卡片 |
| 同步按钮未检测到 | 轮询次数不足 | 增加 `MAX_RETRY_ATTEMPTS` |
| 返回按钮未找到 | 页面状态异常 | 检查 `BACK_BUTTON` 选择器 |
| 脚本无响应 | 未在列表页启动 | 检查页面状态验证逻辑 |

### 5.2 选择器失效检测

**失效表现**:
- 控制台输出 "Source not found"
- "Detail panel timeout"
- 日志显示大量失败

**排查步骤**:
1. 打开 NotebookLM 资料来源页
2. 在控制台执行 `document.querySelector(SELECTORS.XXX)`
3. 使用开发者工具检查元素结构
4. 更新失效的选择器

### 5.3 Google 前端更新应对

**检测方法**:
1. 功能突然失效
2. 控制台无报错但选择器返回 `null`
3. 视觉高亮出现在错误位置

**处理流程**:
1. 使用开发者工具检查目标元素
2. 更新 `SELECTORS` 对象
3. 本地测试验证
4. 更新版本号
5. 发布到 Greasy Fork

### 5.4 调试模式开关

**配置**: `CONFIG.DEBUG_MODE`

- `true`: 显示红框高亮
- `false`: 静默运行

**切换方式**: 修改 `DEBUG_MODE: true` 为 `false`

---

## 6. 开发规范

### 6.1 代码风格

- 使用 ES6+ 语法
- 函数式编程，避免类化
- 常量集中管理在 `CONFIG`、`SELECTORS`、`SYNC_RESULT`
- 异步操作使用 `async/await`
- 错误处理使用 `try-catch`

### 6.2 扩展新功能

遵循现有模式：

```javascript
// 1. 在 CONFIG 中添加配置
const CONFIG = {
    // ...existing
    NEW_CONFIG: value,
};

// 2. 创建独立函数
async function newFeature() {
    try {
        // logic
    } catch (error) {
        logError('Error:', error);
    }
}

// 3. 在适当位置集成
async function syncSource(source) {
    // ...existing
    await newFeature();
}
```

### 6.3 函数设计原则

**单一职责**: 每个函数只负责一个明确的任务

**函数拆分**: 复杂操作拆分为多个可组合的子函数

```javascript
// syncSource 作为编排层，调用各子函数完成流程
async function syncSource(source) {
    const detailPanel = await openSourceDetail(source.container, source.titleElement, source.title);

    if (!detailPanel) return SYNC_RESULT.SKIPPED;

    const syncButton = await waitForSyncButton(detailPanel);
    if (!syncButton) {
        await goBack();
        return SYNC_RESULT.SKIPPED;
    }

    const success = await clickSyncButton(detailPanel);
    await goBack();

    return success ? SYNC_RESULT.UPDATED : SYNC_RESULT.FAILED;
}
```

**命名规范**:
- `getGoogleDriveSources()` - 数据提取
- `openSourceDetail()` - 执行导航
- `waitForSyncButton()` - 等待条件
- `clickSyncButton()` - 执行操作
