# DataViz Studio 技能指南

## 概述

本技能指导如何创建一个专业级数据可视化工作台 HTML 文件，具备以下核心功能：

- **展示图库**：左侧显示高质量图表缩略图，点击可加载对应代码
- **代码编辑器**：中间 CodeMirror 编辑器，支持语法高亮和实时运行
- **实时预览**：右侧 ECharts 图表预览区域
- **数据上传**：支持 CSV 文件上传和预览
- **导出功能**：支持 PNG/SVG 格式导出

## 技术栈

| 技术            | 版本      | 用途     |
| ------------- | ------- | ------ |
| ECharts       | 5.5.0   | 图表渲染引擎 |
| CodeMirror    | 5.65.16 | 代码编辑器  |
| 原生 JavaScript | ES6+    | 无框架依赖  |
| CSS3          | -       | 深色主题样式 |

## 项目架构

```
┌─────────────────────────────────────────────────────────────────┐
│  Header: Logo | 图库 | 编辑器 | 导出PNG | 导出SVG | 运行         │
├──────────────┬────────────────────────────┬────────────────────┤
│              │                            │                    │
│  展示图库     │    代码编辑区              │    实时预览        │
│  (280px)     │    (flex: 1)               │    (480px)         │
│              │                            │                    │
│  ┌─────────┐ │  ┌──────────────────────┐  │  ┌──────────────┐  │
│  │ 图表1   │ │  │ 📁 数据上传区         │  │  │              │  │
│  │ 缩略图  │ │  │    CSV 拖拽上传       │  │  │   ECharts    │  │
│  └─────────┘ │  ├──────────────────────┤  │  │   图表画布    │  │
│  ┌─────────┐ │  │ 📋 图表模板选择       │  │  │              │  │
│  │ 图表2   │ │  │    地图|散点|折线...  │  │  │              │  │
│  │ 缩略图  │ │  ├──────────────────────┤  │  │              │  │
│  └─────────┘ │  │ 💻 CodeMirror         │  │  │              │  │
│  ...         │  │    代码编辑器         │  │  │              │  │
│              │  │    Ctrl+Enter 运行    │  │  │              │  │
│              │  └──────────────────────┘  │  └──────────────┘  │
│              │                            │                    │
└──────────────┴────────────────────────────┴────────────────────┘
```

## HTML 结构模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DataViz Studio - 数据可视化工作台</title>

  <!-- ECharts -->
  <script src="https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js"></script>

  <!-- CodeMirror -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/codemirror.min.css">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/codemirror.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/mode/javascript/javascript.min.js"></script>

  <style>
    /* CSS 样式（见下文） */
  </style>
</head>
<body>
  <!-- Header -->
  <header class="header">...</header>

  <!-- Main Layout -->
  <main class="main">
    <!-- Gallery Panel (Left) -->
    <aside class="gallery-panel">...</aside>

    <!-- Editor Panel (Center) -->
    <section class="editor-panel">...</section>

    <!-- Preview Panel (Right) -->
    <aside class="preview-panel">...</aside>
  </main>

  <script>
    // JavaScript 逻辑（见下文）
  </script>
</body>
</html>
```

## CSS 样式规范

### 1. CSS 变量（深色主题）

```css
:root {
  --bg-primary: #0d1117;      /* 最深背景 */
  --bg-secondary: #161b22;    /* 次深背景 */
  --bg-tertiary: #21262d;     /* 卡片背景 */
  --border-color: #30363d;    /* 边框颜色 */
  --text-primary: #c9d1d9;    /* 主要文字 */
  --text-secondary: #8b949e;  /* 次要文字 */
  --accent: #58a6ff;          /* 强调色（蓝） */
  --accent-hover: #79c0ff;    /* 悬停强调色 */
  --success: #238636;         /* 成功色（绿） */
  --warning: #f0883e;         /* 警告色（橙） */
}
```

### 2. 布局样式

```css
/* 主布局 */
.main {
  display: flex;
  height: calc(100vh - 48px);
}

