# 03 Data Field Dictionary

## 1. 文档范围

- 分析对象：`BrickWallDesigner_v18.html`；证据为 HTML 控件、内嵌 JavaScript、Save/Load JSON 与计算函数。
- 字段按应用当前实际行为记录；Recommended business name 仅为可读候选名，不代表数据库列名。
- 数值在当前 JSON 中实际以 DOM string 保存，Data type 表示业务语义类型。
- 未完成浏览器运行时往返验证；代码无法确认的语义标为 Unknown。

## 2. 持久化字段字典

- `Required` 表示 Save JSON 是否始终写出该 key；“空”指空字符串，非数据库 NULL。
- 该字典回答：用户创建这个项目的时候输入了什么，关闭后还要恢复什么？

| Source path | Current key | Recommended business name | 中文含义 | Data type | Unit | Required | Nullable/default semantics | Allowed values/range | Example/default | Source evidence | Confidence | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| project.appName | appName | application_name | 应用名称 | string | N/A | Yes | 固定非空 | Brick Wall Designer | Brick Wall Designer | `saveProject` L2779-2785 | Confirmed | Load 不校验 |
| project.appVersion | appVersion | application_version | 保存应用版本 | string | N/A | Yes | 固定非空 | 当前写死 v16 | v16 | `saveProject` L2781 | Confirmed | 文件名为v18，版本冲突 |
| project.projectName | projectName | project_name | 项目名称 | string | N/A | Yes | trim后不可空；零不适用 | 非空；文件名另截至120字符 | 2335×1750 Stretcher Bond | `sp_name`; `saveProject` | Confirmed | JSON名称本身未截长 |
| project.savedAt | savedAt | saved_at | 保存时间 | datetime | N/A | Yes | 浏览器时钟生成，不为空 | ISO-8601 string | 2026-09-02T02:00:00.000Z | `new Date().toISOString()` L2783 | Confirmed | 时钟可信度未知 |
| project.state | state | project_state | 项目输入容器 | string | N/A | Yes | 固定对象；无默认替代 | values + radios | object | `collectProjectState` | Confirmed | 结构节点，非业务标量 |
| state.values | values | input_values | value输入容器 | string | N/A | Yes | 固定对象 | 45个白名单key | object | `PROJECT_VALUE_IDS` | Confirmed | 结构节点 |
| state.values.BT | BT | brick_type_code | 砖型代码 | enum | N/A | Yes | 默认s；空/未知会使lookup失败 | s,l,u,m,r | s | `BT`; `getBrick` | Confirmed | 参考第3节 |
| state.values.MJ | MJ | mortar_joint_thickness | 砂浆缝厚 | decimal | mm | Yes | 空、0、NaN计算时均回退10 | HTML 0..30 | 10 | `MJ`; `getMortarThickness` L2200 | Confirmed | 0虽允许但不生效 |
| state.values.WL | WL | entered_wall_length | 输入墙长 | decimal | mm | Yes | 空、0、NaN回退1235 | HTML 100..50000 | 1235 | `WL`; `getWallLength` | Confirmed | 实际墙长可调整 |
| state.values.WH | WH | entered_wall_height | 输入墙高 | decimal | mm | Yes | 空、0、NaN回退1750 | HTML 100..10000 | 1750 | `WH`; `getWallHeight` | Confirmed | — |
| state.values.C1 | C1 | odd_stretcher_colour | 奇数皮砖颜色 | string | N/A | Yes | 固定非空默认；Load可覆盖 | hex colour（未验证） | #C1440E | `C1`; `renderWall` | Confirmed | hidden |
| state.values.C2 | C2 | even_stretcher_colour | 偶数皮砖颜色 | string | N/A | Yes | 同上 | hex colour（未验证） | #A63A0C | `C2`; `renderWall` | Confirmed | hidden |
| state.values.C3 | C3 | special_brick_colour | 特殊砖颜色 | string | N/A | Yes | 同上 | hex colour（未验证） | #8B6914 | `C3`; `renderWall` | Confirmed | hidden |
| state.values.C4 | C4 | cut_brick_colour | 切砖颜色 | string | N/A | Yes | 同上 | hex colour（未验证） | #AE0000 | `C4`; `renderWall` | Confirmed | hidden |
| state.values.C5 | C5 | mortar_colour | 砂浆颜色 | string | N/A | Yes | 同上 | hex colour（未验证） | #D4C8B0 | `C5`; `renderWall` | Confirmed | hidden |
| state.values.C6 | C6 | header_brick_colour | 丁砖颜色 | string | N/A | Yes | 同上 | hex colour（未验证） | #8B3A0C | `C6`; `renderWall` | Confirmed | hidden |
| state.values.CHR | CHR | capping_brick_colour | 压顶砖颜色 | string | N/A | Yes | 同上 | hex colour（未验证） | #710000 | `CHR`; `renderWall` | Confirmed | hidden |
| state.values.sk_hidden | sk_hidden | wall_skin_count | 墙体层数 | integer | count | Yes | 保存值可改；计算固定返回2 | 实际仅2 | 2 | `sk_hidden`; `getSkinCount` | Confirmed | 保存值非2也不生效 |
| state.values.u_cost_rb | u_cost_rb | road_base_unit_cost | 路基层材料单价 | decimal | currency | Yes | 空=代码默认50；0=明确零价 | HTML ≥0 | 空→50 | `userOrDefault` L5667 | Confirmed | 每m³ |
| state.values.u_cost_steel | u_cost_steel | steel_length_unit_cost | 钢筋每根/长度单价 | decimal | currency | Yes | 空=18；0=明确零价 | ≥0 | 空→18 | L5668 | Confirmed | “length”定义未知 |
| state.values.u_cost_conc | u_cost_conc | concrete_unit_cost | 混凝土单价 | decimal | currency | Yes | 空=400；0=明确零价 | ≥0 | 空→400 | L5669 | Confirmed | 每m³ |
| state.values.u_cost_brick | u_cost_brick | brick_unit_cost | 单砖价格 | decimal | currency | Yes | 空=2；0=明确零价 | ≥0 | 空→2.00 | L5670 | Confirmed | 每brick |
| state.values.u_cost_cement | u_cost_cement | cement_unit_cost | 水泥单价 | decimal | currency | Yes | 空=0.5；0=明确零价 | ≥0 | 空→0.50 | L5671 | Confirmed | 每kg |
| state.values.u_cost_lime | u_cost_lime | lime_unit_cost | 石灰单价 | decimal | currency | Yes | 空=1.1；0=明确零价 | ≥0 | 空→1.10 | L5672 | Confirmed | 每kg |
| state.values.u_cost_sand | u_cost_sand | sand_unit_cost | 砂单价 | decimal | currency | Yes | 空=0.1；0=明确零价 | ≥0 | 空→0.10 | L5673 | Confirmed | 每kg |
| state.values.u_allow_site | u_allow_site | site_access_allowance | 场地通行预留费 | decimal | currency | Yes | 空=0；0=明确零额 | ≥0 | 空→0 | L5701 | Confirmed | 固定金额 |
| state.values.u_allow_waste | u_allow_waste | waste_disposal_allowance | 废料处理预留费 | decimal | currency | Yes | 空=0；0=明确零额 | ≥0 | 空→0 | L5702 | Confirmed | 固定金额 |
| state.values.u_lab_prepare | u_lab_prepare | work_area_preparation_rate | 工作区准备人工费率 | decimal | currency | Yes | 空=0；0=明确零费率 | ≥0 | 空→0 | L5703 | Confirmed | 每m² |
| state.values.u_lab_trench | u_lab_trench | trench_excavation_rate | 沟槽开挖人工费率 | decimal | currency | Yes | 空=50；0=明确零费率 | ≥0 | 空→50 | L5704 | Confirmed | 每m³ |
| state.values.u_lab_compact | u_lab_compact | road_base_compaction_rate | 路基层压实人工费率 | decimal | currency | Yes | 空=20；0=明确零费率 | ≥0 | 空→20 | L5705 | Confirmed | 每m² |
| state.values.u_lab_pour | u_lab_pour | concrete_pour_rate | 混凝土浇筑人工费率 | decimal | currency | Yes | 空=50；0=明确零费率 | ≥0 | 空→50 | L5706 | Confirmed | 每m²（按代码） |
| state.values.u_lab_lay | u_lab_lay | bricklaying_rate | 砌砖人工费率 | decimal | currency | Yes | 空=2；0=明确零费率 | ≥0 | 空→2 | L5707 | Confirmed | 每brick |
| state.values.u_min_prepare | u_min_prepare | preparation_minimum_charge | 工作区准备最低费 | decimal | currency | Yes | 空=0；0=明确零额 | ≥0 | 空→0 | L5709 | Confirmed | — |
| state.values.u_min_trench | u_min_trench | trench_minimum_charge | 沟槽开挖最低费 | decimal | currency | Yes | 空=500；0=明确零额 | ≥0 | 空→500 | L5710 | Confirmed | — |
| state.values.u_min_compact | u_min_compact | compaction_minimum_charge | 压实最低费 | decimal | currency | Yes | 空=0；0=明确零额 | ≥0 | 空→0 | L5711 | Confirmed | — |
| state.values.u_min_pour | u_min_pour | concrete_pour_minimum_charge | 浇筑最低费 | decimal | currency | Yes | 空=500；0=明确零额 | ≥0 | 空→500 | L5712 | Confirmed | — |
| state.values.u_min_lay | u_min_lay | bricklaying_minimum_charge | 砌砖最低费 | decimal | currency | Yes | 空=1000；0=明确零额 | ≥0 | 空→1000 | L5713 | Confirmed | — |
| state.values.u_footing_ext | u_footing_ext | footing_extension | 基础外伸量 | decimal | mm | Yes | 空=100；0=明确零 | ≥0 | 空→100 | L5566 | Confirmed | — |
| state.values.u_footing_ratio | u_footing_ratio | wall_height_to_footing_depth_ratio | 墙高与基础深度比 | decimal | N/A | Yes | 空=4；0会导致除零 | ≥0（有效应>0） | 空→4 | L5573-5574 | Confirmed | 有效范围需确认 |
| state.values.u_roadbase_depth | u_roadbase_depth | road_base_depth | 路基层深度 | decimal | mm | Yes | 空=100；0=明确零 | ≥0 | 空→100 | L5594 | Confirmed | — |
| state.values.u_compact_factor | u_compact_factor | compaction_factor | 压实比例 | decimal | percent | Yes | 空=10；0=明确0% | HTML 0..100；100导致除零 | 空→10 | L5602-5603 | Confirmed | 有效上限应<100 |
| state.values.u_mesh_layers | u_mesh_layers | mesh_layer_count | 钢筋网层数 | integer | count | Yes | 空=2；0=明确无网层 | ≥0 | 空→2 | L5591 | Inferred | parseFloat未强制整数 |
| state.values.u_mesh_inset | u_mesh_inset | mesh_inset | 钢筋网内缩 | decimal | mm | Yes | 空=50；0=明确零 | ≥0 | 空→50 | L5583 | Confirmed | — |
| state.values.u_steel_len | u_steel_len | steel_bar_length | 单根钢筋长度 | decimal | m | Yes | 空=8；0会导致除零 | ≥0（有效应>0） | 空→8 | L5611-5613 | Confirmed | — |
| state.values.u_mix_cement | u_mix_cement | mortar_cement_part | 砂浆水泥配比份 | decimal | count | Yes | 空=1；0=明确0份 | ≥0 | 空→1 | L5643 | Confirmed | 三项总和0则质量均0 |
| state.values.u_mix_lime | u_mix_lime | mortar_lime_part | 砂浆石灰配比份 | decimal | count | Yes | 空=0；0=明确0份 | ≥0 | 空→0 | L5644 | Confirmed | — |
| state.values.u_mix_sand | u_mix_sand | mortar_sand_part | 砂浆砂配比份 | decimal | count | Yes | 空=4；0=明确0份 | ≥0 | 空→4 | L5645 | Confirmed | — |
| state.values.u_mortar_contingency | u_mortar_contingency | mortar_contingency_percent | 砂浆损耗比例 | decimal | percent | Yes | 空=10；0=无损耗 | ≥0 | 空→10 | L5635-5636 | Confirmed | — |
| state.values.u_brick_contingency | u_brick_contingency | brick_contingency_percent | 砖损耗比例 | decimal | percent | Yes | 空=10；0=无损耗 | ≥0 | 空→10 | L5619-5620 | Confirmed | — |
| state.values.u_end_clear | u_end_clear | work_area_end_clearance | 工作区端部净距 | decimal | mm | Yes | 空=2000；0=明确零 | ≥0 | 空→2000 | L5553 | Confirmed | 两端各使用一次 |
| state.values.u_side_clear | u_side_clear | work_area_side_clearance | 工作区侧向净距 | decimal | mm | Yes | 空=1500；0=明确零 | ≥0 | 空→1500 | L5552 | Confirmed | 两侧各使用一次 |
| state.radios.bond | bond | bond_type_code | 砌筑方式 | enum | N/A | Yes | 默认stretcher；无选中则不写 | 7项，见第3节 | stretcher | radio `bond`; `getBondType` | Confirmed | — |
| state.radios.cu | cu | cut_bricks_allowed | 是否允许切砖 | boolean | N/A | Yes | y=true；n=false；默认n | y,n | n | radio `cu`; `getCutsAllowed` | Confirmed | JSON实际存enum string |
| state.radios.ad | ad | no_cut_length_adjustment | 禁切砖时长度调整方向 | enum | N/A | Yes | 默认s；cu=y时保留但不主导 | s,l | s | radio `ad`; `getLengthAdjust` | Confirmed | 条件生效 |
| state.radios.cc | cc | capping_included | 是否含压顶层 | boolean | N/A | Yes | y=true；n=false；默认n | y,n | n | radio `cc`; `getHasCapping` | Confirmed | — |
| state.radios.cco | cco | capping_orientation | 压顶砖方向 | enum | N/A | Yes | 默认flat；cc=n时仍保存但不生效 | flat,vertical,soldier | flat | radio `cco`; `getCappingOrientation` | Confirmed | 条件生效 |
| state.radios.ccoff | ccoff | capping_end_offset | 压顶端砖是否错位 | boolean | N/A | Yes | y=true；n=false；默认n；cc=n时不生效 | y,n | n | radio `ccoff`; `getCappingOffset` | Confirmed | — |

