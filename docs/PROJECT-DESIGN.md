# PaneCI — 项目设计文档

版本：v1.0  
作者：自动生成（基于代码静态分析）  
生成日期：2025-10-31

## 一、项目简介
PaneCI（仓库 waitting49/PaneCI）是基于 mxGraph/draw.io 的前端图表编辑器变体。源码以 JavaScript 为主，包含图元（shapes）定义、侧边面板（sidebar）以及渲染器（mxCellRenderer）等模块。该文档从整体设计视角给出技术要点、功能模型与模块划分，重点聚焦于：
- 画布（Canvas / Graph）相关功能
- 图元（Shapes）定义与渲染
- 右侧样式弹框（样式/属性面板）相关功能（基于 mxGraph 样式体系的交互设计）

本文档基于代码中发现的关键文件（主要包括：`src/main/webapp/js/grapheditor/Shapes.js`、`src/main/webapp/js/grapheditor/Sidebar.js`、`src/main/webapp/js/diagramly/sidebar/Sidebar-Bootstrap.js`、`src/main/webapp/js/diagramly/Minimal.js`、`src/main/webapp/mxgraph/src/view/mxCellRenderer.js` 等）进行归纳与设计说明。

## 二、技术要点（Technical highlights）
- 底层渲染与交互依赖 mxGraph（draw.io 源码），使用 mxCell、mxGeometry、mxGraph、mxCellRenderer、mxUtils 等核心类。
- 图元以 Shape 对象和注册机制（mxCellRenderer.registerShape）定义，形状通过重写 paintBackground / paintVertexShape 等方法实现绘制逻辑。
- 样式以 key=value 风格的字符串保存在 cell.style 中（常见 key：fillColor、strokeColor、fontSize、fontColor、rounded、size、inset、dx、direction 等）。
- 侧边栏（Shapes 面板）通过构建 mxCell（含 mxGeometry 和 style 字符串）作为模板，拖拽/插入可创建图元。
- 右侧样式面板应基于 mxGraph 的样式 API（graph.getModel(), graph.getModel().beginUpdate(), graph.setCellStyles(), graph.model.setStyle() 等）来读写样式，从而实现样式同步与实时预览。

## 三、功能模型（Use cases / 功能清单）
1. 画布与编辑
   - 新建/加载图纸，节点/连线的创建、选择、移动、缩放、删除。
   - 右键菜单（popupMenu）、工具栏与快捷交互（在代码中见 popupMenuHandler 的使用）。
2. 图元库（Sidebar）
   - 多个 palette（Bootstrap、Infographic、Minimal 等）预置图形模板。
   - 从侧边栏拖拽/点击将图元添加至画布。
3. 图元渲染与样式
   - 每类图元由 Shape 类实现绘制（支持自定义参数，如 size、inset、top/right/bottom/left 角风格等）。
   - 图元标签（label）支持边界调整（getLabelMargins）。
4. 右侧样式弹框（Inspector）
   - 当选择图元时展示关联样式属性并支持编辑（颜色、边框、圆角、尺寸、特殊 shape 参数等）。
   - 修改即时反映在画布（实时预览），并记录到 cell 样式中以便保存/导出。

## 四、主要模块划分（高层）
- 核心渲染层（mxGraph、mxCellRenderer）  
  责任：创建/管理视觉节点，提供 Shape 注册与绘制生命周期。  
  代表文件：`src/main/webapp/mxgraph/src/view/mxCellRenderer.js`
- Shapes 定义层  
  责任：定义图元的具体绘制逻辑、注册名称及其可配置样式 key。  
  代表文件：`src/main/webapp/js/grapheditor/Shapes.js`
- 侧边栏（Shapes Palette）  
  责任：提供图元模板集合并将模板暴露为拖拽/插入源。  
  代表文件：`src/main/webapp/js/grapheditor/Sidebar.js`、`src/main/webapp/js/diagramly/sidebar/Sidebar-Bootstrap.js`、`src/main/webapp/js/diagramly/sidebar/Sidebar-Infographic.js`
- UI 启动与挂载（主入口/最小 UI）  
  责任：启动 editor UI，控制 popup 窗口、窗口化 sidebar 等行为。  
  代表文件：`src/main/webapp/js/diagramly/Minimal.js`
- 样式/属性面板（右侧弹框）  
  责任：读取/编辑 cell.style，触发 graph 更新并保证 undo/redo。  
  代表位置：基于 mxGraph API 的实现（在仓库中可能位于 grapheditor 的 Format / EditorUi 模块；本次分析未列出具体 format 实现文件，因此以 API 级别说明实现方式）。

## 五、非功能性要求（NFR）
- 可扩展性：新 Shape 的添加应仅需在 Shapes.js 中注册并在 sidebar 中新增模板。
- 实时性：样式修改需实时反馈画布，修改操作可撤销。
- 可保存性：样式与几何信息保存在 cell（model）中，确保导出/复原。
- 性能：复杂形状的绘制避免高开销计算（建议在 paint 方法考虑缓存或减少重复计算）。

---

附：重要发现的代码位置（摘录）
- Shapes 定义：src/main/webapp/js/grapheditor/Shapes.js （shape 注册与 paint* 方法广泛存在）
- Palette / Sidebar：src/main/webapp/js/grapheditor/Sidebar.js, src/main/webapp/js/diagramly/sidebar/Sidebar-Bootstrap.js, src/main/webapp/js/diagramly/sidebar/Sidebar-Infographic.js
- 渲染器核心说明：src/main/webapp/mxgraph/src/view/mxCellRenderer.js
- UI 控制（sidebar 窗口化）：src/main/webapp/js/diagramly/Minimal.js

（注：上面列举文件位置基于代码静态搜索返回的结果；详细实现接口和函数签名将在详细设计文档中展开。）