/* 左侧图库 */
.gallery-panel {
  width: 280px;
  background: var(--bg-secondary);
  border-right: 1px solid var(--border-color);
  display: flex;
  flex-direction: column;
}

/* 中间编辑区 */
.editor-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
}

/* 右侧预览区 */
.preview-panel {
  width: 480px;
  background: var(--bg-secondary);
  border-left: 1px solid var(--border-color);
  display: flex;
  flex-direction: column;
}
```

### 3. 图库项样式

```css
.gallery-item {
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 8px;
  background: var(--bg-tertiary);
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 0.2s;
}

.gallery-item:hover {
  border-color: var(--border-color);
  transform: translateY(-1px);
}

.gallery-item.active {
  border-color: var(--accent);
  background: rgba(88, 166, 255, 0.1);
}
```

## JavaScript 核心逻辑

### 1. 全局状态

```javascript
// 全局状态
let chartInstance = null;      // 当前图表实例
let currentData = null;        // 上传的 CSV 数据
let editor = null;             // CodeMirror 实例
let chinaGeoJson = null;       // 中国地图 GeoJSON
```

### 2. 内置数据集

```javascript
// mtcars 数据集（32辆汽车）
const mtcars = [
  {name:"Mazda RX4", wt:2.620, disp:160, cyl:6, mpg:21.0},
  {name:"Mazda RX4 Wag", wt:2.875, disp:160, cyl:6, mpg:21.0},
  // ... 更多数据
];

// airquality 数据集（纽约空气质量）
const airquality = [
  {Month:5, Day:1, Ozone:41, Wind:7.4, Temp:67},
  // ... 更多数据
];

// 中国城市灯光数据
const chinaLightCities = [
  {name:"北京", lon:116.407, lat:39.904, dn:68.5, cfps:52340},
  {name:"上海", lon:121.474, lat:31.230, dn:72.8, cfps:78560},
  // ... 更多数据
];
```

### 3. 代码模板系统

```javascript
const codeTemplates = {
  'line': `// 折线图模板
const data = currentData || airquality;

option = {
  title: { text: '数据趋势图', left: 'center' },
  tooltip: { trigger: 'axis' },
  xAxis: { 
    type: 'category',
    data: data.map((d, i) => 'Day ' + (i + 1))
  },
  yAxis: { type: 'value' },
  series: [{
    type: 'line',
    data: data.map(d => d.Wind || d.value),
    smooth: true,
    itemStyle: { color: '#58a6ff' }
  }]
};

renderChart(option);`,

  'bar': `// 柱状图模板
const data = currentData || mtcars.slice(0, 10);

option = {
  title: { text: '数据对比', left: 'center' },
  tooltip: { trigger: 'axis' },
  xAxis: { type: 'value' },
  yAxis: { 
    type: 'category',
    data: data.map(d => d.name),
    inverse: true
  },
  series: [{
    type: 'bar',
    data: data.map(d => d.mpg),
    itemStyle: { color: '#58a6ff' }
  }]
};

renderChart(option);`,

  // 更多模板...
};
```

### 4. 图表渲染函数

```javascript
// 渲染图表
function renderChart(option) {
  const container = document.getElementById('chartContainer');
  if (chartInstance) {
    chartInstance.dispose();
  }
  chartInstance = echarts.init(container);
  chartInstance.setOption(option);
}

// 运行代码
function runCode() {
  try {
    const code = editor.getValue();
    // 创建安全的执行上下文
    const fn = new Function(
      'currentData', 
      'mtcars', 
      'airquality', 
      'chinaLightCities',
      'renderChart', 
      'echarts', 
      code
    );
    fn(currentData, mtcars, airquality, chinaLightCities, renderChart, echarts);
  } catch (error) {
    console.error('Error:', error);
    document.getElementById('chartContainer').innerHTML = `
      <div class="empty-state">
        <div class="empty-icon">⚠️</div>
        <div class="empty-title">代码错误</div>
        <div class="empty-desc">${error.message}</div>
      </div>
    `;
  }
}
```

### 5. CSV 上传处理

```javascript
// 处理文件上传
function handleFileUpload(event) {
  const file = event.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = function(e) {
    parseCSV(e.target.result);
  };
  reader.readAsText(file);
}