## 3. 参考数据字典
- 用户可以从哪些标准选项中进行选择

| Data set | Code | Label | Dimensions/value | Unit | Source evidence | Confidence |
|---|---|---|---|---|---|---|
| Brick Types | s | Standard | 230×110×76 | mm | `BRICK_TYPES`,`BRICK_NAMES` L2150-2163 | Confirmed |
| Brick Types | l | Slimline | 290×90×47 | mm | 同上 | Confirmed |
| Brick Types | u | Utility | 290×90×76 | mm | 同上 | Confirmed |
| Brick Types | m | Modular | 190×90×90 | mm | 同上 | Confirmed |
| Brick Types | r | Roman | 290×90×40 | mm | 同上 | Confirmed |
| Bond Types | stretcher | Stretcher Bond | code | N/A | `bond` radios; `NAMES` L5158-5166 | Confirmed |
| Bond Types | header | Header Bond | code | N/A | 同上 | Confirmed |
| Bond Types | english | English Bond | code | N/A | 同上 | Confirmed |
| Bond Types | englishGardenWall | English Garden Wall Bond | code | N/A | 同上 | Confirmed |
| Bond Types | flemish | Flemish Bond | code | N/A | 同上 | Confirmed |
| Bond Types | flemishGardenWall | Flemish Garden Wall Bond | code | N/A | 同上 | Confirmed |
| Bond Types | common | Common Bond (American) | code | N/A | 同上 | Confirmed |
| Capping orientations | flat | Flat (header on flat) | unitW=brick width; rowH=brick height | mm | `cco`; `cappingDimensions` L2230 | Confirmed |
| Capping orientations | vertical | Vertical (header on edge) | unitW=brick height; rowH=brick width | mm | 同上 | Confirmed |
| Capping orientations | soldier | Soldier (stretcher on end) | unitW=brick height; rowH=brick length | mm | 同上 | Confirmed |
| Adjustment directions | s | Shorter | code | N/A | `ad` radios L1055-1058 | Confirmed |
| Adjustment directions | l | Longer | code | N/A | 同上 | Confirmed |
| Yes/No flags | y | Yes/true | cu,cc,ccoff | N/A | corresponding radios/getters | Confirmed |
| Yes/No flags | n | No/false | cu,cc,ccoff | N/A | corresponding radios/getters | Confirmed |
| Brick render categories | o/e/h/s/c/cc/ccc | odd/even/header/special/cut/capping/capping-cut | render/count codes | N/A | `colorMap`,`counts` L4999,L5408 | Confirmed |

