# 01 Application Discovery Report

> 分析对象：`BrickWallDesigner_v18.html`  
> 证据状态：**Confirmed**＝源代码已确认；**Inferred**＝依据代码行为推断；**Unknown**＝现有材料无法确认。  
> 本报告只进行 application reverse engineering，不进行数据库、API、CMS 或实现设计。

# 1. Application 基本信息与分析范围

| 项目 | 发现 | 状态与证据 |
|---|---|---|
| Application 名称 | Brick Wall Designer | **Confirmed**：HTML title、启动页及保存对象 `appName`。 |
| HTML 文件名 | `BrickWallDesigner_v18.html` | **Confirmed**：实际源文件。 |
| Application 版本 | 应用自报 **v16**；文件名为 **v18**；历史来源注释指向 **v15** | **Confirmed**：L2、L6、L977、L2781。版本标识不一致，不能擅自认定真实发布版本。 |
| 分析日期 | 2026-09-02（Australia/Sydney） | **Confirmed**：分析环境日期。 |
| 文件信息 | 217,237 bytes；PowerShell 文本统计 5,649 行、17,655 words、211,446 characters；最后修改 2026-09-02T12:17:31.2824377+10:00 | **Confirmed**：文件系统检查。不同换行计数工具可能显示至 L5792。 |
| SHA-256 | `D311909E45DE1C84E2F851B3B77425704A08EE712584508BD70AF695EDA9A550` | **Confirmed**。 |
| 文件完整性 | HTML 有 DOCTYPE、完整 `html/head/body`、闭合 script/body/html，内嵌 CSS/JS 可完整读取 | **Confirmed**（结构完整）；**Unknown**：无法证明其业务发布包未缺文件。 |
| 单文件 application | 是；CSS、JavaScript、SVG 生成逻辑全部内嵌，未发现外部 `src`/`href` 依赖 | **Confirmed**。 |
| 已知缺失文件 | 无代码引用的缺失文件 | **Confirmed**。原注释只说明该文件曾从 Outlook 缓存保存，不构成运行依赖。 |
| 已知缺失信息 | 产品负责人、目标部署浏览器、版本治理、正式价格来源、用户/项目权限、保留策略、真实业务量 | **Unknown**。 |

本次分析包含：HTML/CSS/JavaScript 静态分析；页面、控件、事件、状态和计算路径；JSON Save/Load；数据来源与持久化路径；尝试运行时验证；概念数据发现的应用侧证据。

本次分析不包含：最终数据库表/字段、键实现、PostgreSQL 类型、索引、SQL、migration、CMS schema、API、施工或造价正确性认证、跨浏览器兼容认证、生产安全测试。

分析限制：当前自动化环境没有可用浏览器实例，无法完成 UI 交互、下载/上传往返、Storage/Network 面板检查和打印窗口验证。因此所有动态结果仅能依据代码路径判断，不能提升为 runtime-confirmed。

# 2. 分析方法

| 方法 | 完成情况 | 说明 |
|---|---|---|
| Static Code Analysis | **Completed / Confirmed** | 阅读完整单文件结构，提取控件、函数、常量、状态、保存字段、计算与渲染路径。 |
| Dynamic Runtime Analysis | **Attempted but unavailable / Unknown** | 已尝试连接本地浏览器；环境返回无可用浏览器，未能执行交互。 |
| Scenario-Based Analysis | **Completed statically / Inferred** | 依据事件处理器逐项追踪 New、Draw/Edit、Save、Load、Report、Reset 等场景。 |
| Data Flow Analysis | **Completed / Confirmed** | 追踪 UI → getter/变量 → layout/WALL → costs/report/save。 |
| Persistence Path Analysis | **Completed / Confirmed** | 追踪 `collectProjectState`、`JSON.stringify`、Blob 下载、FileReader、`JSON.parse`、`applyProjectState`。 |
| Traceability Analysis | **Completed / Confirmed** | 重要结论均对应 HTML ID、函数名或代码位置；运行结果相关结论保留为 Unknown。 |

# 3. Application 主要用途和用户任务

主要用途是根据墙体尺寸、砖型、砂浆厚度、砌筑方式、切砖和压顶选项生成双层砖墙示意图、砖数、材料用量、成本估算和可打印报告，并以 JSON 文件保存/恢复输入。**Confirmed**。

