# 03 Data Field Dictionary

## 1. Document Scope

- Target: `BrickWallDesigner_v18.html`; evidence comes from HTML controls, embedded JavaScript, Save/Load JSON, and calculation functions.
- Fields reflect current application behaviour. Recommended business names are readable candidates, not database column names.
- Current JSON stores numeric DOM values as strings; Data type describes business semantics.
- Browser round-trip validation was not completed; meanings not confirmable from code are marked Unknown.

## 2. Persistent Field Dictionary

- `Required` means the key is always emitted by Save JSON; “blank” means an empty string, not database NULL.
- This dictionary answers what the user enters when creating a project and what must be restored after closing it.

| Source path | Current key | Recommended business name | Meaning | Data type | Unit | Required | Nullable/default semantics | Allowed values/range | Example/default | Source evidence | Confidence | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| project.appName | appName | application_name | Application name | string | N/A | Yes | Fixed, nonblank | Brick Wall Designer | Brick Wall Designer | `saveProject` L2779-2785 | Confirmed | Not validated on Load |
| project.appVersion | appVersion | application_version | Saving application version | string | N/A | Yes | Fixed, nonblank | Currently hard-coded v16 | v16 | `saveProject` L2781 | Confirmed | Filename says v18; conflict |
| project.projectName | projectName | project_name | Project name | string | N/A | Yes | Must be nonblank after trim; zero N/A | Nonblank; filename separately limited to 120 chars | 2335×1750 Stretcher Bond | `sp_name`; `saveProject` | Confirmed | JSON name itself is not truncated |
| project.savedAt | savedAt | saved_at | Save timestamp | datetime | N/A | Yes | Browser-generated, nonblank | ISO-8601 string | 2026-09-02T02:00:00.000Z | `new Date().toISOString()` L2783 | Confirmed | Clock trust unknown |
| project.state | state | project_state | Project-input container | string | N/A | Yes | Fixed object; no fallback | values + radios | object | `collectProjectState` | Confirmed | Structural node, not scalar |
| state.values | values | input_values | Value-input container | string | N/A | Yes | Fixed object | 45 whitelisted keys | object | `PROJECT_VALUE_IDS` | Confirmed | Structural node |
| state.values.BT | BT | brick_type_code | Brick-type code | enum | N/A | Yes | Default s; blank/unknown breaks lookup | s,l,u,m,r | s | `BT`; `getBrick` | Confirmed | See Section 3 |
| state.values.MJ | MJ | mortar_joint_thickness | Mortar-joint thickness | decimal | mm | Yes | Blank, 0, or NaN falls back to 10 | HTML 0..30 | 10 | `MJ`; `getMortarThickness` L2200 | Confirmed | Allowed 0 is ineffective |
| state.values.WL | WL | entered_wall_length | Entered wall length | decimal | mm | Yes | Blank, 0, or NaN falls back to 1235 | HTML 100..50000 | 1235 | `WL`; `getWallLength` | Confirmed | Actual length may be adjusted |
| state.values.WH | WH | entered_wall_height | Entered wall height | decimal | mm | Yes | Blank, 0, or NaN falls back to 1750 | HTML 100..10000 | 1750 | `WH`; `getWallHeight` | Confirmed | — |
| state.values.C1 | C1 | odd_stretcher_colour | Odd-course stretcher colour | string | N/A | Yes | Fixed nonblank default; Load may replace | hex colour (unvalidated) | #C1440E | `C1`; `renderWall` | Confirmed | Hidden |
| state.values.C2 | C2 | even_stretcher_colour | Even-course stretcher colour | string | N/A | Yes | Same | hex colour (unvalidated) | #A63A0C | `C2`; `renderWall` | Confirmed | Hidden |
| state.values.C3 | C3 | special_brick_colour | Special-brick colour | string | N/A | Yes | Same | hex colour (unvalidated) | #8B6914 | `C3`; `renderWall` | Confirmed | Hidden |
| state.values.C4 | C4 | cut_brick_colour | Cut-brick colour | string | N/A | Yes | Same | hex colour (unvalidated) | #AE0000 | `C4`; `renderWall` | Confirmed | Hidden |
| state.values.C5 | C5 | mortar_colour | Mortar colour | string | N/A | Yes | Same | hex colour (unvalidated) | #D4C8B0 | `C5`; `renderWall` | Confirmed | Hidden |
| state.values.C6 | C6 | header_brick_colour | Header-brick colour | string | N/A | Yes | Same | hex colour (unvalidated) | #8B3A0C | `C6`; `renderWall` | Confirmed | Hidden |
| state.values.CHR | CHR | capping_brick_colour | Capping-brick colour | string | N/A | Yes | Same | hex colour (unvalidated) | #710000 | `CHR`; `renderWall` | Confirmed | Hidden |
| state.values.sk_hidden | sk_hidden | wall_skin_count | Wall skin count | integer | count | Yes | Saved value may change; calculation always returns 2 | Effectively 2 only | 2 | `sk_hidden`; `getSkinCount` | Confirmed | Non-2 saved value has no effect |
| state.values.u_cost_rb | u_cost_rb | road_base_unit_cost | Road-base unit cost | decimal | currency | Yes | Blank=code default 50; 0=explicit zero | HTML ≥0 | blank→50 | `userOrDefault` L5667 | Confirmed | Per m³ |
| state.values.u_cost_steel | u_cost_steel | steel_length_unit_cost | Reinforcing-steel length unit cost | decimal | currency | Yes | Blank=18; 0=explicit zero | ≥0 | blank→18 | L5668 | Confirmed | Meaning of “length” unknown |
| state.values.u_cost_conc | u_cost_conc | concrete_unit_cost | Concrete unit cost | decimal | currency | Yes | Blank=400; 0=explicit zero | ≥0 | blank→400 | L5669 | Confirmed | Per m³ |
| state.values.u_cost_brick | u_cost_brick | brick_unit_cost | Unit brick cost | decimal | currency | Yes | Blank=2; 0=explicit zero | ≥0 | blank→2.00 | L5670 | Confirmed | Per brick |
| state.values.u_cost_cement | u_cost_cement | cement_unit_cost | Cement unit cost | decimal | currency | Yes | Blank=0.5; 0=explicit zero | ≥0 | blank→0.50 | L5671 | Confirmed | Per kg |
| state.values.u_cost_lime | u_cost_lime | lime_unit_cost | Lime unit cost | decimal | currency | Yes | Blank=1.1; 0=explicit zero | ≥0 | blank→1.10 | L5672 | Confirmed | Per kg |
| state.values.u_cost_sand | u_cost_sand | sand_unit_cost | Sand unit cost | decimal | currency | Yes | Blank=0.1; 0=explicit zero | ≥0 | blank→0.10 | L5673 | Confirmed | Per kg |
| state.values.u_allow_site | u_allow_site | site_access_allowance | Site-access allowance | decimal | currency | Yes | Blank=0; 0=explicit zero | ≥0 | blank→0 | L5701 | Confirmed | Fixed amount |
| state.values.u_allow_waste | u_allow_waste | waste_disposal_allowance | Waste-disposal allowance | decimal | currency | Yes | Blank=0; 0=explicit zero | ≥0 | blank→0 | L5702 | Confirmed | Fixed amount |
| state.values.u_lab_prepare | u_lab_prepare | work_area_preparation_rate | Work-area preparation labour rate | decimal | currency | Yes | Blank=0; 0=explicit zero rate | ≥0 | blank→0 | L5703 | Confirmed | Per m² |
| state.values.u_lab_trench | u_lab_trench | trench_excavation_rate | Trench excavation labour rate | decimal | currency | Yes | Blank=50; 0=explicit zero rate | ≥0 | blank→50 | L5704 | Confirmed | Per m³ |
| state.values.u_lab_compact | u_lab_compact | road_base_compaction_rate | Road-base compaction labour rate | decimal | currency | Yes | Blank=20; 0=explicit zero rate | ≥0 | blank→20 | L5705 | Confirmed | Per m² |
| state.values.u_lab_pour | u_lab_pour | concrete_pour_rate | Concrete-pour labour rate | decimal | currency | Yes | Blank=50; 0=explicit zero rate | ≥0 | blank→50 | L5706 | Confirmed | Per m² in code |
| state.values.u_lab_lay | u_lab_lay | bricklaying_rate | Bricklaying labour rate | decimal | currency | Yes | Blank=2; 0=explicit zero rate | ≥0 | blank→2 | L5707 | Confirmed | Per brick |
| state.values.u_min_prepare | u_min_prepare | preparation_minimum_charge | Preparation minimum charge | decimal | currency | Yes | Blank=0; 0=explicit zero | ≥0 | blank→0 | L5709 | Confirmed | — |
| state.values.u_min_trench | u_min_trench | trench_minimum_charge | Trench minimum charge | decimal | currency | Yes | Blank=500; 0=explicit zero | ≥0 | blank→500 | L5710 | Confirmed | — |
| state.values.u_min_compact | u_min_compact | compaction_minimum_charge | Compaction minimum charge | decimal | currency | Yes | Blank=0; 0=explicit zero | ≥0 | blank→0 | L5711 | Confirmed | — |
| state.values.u_min_pour | u_min_pour | concrete_pour_minimum_charge | Concrete-pour minimum charge | decimal | currency | Yes | Blank=500; 0=explicit zero | ≥0 | blank→500 | L5712 | Confirmed | — |
| state.values.u_min_lay | u_min_lay | bricklaying_minimum_charge | Bricklaying minimum charge | decimal | currency | Yes | Blank=1000; 0=explicit zero | ≥0 | blank→1000 | L5713 | Confirmed | — |
| state.values.u_footing_ext | u_footing_ext | footing_extension | Footing extension | decimal | mm | Yes | Blank=100; 0=explicit zero | ≥0 | blank→100 | L5566 | Confirmed | — |
| state.values.u_footing_ratio | u_footing_ratio | wall_height_to_footing_depth_ratio | Wall-height to footing-depth ratio | decimal | N/A | Yes | Blank=4; 0 causes division by zero | ≥0 (effective value must be >0) | blank→4 | L5573-L5574 | Confirmed | Valid range needs confirmation |
| state.values.u_roadbase_depth | u_roadbase_depth | road_base_depth | Road-base depth | decimal | mm | Yes | Blank=100; 0=explicit zero | ≥0 | blank→100 | L5594 | Confirmed | — |
| state.values.u_compact_factor | u_compact_factor | compaction_factor | Compaction factor | decimal | percent | Yes | Blank=10; 0=explicit 0% | HTML 0..100; 100 divides by zero | blank→10 | L5602-L5603 | Confirmed | Effective maximum must be <100 |
| state.values.u_mesh_layers | u_mesh_layers | mesh_layer_count | Reinforcement mesh layer count | integer | count | Yes | Blank=2; 0=explicit no layers | ≥0 | blank→2 | L5591 | Inferred | parseFloat does not enforce integer |
| state.values.u_mesh_inset | u_mesh_inset | mesh_inset | Reinforcement mesh inset | decimal | mm | Yes | Blank=50; 0=explicit zero | ≥0 | blank→50 | L5583 | Confirmed | — |
| state.values.u_steel_len | u_steel_len | steel_bar_length | Steel-bar length | decimal | m | Yes | Blank=8; 0 causes division by zero | ≥0 (effective value must be >0) | blank→8 | L5611-L5613 | Confirmed | — |
| state.values.u_mix_cement | u_mix_cement | mortar_cement_part | Cement parts in mortar mix | decimal | count | Yes | Blank=1; 0=explicit no cement | ≥0 | blank→1 | L5643 | Confirmed | All three zero produces zero masses |
| state.values.u_mix_lime | u_mix_lime | mortar_lime_part | Lime parts in mortar mix | decimal | count | Yes | Blank=0; 0=explicit no lime | ≥0 | blank→0 | L5644 | Confirmed | — |
| state.values.u_mix_sand | u_mix_sand | mortar_sand_part | Sand parts in mortar mix | decimal | count | Yes | Blank=4; 0=explicit no sand | ≥0 | blank→4 | L5645 | Confirmed | — |
| state.values.u_mortar_contingency | u_mortar_contingency | mortar_contingency_percent | Mortar contingency | decimal | percent | Yes | Blank=10; 0=no contingency | ≥0 | blank→10 | L5635-L5636 | Confirmed | — |
| state.values.u_brick_contingency | u_brick_contingency | brick_contingency_percent | Brick contingency | decimal | percent | Yes | Blank=10; 0=no contingency | ≥0 | blank→10 | L5619-L5620 | Confirmed | — |
| state.values.u_end_clear | u_end_clear | work_area_end_clearance | Work-area end clearance | decimal | mm | Yes | Blank=2000; 0=explicit zero | ≥0 | blank→2000 | L5553 | Confirmed | Applied at both ends |
| state.values.u_side_clear | u_side_clear | work_area_side_clearance | Work-area side clearance | decimal | mm | Yes | Blank=1500; 0=explicit zero | ≥0 | blank→1500 | L5552 | Confirmed | Applied on both sides |
| state.radios.bond | bond | bond_type_code | Bond type | enum | N/A | Yes | Default stretcher; omitted if none selected | Seven values; Section 3 | stretcher | `bond`; `getBondType` | Confirmed | — |
| state.radios.cu | cu | cut_bricks_allowed | Whether cut bricks are allowed | boolean | N/A | Yes | y=true; n=false; default n | y,n | n | `cu`; `getCutsAllowed` | Confirmed | JSON stores enum string |
| state.radios.ad | ad | no_cut_length_adjustment | Length adjustment when cuts are disabled | enum | N/A | Yes | Default s; retained but not primary when cu=y | s,l | s | `ad`; `getLengthAdjust` | Confirmed | Conditional effect |
| state.radios.cc | cc | capping_included | Whether capping course is included | boolean | N/A | Yes | y=true; n=false; default n | y,n | n | `cc`; `getHasCapping` | Confirmed | — |
| state.radios.cco | cco | capping_orientation | Capping-brick orientation | enum | N/A | Yes | Default flat; saved but inactive when cc=n | flat,vertical,soldier | flat | `cco`; `getCappingOrientation` | Confirmed | Conditional effect |
| state.radios.ccoff | ccoff | capping_end_offset | Whether capping end bricks are offset | boolean | N/A | Yes | y=true; n=false; default n; inactive when cc=n | y,n | n | `ccoff`; `getCappingOffset` | Confirmed | — |