// 解析 CSV
function parseCSV(text) {
  const lines = text.trim().split('\n');
  const headers = lines[0].split(',').map(h => h.trim());
  const data = [];

  for (let i = 1; i < lines.length; i++) {
    const values = lines[i].split(',');
    const row = {};
    headers.forEach((h, j) => {
      const val = values[j] ? values[j].trim() : '';
      row[h] = isNaN(val) ? val : parseFloat(val);
    });
    data.push(row);
  }

  currentData = data;
  // 更新 UI 显示数据预览
  showDataPreview(data);
}
```

### 6. CodeMirror 初始化

```javascript
function initEditor() {
  editor = CodeMirror.fromTextArea(document.getElementById('codeEditor'), {
    mode: 'javascript',
    theme: 'default',
    lineNumbers: true,
    lineWrapping: true,
    indentUnit: 2,
    tabSize: 2,
    extraKeys: {
      'Ctrl-Enter': runCode,
      'Cmd-Enter': runCode
    }
  });

  // 设置深色主题
  editor.getWrapperElement().style.background = '#0d1117';
  editor.getWrapperElement().style.color = '#c9d1d9';

  // 加载默认模板
  loadTemplate('line');
}
```

### 7. 导出功能

```javascript
function exportChart(format) {
  if (!chartInstance) return;
  const url = chartInstance.getDataURL({
    type: format,
    pixelRatio: 2,
    backgroundColor: '#fff'
  });
  const link = document.createElement('a');
  link.download = 'chart.' + format;
  link.href = url;
  link.click();
}
```

### 8. 中国地图 GeoJSON 加载

```javascript
function loadChinaGeoJSON(callback) {
  if (chinaGeoJson) {
    callback(chinaGeoJson);
    return;
  }

  const urls = [
    'https://geo.datav.aliyun.com/areas_v3/bound/100000_full.json',
    'https://geo.datav.aliyun.com/areas_v3/bound/geojson/china.json'
  ];

  // 尝试加载，失败则使用内置简化版
  // ... 加载逻辑
}
```

## 图库缩略图实现

每个图库项显示实际的 ECharts 缩略图：

```javascript
function initGalleryThumbnails() {
  const thumbnails = document.querySelectorAll('.gallery-thumbnail');

  thumbnails.forEach((container, index) => {
    const chart = echarts.init(container);

    // 根据索引设置不同的缩略图配置
    const options = getThumbnailOption(index);
    chart.setOption(options);
  });
}

function getThumbnailOption(index) {
  const baseOption = {
    grid: { left: 30, right: 10, top: 20, bottom: 20 },
    xAxis: { type: 'category', show: false },
    yAxis: { type: 'value', show: false }
  };

  switch(index) {
    case 0: // 折线图
      return {
        ...baseOption,
        series: [{ type: 'line', data: [30, 50, 40, 60, 55], smooth: true, itemStyle: { color: '#58a6ff' } }]
      };
    case 1: // 柱状图
      return {
        ...baseOption,
        series: [{ type: 'bar', data: [40, 60, 35, 50], itemStyle: { color: '#238636' } }]
      };
    // ... 更多类型
  }
}
```

## 图表模板清单

| 模板名称    | 类型  | 数据源              | 特性         |
| ------- | --- | ---------------- | ---------- |
| line    | 折线图 | airquality       | 平滑曲线、面积填充  |
| bar     | 柱状图 | mtcars           | 渐变色、圆角     |
| scatter | 散点图 | mtcars           | 回归线、异常检测   |
| pie     | 饼图  | chinaLightCities | 环形、标签      |
| heatmap | 热力图 | 模拟数据             | 颜色映射       |
| map     | 地图  | chinaLightCities | GeoJSON、气泡 |
| radar   | 雷达图 | 模拟数据             | 多维度对比      |
| sankey  | 桑基图 | 模拟数据             | 流向可视化      |

## 用户交互流程

```
用户操作                      系统响应
─────────────────────────────────────────────────────
点击图库项          ──▶     加载对应代码模板到编辑器
                          在预览区渲染图表