目标用户：计划砖墙的业主、估算人员或设计辅助人员是合理推断；代码没有身份、角色或专业资质模型。**Inferred**。应用明确声明图形仅为 schematic、不得用于施工，成本仅供规划。**Confirmed**（L1197、打印报告 disclaimer）。

主要功能区域：

1. Start：New Project；Existing Project 显示 Coming soon。
2. Designer：墙体参数、砖型、bond、切砖、压顶、SVG 墙图、摘要、brick count、zoom、geometry diagnostics。
3. Materials & Costs：计算输入覆盖值、派生材料量、材料/人工成本和总额（含 GST）。
4. Reports：当前墙图、设计摘要、成本摘要、打印/导出到浏览器打印流程。
5. Save Project：项目命名、JSON 下载、JSON 文件加载。

最终结果包括 SVG 示意墙、实际尺寸、砖型数量、材料/成本估算、HTML 打印报告，以及包含可恢复输入的 JSON 项目文件。**Confirmed**。

# 4. Application 工作流程

| 流程 | 触发/UI | JavaScript | 数据影响 | 保存/加载 |
|---|---|---|---|---|
| New Project | 启动页 `NEW PROJECT` | 内联 handler → `showPage(1)` | 仅隐藏启动遮罩并显示 Designer；没有清空状态 | 否 |
| Create Design / Draw | Designer `Draw Wall` | `drawWall` → `calculateLayout` → `renderWall` → `recalculateCosts` | 读取设计输入，重建 `lastLayout`、SVG、brick counts、`WALL`、成本输出 | 不自动保存 |
| Edit Design | 修改任一设计输入后再次 Draw | 同上 | 整体重算；不是局部实体编辑 | 可在后续 Save 保存输入 |
| Add Item | 无独立 UI | 不存在 | 砖图元由布局引擎生成，不是用户添加的持久对象 | 不适用 |
| Remove Item | 无独立 UI | 不存在 | 不适用 | 不适用 |
| Copy Item | 无独立 UI | 不存在 | 不适用 | 不适用 |
| Delete | 无项目/元素删除功能 | 不存在 | 不适用 | 不适用 |
| Materials override | Page 2 数值输入 `u_*` | `oninput=recalculateCosts()`、`userOrDefault` | 空值使用内置默认；包括 0 的非空值覆盖默认；更新派生用量/成本 DOM | 45 个项目值中的相关 34 项被保存 |
| View Report | nav3 | `showPage(3)` → `populateReport` | 克隆当前 SVG，读取 `WALL` 与成本 DOM | 否 |
| Print/Export Report | 两个 `printReport` 按钮 | `printReport` | 组装临时 HTML；`window.open` + `print`，失败时本页 `window.print` | 浏览器打印目标可能保存 PDF；应用无专用 PDF 数据对象 |
| Suggest project name | `Suggest a name` | `suggestProjectName` | 基于实际长×高和 bond 生成 `sp_name` | 项目名进入 Save 外层对象 |
| Save | `Save Project to File` | `saveProject` → `collectProjectState` → `JSON.stringify` → Blob/anchor download | 生成 appName、appVersion、projectName、savedAt、state | 下载 `.json`；无本地数据库 |
| Load / Import | file `sp_load_file` + `Load Project` | `loadProject` → FileReader → `JSON.parse` → `applyProjectState` | 覆盖列入清单的输入与 radio，触发 change，再 draw/recalculate | 从用户选择的 JSON 文件恢复 |
| Reset | nav Start | `showStartScreen` | 仅显示启动遮罩，不重置输入 | 否 |
| Diagnostics | `Run Wall Integrity Test` | `runWallIntegrityTest`、`checkLayoutIntegrity` | 临时快照部分设计输入，批量改变长度/bond/brick，完成后恢复；失败明细仅 DOM 内存 | 否 |
| Zoom/Fit | toolbar | `zoom`、`fitToWindow` | 修改 `currentScale` 并重绘 SVG | 不保存 |

# 5. 外部依赖和数据来源