## 3. Reference Data Dictionary

- Standard options available for user selection.

| Data set | Code | Label | Dimensions/value | Unit | Source evidence | Confidence |
|---|---|---|---|---|---|---|
| Brick Types | s | Standard | 230×110×76 | mm | `BRICK_TYPES`,`BRICK_NAMES` L2150-L2163 | Confirmed |
| Brick Types | l | Slimline | 290×90×47 | mm | Same | Confirmed |
| Brick Types | u | Utility | 290×90×76 | mm | Same | Confirmed |
| Brick Types | m | Modular | 190×90×90 | mm | Same | Confirmed |
| Brick Types | r | Roman | 290×90×40 | mm | Same | Confirmed |
| Bond Types | stretcher | Stretcher Bond | code | N/A | `bond` radios; `NAMES` L5158-L5166 | Confirmed |
| Bond Types | header | Header Bond | code | N/A | Same | Confirmed |
| Bond Types | english | English Bond | code | N/A | Same | Confirmed |
| Bond Types | englishGardenWall | English Garden Wall Bond | code | N/A | Same | Confirmed |
| Bond Types | flemish | Flemish Bond | code | N/A | Same | Confirmed |
| Bond Types | flemishGardenWall | Flemish Garden Wall Bond | code | N/A | Same | Confirmed |
| Bond Types | common | Common Bond (American) | code | N/A | Same | Confirmed |
| Capping orientations | flat | Flat (header on flat) | unitW=brick width; rowH=brick height | mm | `cco`; `cappingDimensions` L2230 | Confirmed |
| Capping orientations | vertical | Vertical (header on edge) | unitW=brick height; rowH=brick width | mm | Same | Confirmed |
| Capping orientations | soldier | Soldier (stretcher on end) | unitW=brick height; rowH=brick length | mm | Same | Confirmed |
| Adjustment directions | s | Shorter | code | N/A | `ad` radios L1055-L1058 | Confirmed |
| Adjustment directions | l | Longer | code | N/A | Same | Confirmed |
| Yes/No flags | y | Yes/true | cu,cc,ccoff | N/A | Corresponding radios/getters | Confirmed |
| Yes/No flags | n | No/false | cu,cc,ccoff | N/A | Corresponding radios/getters | Confirmed |
| Brick render categories | o/e/h/s/c/cc/ccc | odd/even/header/special/cut/capping/capping-cut | render/count codes | N/A | `colorMap`,`counts` L4999,L5408 | Confirmed |

