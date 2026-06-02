# 股票数据看板 - Code Wiki

## 1. 项目概述

### 1.1 项目简介
这是一个轻量级的股票数据看板单页面应用，用于实时展示中国A股市场的主要指数和自选股票信息。

### 1.2 核心功能
- 上证指数和创业板指实时数据展示
- 自选股票管理（添加/删除）
- 股票卡片式数据展示（价格、涨跌幅、开盘价、成交量等）
- 数据每30秒自动刷新
- 用户配置持久化存储（使用localStorage）

### 1.3 技术栈
- 前端：HTML5 + CSS3 + 原生JavaScript
- 数据来源：腾讯财经API（https://qt.gtimg.cn/）

---

## 2. 项目架构

### 2.1 文件结构
```
/workspace/
├── index.html              # 主页面文件（包含HTML、CSS、JavaScript）
└── CODE_WIKI.md            # 本文档
```

### 2.2 架构设计
项目采用单页面应用（SPA）架构，所有代码集成在一个 [index.html](file:///workspace/index.html) 文件中：

```
index.html
├── <head> 标签
│   ├── 元数据配置
│   └── 样式定义 (<style> 标签)
└── <body> 标签
    ├── DOM结构
    └── JavaScript代码 (<script> 标签)
```

---

## 3. 主要模块与功能

### 3.1 数据获取模块

#### 3.1.1 fetchStockData(code)
**位置**：[index.html#L194-L237](file:///workspace/index.html#L194-L237)

**功能**：获取单只股票的实时数据

**参数**：
- `code` (String)：6位股票代码

**返回值**：
- 成功返回包含股票数据的对象
- 失败返回 `null`

**数据结构**：
```javascript
{
    code: String,           // 股票代码
    name: String,           // 股票名称
    open: Number,           // 开盘价
    price: Number,          // 当前价
    high: Number,           // 最高价
    low: Number,            // 最低价
    volume: Number,         // 成交量（万手）
    amount: Number,         // 成交额（亿元）
    change: Number,         // 涨跌额
    changePct: String       // 涨跌幅（百分比，保留2位小数）
}
```

#### 3.1.2 fetchIndexData(symbol)
**位置**：[index.html#L239-L261](file:///workspace/index.html#L239-L261)

**功能**：获取指数数据

**参数**：
- `symbol` (String)：指数代码（如 'sh000001' 代表上证指数）

**返回值**：包含指数数据的对象

#### 3.1.3 decodeGBK(str)
**位置**：[index.html#L184-L192](file:///workspace/index.html#L184-L192)

**功能**：将GB18030编码字符串转换为UTF-8

**说明**：腾讯财经API返回的数据使用GB18030编码，需要解码处理

---

### 3.2 渲染模块

#### 3.2.1 renderStocks()
**位置**：[index.html#L273-L319](file:///workspace/index.html#L273-L319)

**功能**：渲染所有自选股票卡片

**流程**：
1. 检查股票列表是否为空
2. 获取所有股票数据
3. 清空并重新渲染股票网格
4. 更新数据更新时间

#### 3.2.2 updateIndices()
**位置**：[index.html#L263-L271](file:///workspace/index.html#L263-L271)

**功能**：更新上证指数和创业板指数据

---

### 3.3 交互模块

#### 3.3.1 addStock()
**位置**：[index.html#L321-L334](file:///workspace/index.html#L321-L334)

**功能**：添加新股票到自选列表

**验证规则**：
- 必须为6位数字
- 不能重复添加

#### 3.3.2 removeStock(code)
**位置**：[index.html#L336-L340](file:///workspace/index.html#L336-L340)

**功能**：从自选列表中删除股票

---

## 4. 关键类与函数说明

### 4.1 全局变量
| 变量名 | 类型 | 说明 | 位置 |
|--------|------|------|------|
| `stocks` | Array | 自选股票代码列表 | [index.html#L180](file:///workspace/index.html#L180) |
| `lastUpdateTime` | String | 数据最后更新时间 | [index.html#L181](file:///workspace/index.html#L181) |

### 4.2 关键函数速览

| 函数名 | 功能 | 调用位置 |
|--------|------|----------|
| `decodeGBK()` | GB18030解码 | `fetchStockData()`, `fetchIndexData()` |
| `fetchStockData()` | 获取股票数据 | `renderStocks()` |
| `fetchIndexData()` | 获取指数数据 | `updateIndices()` |
| `updateIndices()` | 更新指数显示 | 初始化、定时刷新 |
| `renderStocks()` | 渲染股票卡片 | 初始化、添加/删除股票、定时刷新 |
| `addStock()` | 添加股票 | 按钮点击、回车键 |
| `removeStock()` | 删除股票 | 删除按钮点击 |

---

## 5. 依赖关系

### 5.1 外部依赖
- **腾讯财经API**：https://qt.gtimg.cn/
  - 用途：获取股票和指数实时数据
  - 编码：GB18030
  - 数据格式：CSV风格字符串，用 `~` 分隔

### 5.2 浏览器API依赖
- `localStorage`：持久化存储自选股票列表
- `fetch()`：网络请求
- `TextDecoder`：文本编码转换
- `setInterval()`：定时刷新
- `DOM API`：页面元素操作

---

## 6. 数据流程

### 6.1 初始化流程
```
页面加载
  ↓
从localStorage读取自选股票列表（默认为 ['600519', '000858']）
  ↓
updateIndices() → 获取上证指数、创业板指数据 → 更新显示
  ↓
renderStocks() → 获取所有自选股票数据 → 渲染股票卡片
  ↓
启动30秒定时刷新
```

### 6.2 数据刷新流程
```
定时器触发（每30秒）
  ↓
updateIndices() → 更新指数
  ↓
renderStocks() → 更新股票卡片
```

### 6.3 添加股票流程
```
用户输入股票代码
  ↓
点击"添加股票"或按回车
  ↓
验证（6位数字、不重复）
  ↓
添加到stocks数组
  ↓
保存到localStorage
  ↓
调用renderStocks()刷新显示
```

---

## 7. 样式设计

### 7.1 配色方案
- 背景色：`#1a1a2e`（深蓝黑）
- 卡片背景：`#16213e`（深蓝）
- 主色调：渐变 `#667eea` → `#764ba2`（紫蓝）
- 上涨色：`#e74c3c`（红）
- 下跌色：`#2ecc71`（绿）

### 7.2 布局
- 响应式网格布局（`grid-template-columns: repeat(auto-fill, minmax(300px, 1fr))`）
- 卡片式设计，圆角16px
- 最大宽度1200px，居中显示

---

## 8. 项目运行方式

### 8.1 运行步骤
1. 直接在浏览器中打开 [index.html](file:///workspace/index.html) 文件
2. 或使用任何HTTP服务器（推荐用于解决CORS问题）：
   ```bash
   # 使用Python
   python3 -m http.server 8080
   # 然后访问 http://localhost:8080
   
   # 或使用Node.js (http-server)
   npx http-server -p 8080
   # 然后访问 http://localhost:8080
   ```

### 8.2 浏览器兼容性
- 需要支持ES6+的现代浏览器
- 需要支持 `TextDecoder` API
- 需要支持 `fetch` API

---

## 9. API说明

### 9.1 腾讯财经API
**接口地址**：`https://qt.gtimg.cn/q={symbol}`

**参数**：
- `symbol`：股票/指数代码，格式为 `市场代码+6位数字`
  - 上海市场：`sh` + 代码（如 `sh600519`）
  - 深圳市场：`sz` + 代码（如 `sz000858`）

**返回格式**：
```
v_{symbol}="{data}"
```
其中 `{data}` 是用 `~` 分隔的字符串，包含约50个字段

**关键字段索引**：
| 索引 | 说明 |
|------|------|
| 0 | 股票代码 |
| 1 | 股票名称 |
| 3 | 当前价 |
| 4 | 最高价 |
| 5 | 开盘价 |
| 6 | 最低价 |
| 7 | 成交量（手） |
| 8 | 成交额（元） |
| 30 | 更新时间 |

---

## 10. 扩展建议

### 10.1 功能扩展
- 添加K线图展示
- 支持港股/美股
- 添加预警功能
- 支持分组管理

### 10.2 技术优化
- 分离HTML、CSS、JavaScript到不同文件
- 使用构建工具（如Vite/Webpack）
- 添加单元测试
- 实现错误重试机制
- 添加加载动画

---

## 11. 注意事项

1. **CORS问题**：直接打开HTML文件可能存在跨域问题，建议使用本地HTTP服务器
2. **API依赖**：项目依赖腾讯财经API，若API变更可能导致功能失效
3. **数据刷新**：30秒刷新间隔可根据需要调整，注意不要过于频繁以免被限流
4. **localStorage**：数据存储在浏览器本地，清除浏览器数据会丢失自选股票列表

---

*文档生成时间：2026-06-02*