| 来源 | 代码位置 | 用途 | 可访问 | 完整性影响 |
|---|---|---|---|---|
| 内嵌 HTML/CSS/JS | 全文件；script L2146-L5789 | UI、算法、状态、保存加载 | 是 | 无缺失 |
| 外部 JavaScript/CSS | 未发现 | 无 | 不适用 | 无 |
| JSON 项目导入文件 | `sp_load_file` L2117-L2127；`loadProject` L2802-L2835 | 恢复项目输入 | 结构可由代码确认；没有样本文件 | 运行往返未验证 |
| JSON/CSV/配置文件 | 除用户项目 JSON 外未发现 | 无 | 不适用 | 无 |
| API/fetch/XHR | 未发现 | 无 | 不适用 | 无 |
| 图片/产品资源 | 未发现 URL 或图片文件；墙图由 SVG 动态生成 | 视觉输出 | 是（代码） | 无 |
| LocalStorage | 未发现 | 无 | 不适用 | 无 |
| SessionStorage | 未发现 | 无 | 不适用 | 无 |
| IndexedDB | 未发现 | 无 | 不适用 | 无 |
| URL 参数 | 未发现 `URLSearchParams`/`location.search` | 无 | 不适用 | 无 |
| 浏览器时间 | `new Date().toISOString()` L2783；加载时 locale 格式化 | 保存时间与显示 | 运行环境提供 | 时区/时钟可信度未验证 |
| Blob/Object URL/下载 | L2787-L2797 | 将 JSON 交给浏览器下载 | 代码可访问 | 实际下载未验证 |
| 打印窗口 | L2552-L2587 | HTML 报告与浏览器打印 | 代码可访问 | popup/print 策略影响运行 |
| Outlook cache provenance | L2 注释中的 v15 file URL | 文件来源线索，不在运行时读取 | 原位置不可确认 | 版本溯源存在疑问 |

# 6. 页面输入和 UI 控件盘点

以下均为 **Confirmed**。Save 列：`V`＝进入 `state.values`；`R`＝进入 `state.radios`；`N`＝不保存；外层＝保存对象顶层。

## 6.1 Designer 与诊断控件

| 控件/ID或name | 类型；数据类型；示例/允许值 | JS / Event | Save | 证据 |
|---|---|---|---|---|
| `WL` Length | number；mm；1235；100..50000 | `getWallLength`；Draw 时读取 | V | L1024、L2205、L2658 |
| `WH` Height | number；mm；1750；100..10000 | `getWallHeight` | V | L1027、L2206、L2659 |
| `MJ` Mortar | number；mm；10；0..30 | `getMortarThickness`（0 会回退 10） | V | L1030、L2200、L2657 |
| `BT` Brick Type | select；code；s/l/u/m/r | `getBrick`/`getBrickLabel` | V | L1035-L1041、L2199 |
| `cu` Allow cuts | radio；y/n；默认 n | `getCutsAllowed`；change 改 `AR` opacity | R | L1048-L1050、L2201-L2227 |
| `ad` Adjust direction | radio；s/l；默认 s | `getLengthAdjust` | R | L1055-L1058 |
| `bond` | radio；7 值；默认 stretcher | `getBondType`、layout/min length | R | L1063-L1076 |
| `cc` Include capping | radio；y/n；默认 n | `getHasCapping`；change 显隐 options | R | L1083-L1085、L2210-L2223 |
| `cco` Orientation | radio；flat/vertical/soldier | `getCappingOrientation` | R | L1091-L1096 |
| `ccoff` End offset | radio；y/n；默认 n | `getCappingOffset` | R | L1102-L1104 |
| `sk_hidden` | hidden；integer-like string；2 | `getSkinCount` 实际固定返回 2；字段未读取 | V | L1113、L2207、L2667 |
| `C1,C2,C6,C3,C4,C5,CHR` | hidden；hex colour；如 `#C1440E` | `renderWall` colour map | V | L1115-L1121、L4997-L5008 |
| `DIAG_MIN/MAX/STEP/MORTAR` | number；600/10000/1/17 | `runWallIntegrityTest` | N | L928-L940、L4830-L4836 |
| `DIAG_ALLBONDS/ALLBRICKS/CUTS_ONLY` | checkbox；boolean；cuts 默认 true | `runWallIntegrityTest` | N | L945-L959、L4834-L4836 |
| Draw button | button | `drawWall()` | N | L1123 |
| Diagnostic button/modal controls | buttons | `openDiagModal`,`closeDiagModal`,`runWallIntegrityTest`,`loadFailingWall` | N | L911、L961、L1124、L4951 |
| Zoom +/−/Fit | buttons | `zoom(1.25)`,`zoom(0.8)`,`fitToWindow` | N | L1193-L1196 |