## 4. 计算配置字段字典

- “用户覆盖”均为：UI空值使用代码默认，输入0为明确覆盖；这些 override 已逐项列于第2节。
- 应该使用哪些参数、价格、规则来进行计算，他们控制计算结果

| Field | 当前形态 | Default/rule | Unit | Source evidence | Confidence |
|---|---|---:|---|---|---|
| entered_wall_length/height | 用户输入+HTML默认 | 1235 / 1750 | mm | `WL`,`WH`; getters | Confirmed |
| mortar_joint_thickness | 用户输入+HTML默认 | 10 | mm | `MJ`; `getMortarThickness` | Confirmed |
| wall_skin_count | 硬编码 | 2 | count | `getSkinCount`; WALL width | Confirmed |
| side/end clearance | 用户覆盖 | 1500 / 2000 | mm | L5552-5553 | Confirmed |
| footing extension/ratio | 用户覆盖 | 100 / 4 | mm / N/A | L5566,L5573 | Confirmed |
| mesh inset/layers | 用户覆盖 | 50 / 2 | mm / count | L5583,L5591 | Confirmed |
| road base depth/compaction | 用户覆盖 | 100 / 10 | mm / percent | L5594,L5602 | Confirmed |
| steel bar length | 用户覆盖 | 8 | m | L5611 | Confirmed |
| brick/mortar contingency | 用户覆盖 | 10 / 10 | percent | L5619,L5635 | Confirmed |
| mortar cement/lime/sand parts | 用户覆盖 | 1 / 0 / 4 | count | L5643-5646 | Confirmed |
| cement/lime/sand density | 硬编码 | 1440 / 500 / 1600 | kg/m³ | L5648-5650 | Confirmed |
| road base/steel/concrete/brick costs | 用户覆盖 | 50 / 18 / 400 / 2 | currency | L5667-5670 | Confirmed |
| cement/lime/sand costs | 用户覆盖 | 0.5 / 1.1 / 0.1 | currency | L5671-5673 | Confirmed |
| site/waste allowances | 用户覆盖 | 0 / 0 | currency | L5701-5702 | Confirmed |
| prepare/trench/compact/pour/lay rates | 用户覆盖 | 0 / 50 / 20 / 50 / 2 | currency | L5703-5707 | Confirmed |
| prepare/trench/compact/pour/lay minimums | 用户覆盖 | 0 / 500 / 0 / 500 / 1000 | currency | L5709-5713 | Confirmed |
| mortar face assumption | 硬编码规则 | bottom + 50% long side + one end | N/A | L5623-5634 | Confirmed |
| labour minimum rule | 硬编码规则 | max(calculated charge, minimum) | currency | L5715-5731 | Confirmed |
| GST included component | 硬编码规则 | grand total / 11 | currency | L5750-5754 | Confirmed |
| geometry tolerance | 硬编码规则 | 0.5 | mm | `validateLayoutGeometry` L5188-5207 | Confirmed |
| next-valid-length search | 硬编码规则 | 5mm step; +3000mm cap | mm | `findNextValidLength` L5227-5237 | Confirmed |