## 4. Calculation Configuration Field Dictionary

- For every “user override,” a blank UI value uses the code default and input 0 is an explicit override; all override fields appear individually in Section 2.
- These fields define the parameters, prices, and rules used to calculate results.

| Field | Current form | Default/rule | Unit | Source evidence | Confidence |
|---|---|---:|---|---|---|
| entered_wall_length/height | User input + HTML default | 1235 / 1750 | mm | `WL`,`WH`; getters | Confirmed |
| mortar_joint_thickness | User input + HTML default | 10 | mm | `MJ`; getter | Confirmed |
| wall_skin_count | Hard-coded | 2 | count | `getSkinCount`; WALL width | Confirmed |
| side/end clearance | User override | 1500 / 2000 | mm | L5552-L5553 | Confirmed |
| footing extension/ratio | User override | 100 / 4 | mm / N/A | L5566,L5573 | Confirmed |
| mesh inset/layers | User override | 50 / 2 | mm / count | L5583,L5591 | Confirmed |
| road-base depth/compaction | User override | 100 / 10 | mm / percent | L5594,L5602 | Confirmed |
| steel-bar length | User override | 8 | m | L5611 | Confirmed |
| brick/mortar contingency | User override | 10 / 10 | percent | L5619,L5635 | Confirmed |
| mortar cement/lime/sand parts | User override | 1 / 0 / 4 | count | L5643-L5646 | Confirmed |
| cement/lime/sand density | Hard-coded | 1440 / 500 / 1600 | kg/m³ | L5648-L5650 | Confirmed |
| road-base/steel/concrete/brick costs | User override | 50 / 18 / 400 / 2 | currency | L5667-L5670 | Confirmed |
| cement/lime/sand costs | User override | 0.5 / 1.1 / 0.1 | currency | L5671-L5673 | Confirmed |
| site/waste allowances | User override | 0 / 0 | currency | L5701-L5702 | Confirmed |
| prepare/trench/compact/pour/lay rates | User override | 0 / 50 / 20 / 50 / 2 | currency | L5703-L5707 | Confirmed |
| prepare/trench/compact/pour/lay minima | User override | 0 / 500 / 0 / 500 / 1000 | currency | L5709-L5713 | Confirmed |
| mortar face assumption | Hard-coded rule | bottom + 50% long side + one end | N/A | L5623-L5634 | Confirmed |
| labour minimum rule | Hard-coded rule | max(calculated charge, minimum) | currency | L5715-L5731 | Confirmed |
| GST included component | Hard-coded rule | grand total / 11 | currency | L5750-L5754 | Confirmed |
| geometry tolerance | Hard-coded rule | 0.5 | mm | `validateLayoutGeometry` L5188-L5207 | Confirmed |
| next-valid-length search | Hard-coded rule | 5mm step; +3000mm cap | mm | `findNextValidLength` L5227-L5237 | Confirmed |