## 6.2 Materials & Costs 输入（均为 number、`oninput=recalculateCosts()`、空值使用代码默认、进入 Save V）

| ID | 业务名称；单位 | 默认值（代码） | 来源 |
|---|---|---:|---|
| `u_side_clear` | side clearance；mm | 1500 | L5552 |
| `u_end_clear` | end clearance；mm | 2000 | L5553 |
| `u_footing_ext` | footing extension；mm | 100 | L5566 |
| `u_footing_ratio` | wall height : footing depth ratio | 4 | L5573 |
| `u_mesh_inset` | mesh inset；mm | 50 | L5583 |
| `u_mesh_layers` | mesh layers；count | 2 | L5591 |
| `u_roadbase_depth` | road base depth；mm | 100 | L5594 |
| `u_compact_factor` | compaction；percent，HTML 0..100 | 10 | L5602 |
| `u_steel_len` | reinforcing steel bar length；m | 8 | L5611 |
| `u_brick_contingency` | brick contingency；percent | 10 | L5619 |
| `u_mortar_contingency` | mortar contingency；percent | 10 | L5635 |
| `u_mix_cement` | mortar mix cement part | 1 | L5643 |
| `u_mix_lime` | mortar mix lime part | 0 | L5644 |
| `u_mix_sand` | mortar mix sand part | 4 | L5645 |
| `u_cost_rb` | road base；$/m³ | 50 | L5667 |
| `u_cost_steel` | reinforcing steel；$/length | 18 | L5668 |
| `u_cost_conc` | concrete；$/m³ | 400 | L5669 |
| `u_cost_brick` | bricks；$/brick | 2.00 | L5670 |
| `u_cost_cement` | cement；$/kg | 0.50 | L5671 |
| `u_cost_lime` | lime；$/kg | 1.10 | L5672 |
| `u_cost_sand` | sand；$/kg | 0.10 | L5673 |
| `u_allow_site` | site access allowance；$ | 0 | L5701 |
| `u_allow_waste` | waste disposal allowance；$ | 0 | L5702 |
| `u_lab_prepare` | prepare area；$/m² | 0 | L5703 |
| `u_lab_trench` | trench excavation；$/m³ | 50 | L5704 |
| `u_lab_compact` | road base compaction；$/m² | 20 | L5705 |
| `u_lab_pour` | concrete pour；$/m² | 50 | L5706 |
| `u_lab_lay` | bricklaying；$/brick | 2 | L5707 |
| `u_min_prepare` | prepare minimum；$ | 0 | L5709 |
| `u_min_trench` | trench minimum；$ | 500 | L5710 |
| `u_min_compact` | compaction minimum；$ | 0 | L5711 |
| `u_min_pour` | pour minimum；$ | 500 | L5712 |
| `u_min_lay` | bricklaying minimum；$ | 1000 | L5713 |

这些控件的 HTML `min` 均为 0（`u_compact_factor` 另有 max=100；若干货币项 step=0.01）。**Confirmed**。页面显示值 `cv_*`、`rp_*` 是动态输出而非输入，不参与 Save。

## 6.3 Save/Load 与导航控件

| 控件 | 类型；示例 | JS/Event | Save | 证据 |
|---|---|---|---|---|
| `sp_name` | text；示例 “Foley Residence …” | `suggestProjectName`,`saveProject`; Load 后赋值 | 外层 `projectName` | L2052-L2065、L2771-L2785、L2814 |
| `sp_load_file` | upload；accept JSON | `loadProject`/FileReader | N | L2117-L2129 |
| Save button | button | `saveProject()` | 执行保存 | L2085-L2087 |
| Load button | button | `loadProject()` | 执行加载 | L2128-L2130 |
| `nav0..nav4` | buttons | `showStartScreen`,`showPage(1..4)` | N | L999-L1013 |
| Print buttons（2） | buttons | `printReport()` | N | 静态控件盘点；L2484 起 |