## 5. 派生输出字典
- 回答的问题是：应该根据输入计算出了什么？

| Derived output | 中文含义/单位 | 计算来源 | 可重算 | 当前保存 |
|---|---|---|---|---|
| WALL.length / height / width | 实际墙长、高、双层墙宽；mm | layout + brick width + mortar | Yes | No |
| actual course count / capping height | 墙皮数、压顶高度；count/mm | wall height、brick height、mortar、orientation | Yes | No |
| stretchers/headers/specials/cuts/cappingBricks | 各类砖数；count | layout brick categories与倍数 | Yes | No |
| WALL.total / bricksToOrder | 设计总砖数/含损耗订购数；count | 分类合计；`ceil(total×(1+contingency))` | Yes | No |
| workAreaM2 | 工作面积；m² | 墙长宽+端/侧净距 | Yes | No |
| wallHorizontal/VerticalFootprint | 墙水平/垂直投影；m² | WALL dimensions | Yes | No |
| footingFootprint/depth/volume | 基础面积、深度、体积；m²/mm/m³ | 墙尺寸、外伸、深度比 | Yes | No |
| meshWidth/Length/Layers | 钢筋网宽、长、层数；mm/count | 基础尺寸、内缩、层数 | Yes | No |
| excavationVolume | 开挖体积；m³ | 基础面积×(基础深+路基深) | Yes | No |
| roadBaseVolume/originalVolume | 压实后/压实前路基体积；m³ | 路基深与压实比例 | Yes | No |
| steelBarCount | 钢筋根数；count | `ceil(mesh total length/bar length)` | Yes | No |
| mortarVolume/totalVolume | 净砂浆/含损耗砂浆体积；m³ | 砖数、砖尺寸、缝厚、损耗 | Yes | No |
| cement/lime/sand mass | 各砂浆材料质量；kg | 总砂浆×配比×密度 | Yes | No |
| materialRoadBase/Steel/Concrete/Bricks/Cement/Lime/Sand | 七项材料成本；currency | 数量×对应单价 | Yes | No |
| materialsTotal | 材料成本合计；currency | 七项材料成本求和 | Yes | No |
| labourSite/Waste | 场地、废料固定预留；currency | 用户值或默认 | Yes | No |
| labourPrepare/Trench/Compact/Pour/Lay | 五项人工成本；currency | 工程量×费率，与最低费取大 | Yes | No |
| labourTotal | 人工及预留合计；currency | 七项求和 | Yes | No |
| gstComponent | 总额内含GST；currency | grandTotal/11 | Yes | No |
| grandTotal | 总估算成本；currency | materialsTotal+labourTotal | Yes | No |
| report SVG/summary | 墙图与报告显示 | layout、WALL、成本DOM | Yes | No（仅浏览器打印） |