## 5. Derived Output Dictionary

- Answers: what does the application calculate from the inputs?

| Derived output | Meaning/unit | Calculation source | Recalculable | Currently saved |
|---|---|---|---|---|
| WALL.length / height / width | Actual wall length, height, double-skin width; mm | layout + brick width + mortar | Yes | No |
| actual course count / capping height | Wall courses/capping height; count/mm | wall/brick height, mortar, orientation | Yes | No |
| stretchers/headers/specials/cuts/cappingBricks | Counts by brick category; count | layout categories and multipliers | Yes | No |
| WALL.total / bricksToOrder | Design total / contingency-adjusted order; count | sum; `ceil(total×(1+contingency))` | Yes | No |
| workAreaM2 | Work area; m² | wall dimensions + end/side clearance | Yes | No |
| wallHorizontal/VerticalFootprint | Horizontal/vertical wall footprint; m² | WALL dimensions | Yes | No |
| footingFootprint/depth/volume | Footing area, depth, volume; m²/mm/m³ | wall, extension, ratio | Yes | No |
| meshWidth/Length/Layers | Mesh dimensions/layers; mm/count | footing, inset, layers | Yes | No |
| excavationVolume | Excavation volume; m³ | footing area × (footing + road-base depth) | Yes | No |
| roadBaseVolume/originalVolume | Compacted/original road-base volume; m³ | depth and compaction factor | Yes | No |
| steelBarCount | Steel-bar quantity; count | `ceil(mesh total length/bar length)` | Yes | No |
| mortarVolume/totalVolume | Net/contingency mortar volume; m³ | brick count/dimensions, joint, contingency | Yes | No |
| cement/lime/sand mass | Mortar component masses; kg | mortar × mix ratio × density | Yes | No |
| materialRoadBase/Steel/Concrete/Bricks/Cement/Lime/Sand | Seven material costs; currency | quantity × unit cost | Yes | No |
| materialsTotal | Total materials cost; currency | sum of seven material costs | Yes | No |
| labourSite/Waste | Fixed site/waste allowances; currency | user value or default | Yes | No |
| labourPrepare/Trench/Compact/Pour/Lay | Five labour costs; currency | max(quantity×rate, minimum) | Yes | No |
| labourTotal | Labour and allowances total; currency | sum of seven items | Yes | No |
| gstComponent | GST included in total; currency | grandTotal/11 | Yes | No |
| grandTotal | Total estimated cost; currency | materialsTotal+labourTotal | Yes | No |
| report SVG/summary | Wall graphic and report display | layout, WALL, cost DOM | Yes | No (browser print only) |