未发现 slider、toggle 专用元素、textarea、动态生成的输入字段、item add/remove/copy 控件。条件显示字段包括 `AR`（仅 opacity）及 `CC_OPTIONS`（display）。

# 7. JavaScript 数据对象盘点

| 名称 | 类型/示例 | 创建、读取、修改和函数 | Save/Load | 证据 |
|---|---|---|---|---|
| `BRICK_TYPES` | object map；`s:{l:230,w:110,h:76}`，共5项 | L2150 创建；`getBrick`、layout、diagnostics、minLength 读取；运行时不修改 | 间接：只保存选择 code，不保存此表 | L2150-L2156 |
| `BRICK_NAMES` | object map；s→Standard 等5项 | L2157 创建；labels/diagnostics 读取 | 否 | L2157-L2163 |
| `NAMES` | object map；7 个 bond code→label | L5158 创建；报告、建议名称、验证提示读取 | 只保存 bond code | L5158-L5166 |
| `ALL_BOND_VALUES` | array；7 bond codes | diagnostics 遍历 | 否 | L4752-L4760 |
| `PROJECT_VALUE_IDS` | array；45 个 element IDs | collect/apply 对称遍历 | 定义 values 保存/加载范围 | L2655-L2701 |
| `PROJECT_RADIO_GROUPS` | array；6 names | collect/apply 对称遍历 | 定义 radios 保存/加载范围 | L2702 |
| `WALL` | mutable application state；length/height/.../ready | L2183 创建；`drawWall` L5473-L5500 更新；cost/report 读取 | 不直接保存；Load 后通过 draw 重建 | L2183-L2195 |
| `lastLayout` | object/null | `drawWall` 赋布局；zoom/fit/render 读取 | 否；可重算 | L2178、L5764-L5773 |
| `currentScale` | number；1 | zoom/fit 修改，render 使用 | 否；临时 UI state | L2178-L2179 |
| `layout` | 复杂计算对象 | `calculateLayout` 返回；包含 `wallBricks`,`ccBricks`,`aL`,`eH`,`m`,`totalH` 等 | 否；可由输入重算 | L2948 起、L4740-L4742 |
| brick geometry records | object；`{x1,x2,y1,y2,t,...}` | row/capping builders 创建；render/count/integrity 读取 | 否 | `drawStretcherRow`,`drawHeaderCourseRow`,`buildCappingBricks` |
| `counts` | object；`o/e/h/s/c/cc/ccc` | draw 时从 layout 图元累计 | 否；派生 | L5408-L5414 |
| `countRows` | array of `{label,s1,s2}` | 砖数摘要表动态创建 | 否 | L5420-L5440 |
| `state` | `{values:{},radios:{}}` | `collectProjectState` 创建；`applyProjectState` 读取 | 是，项目核心 state | L2704-L2744 |
| `project` | `{appName,appVersion,projectName,savedAt,state}` | `saveProject` 创建；Load 解析并部分读取 | 是，JSON 根对象 | L2779-L2785 |
| diagnostics `snap` | object；WL/WH/MJ/BT/bond/cuts/cc | 测试前创建、测试后恢复 | 否 | L4846-L4854、L4901-L4909 |
| diagnostics `failures` | array of failure objects | integrity check/sweep 创建；results table 读取 | 否 | L4778-L4825、L4872-L4914 |
| `geomMinCache` | object map；key `bond|brick|mortar` | `getGeomMinLength` 填充/读取 | 否 | L5240-L5255 |
| `colorMap` | object；brick category→hidden colour | `renderWall` 每次创建 | 否（源颜色保存） | L4997-L5008 |
| cost locals | numbers；workAreaM2、footingVolume、bricksToOrder、grandTotal 等 | `recalculateCosts` 每次创建、转换并写 DOM | 否；可重算 | L5534-L5755 |
| default/preset constants | 砖尺寸、颜色、成本、施工参数、密度 1440/500/1600、GST=`total/11` | HTML/代码常量 | 参考/配置，未独立版本化 | L2150-L2163、L5514-L5754 |
| sample/test data | 诊断默认 600..10000、17；项目名示例 | 仅 UI 示例/测试参数 | 否 | L928-L959、L2055 |