## 6. 缺失但需要业务确认的数据

| Question | Affected fields | Why it matters |
|---|---|---|
| 权威应用版本是v18还是v16？ | appVersion | 决定保存格式与计算规则的版本含义 |
| `$`代表何种币种、适用地区？ | 全部cost/rate/allowance/total | 金额无法稳定解释或比较 |
| GST `/11` 的税率、适用条件和有效期？ | gstComponent, grandTotal | 税务规则可能变化或不适用 |
| 默认价格、费率、密度和工程参数的来源/有效期？ | 第4节配置 | 决定默认值可信度和重算一致性 |
| “steel $/length”的length具体为何？ | u_cost_steel, steelBarCount | 单价单位含义不完整 |
| 空值是否必须与“采用默认值”永久等价？ | 全部u_* override | 空、零、缺失具有不同业务语义 |
| footing ratio、compaction、steel length的合法业务范围？ | u_footing_ratio,u_compact_factor,u_steel_len | 当前0或100可造成除零/无效结果 |
| 旧项目是否必须复现保存时的成本结果？ | appVersion、配置、全部派生输出 | 当前只存输入，新代码会重算出不同结果 |
| 颜色字段是否属于业务项目数据？ | C1-C6,CHR | 当前隐藏且保存，真实维护责任未知 |
| 项目名是否有长度、字符集或唯一性业务要求？ | projectName | 当前仅下载文件名被清洗，JSON名称限制不明 |