## 6. Missing Data Requiring Business Confirmation

| Question | Affected fields | Why it matters |
|---|---|---|
| Is the authoritative application version v18 or v16? | appVersion | Determines save-format and calculation-rule version meaning |
| Which currency and region does `$` represent? | all cost/rate/allowance/total fields | Monetary values cannot be interpreted or compared reliably |
| What rate, conditions, and validity period govern GST `/11`? | gstComponent, grandTotal | Tax rule may change or not apply |
| What are the source/effective dates of default prices, rates, densities, and engineering parameters? | Section 4 configuration | Determines default credibility and reproducibility |
| What exactly is one steel “length” for `$ / length`? | u_cost_steel, steelBarCount | Unit-price meaning is incomplete |
| Must blank permanently mean “use the current default”? | all `u_*` overrides | Blank, zero, and omitted have different semantics |
| What are valid business ranges for footing ratio, compaction, and steel length? | u_footing_ratio,u_compact_factor,u_steel_len | Current 0/100 inputs can cause division by zero or invalid output |
| Must old projects reproduce their original estimated results? | appVersion, configuration, all derived outputs | Only inputs are saved; new code may calculate different values |
| Are colour fields business project data? | C1-C6,CHR | They are hidden but saved; ownership is unclear |
| Does project name have length, character-set, or uniqueness rules? | projectName | Only the downloaded filename is sanitized; JSON name rules are unknown |