`lastLayout` 的所有内部字段由多分支算法构造，属于运行时派生几何，不应在本阶段直接转成数据库字段。

# 8. Application 持久化行为概览

- Save：`saveProject` 要求非空项目名；`collectProjectState` 收集 45 个 value ID 和 6 个 radio group；创建包含元数据的 `project`；`JSON.stringify(project,null,2)`；Blob MIME `application/json`；临时 Object URL；模拟 anchor 下载。**Confirmed**。
- Load/Import：用户选择 `.json`；FileReader `readAsText`；`JSON.parse`；只把 `project.state` 传给 `applyProjectState`；项目名和保存时间用于 UI；随后 `drawWall`、`recalculateCosts`。**Confirmed**。
- 对称性：values 和 radios 通过相同清单收集/应用，字段级对称。**Confirmed**。但根对象 `appName`/`appVersion` 在加载时不校验、不使用；`savedAt` 只显示；未知字段忽略；缺字段保留当前 UI 值。**Confirmed**。
- Auto-save、LocalStorage、SessionStorage、IndexedDB、API request/response：未发现。**Confirmed**。
- 历史兼容：没有 schemaVersion、migration 或版本分支；`appVersion` 仅写出。缺字段容忍是弱兼容行为，但未被明确声明为版本策略。**Confirmed/Inferred**。
- 未持久化：`WALL`、layout/SVG、派生成本、诊断结果、zoom、当前 page、折叠状态、诊断控件、文件 input、保存/加载状态文本。它们大多可重算或是 UI 临时态。**Confirmed**。
- 潜在缺陷：Save 宣称保存“every input”，但 `sp_name` 位于根对象而非 state；诊断输入不保存；这是设计范围差异而非必然 bug。`sk_hidden` 被保存但计算 getter 固定返回 2。**Confirmed**。

# 9. 运行时验证

运行时浏览器连接已尝试，但环境没有可用浏览器实例。故以下测试均未能执行：

| 测试场景 | 计划步骤 | 实际结果 | 静态预期 |
|---|---|---|---|
| 创建空白设计 | New Project → Designer | **Unknown：未执行** | 仅进入现有默认值页面；不会清空状态 |
| 修改主要输入 | 修改 WL/WH/MJ/BT/bond | **Unknown：未执行** | 点击 Draw 后整体重算 |
| 添加设计元素 | 查找 Add | **Not applicable（静态确认无控件）** | 砖图元由算法批量生成 |
| 修改设计元素 | 修改单砖 | **Not applicable** | 不支持单砖编辑 |
| 删除设计元素 | 查找 Delete | **Not applicable** | 不支持单砖/项目删除 |
| 保存/导出 | 命名后下载 JSON；打印报告 | **Unknown：未执行** | 生成 JSON Blob；打印窗口生成 HTML |
| 重新加载/导入 | 选择刚下载 JSON | **Unknown：未执行** | 恢复 state，重绘并重算 |
| 前后数据比较 | 比较 JSON/UI/WALL | **Unknown：未执行** | 清单字段应对称，派生数据应重算 |
| Storage/Network | 检查 Local/Session/IDB/network | **Unknown：未执行** | 静态代码预期均无应用持久化/请求 |

**当前结论仅来自静态代码分析，尚未完成运行时验证。**

# 10. Application 报告结论

已确认：4 页主流程、双层墙设计参数、5 种砖型、7 种 bond、切砖和压顶逻辑、SVG/砖数/材料成本派生、诊断扫描、打印报告、JSON 文件保存和加载。

已确认的数据来源：HTML defaults、JavaScript reference/config constants、用户 UI 输入、本地导入 JSON、浏览器时间；无 API、外部配置、LocalStorage、SessionStorage 或 IndexedDB。

无法访问的依赖：没有代码依赖缺失；但浏览器运行环境不可用，下载、文件导入、打印、DOM 实际行为和网络/Storage 面板未验证。

未理解功能：没有未识别函数模块；算法工程正确性和业务正确性未认证。未映射 UI 控件：0 个业务输入（动态输出另列为派生显示）；未映射 JavaScript 持久对象：0 个已发现的持久候选；大量局部几何/计算变量已按派生类归组而非逐变量建模。

**分析完整性结论：Static analysis complete, runtime verification missing**。