上传 CSV 文件       ──▶     解析 CSV 数据
                          显示数据预览
                          数据保存到 currentData

编辑代码            ──▶     实时语法高亮
                          无自动运行（需手动）

点击运行 / Ctrl+Enter ──▶   执行代码
                          在预览区渲染图表
                          错误时显示错误信息

点击导出 PNG/SVG    ──▶     调用 ECharts getDataURL
                          触发下载
```

## 代码模板编写规范

### 1. 模板结构

```javascript
`// 注释：图表名称和说明
// 数据来源说明

// 1. 数据准备
const data = currentData || 默认数据集;

// 2. 数据处理（如需要）
// ... 计算逻辑

// 3. ECharts 配置
option = {
  // 配置项
};

// 4. 渲染
renderChart(option);`
```

### 2. 数据访问

```javascript
// 使用上传的 CSV 数据
const data = currentData;

// 使用内置数据集
const data = mtcars;
const data = airquality;
const data = chinaLightCities;

// 混合使用
const data = currentData || mtcars;
```

### 3. 列名访问

```javascript
// CSV 列名直接作为属性访问
data.map(d => d.列名)

// 示例
data.map(d => d.销售额)
data.map(d => d['月份'])
```

## 常见问题解决

### Q1: 图表不显示

- 检查 `renderChart(option)` 是否被调用
- 检查 `option` 配置是否正确
- 检查数据是否为空

### Q2: CSV 数据读取错误

- 确保使用 `currentData` 而非 `sampleData.uploaded`
- 检查 CSV 列名是否正确
- 检查数据类型（字符串 vs 数字）

### Q3: 地图不显示

- 检查 GeoJSON 是否加载成功
- 检查 `echarts.registerMap('china', geoJson)` 是否执行
- 检查坐标范围是否正确

### Q4: 代码运行错误

- 检查语法错误（缺少分号、括号不匹配等）
- 检查变量是否定义
- 检查 API 调用是否正确

## 扩展建议

### 1. 添加更多图表类型

- 仪表盘（gauge）
- 漏斗图（funnel）
- 树图（tree）
- 关系图（graph）

### 2. 增强数据功能

- 支持 Excel 文件上传
- 数据清洗和转换
- 数据统计摘要

### 3. 增强编辑器功能

- 代码自动补全
- 代码格式化
- 代码历史记录

### 4. 增强导出功能

- 导出 PDF
- 批量导出
- 自定义分辨率

## 检查清单

创建 DataViz Studio 时，确保：

- [ ] HTML 结构完整（header + main + 三个面板）
- [ ] CSS 样式正确（深色主题、响应式）
- [ ] ECharts 正确引入（CDN）
- [ ] CodeMirror 正确引入（CDN + JS mode）
- [ ] 内置数据集定义（mtcars、airquality、chinaLightCities）
- [ ] 代码模板完整（至少 6 种图表类型）
- [ ] 图库缩略图初始化
- [ ] CSV 上传功能正常
- [ ] 导出功能正常
- [ ] 错误处理完善

## 示例输出

完成后的 DataViz Studio 应具备：

1. **左侧图库**：7 个图表缩略图，点击加载代码
2. **中间编辑器**：CodeMirror 编辑器，语法高亮
3. **右侧预览**：ECharts 图表实时渲染
4. **数据上传**：CSV 拖拽上传，数据预览
5. **导出功能**：PNG/SVG 一键导出

---

**技能版本**：v1.0  
**最后更新**：2026年  
**适用场景**：数据可视化工作台、在线图表编辑器、数据分析工具
