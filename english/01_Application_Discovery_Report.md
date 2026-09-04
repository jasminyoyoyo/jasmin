# 01 Application Discovery Report

> Analysis target: `BrickWallDesigner_v18.html`  
> Evidence status: **Confirmed** = verified in source code; **Inferred** = inferred from code behaviour; **Unknown** = cannot be confirmed from available material.  
> This report covers application reverse engineering only. It does not define a database, API, CMS, or implementation design.

# 1. Application Information and Analysis Scope

| Item | Finding | Status and evidence |
|---|---|---|
| Application name | Brick Wall Designer | **Confirmed**: HTML title, start screen, and saved `appName`. |
| HTML filename | `BrickWallDesigner_v18.html` | **Confirmed**: source file inspected. |
| Application version | Application reports **v16**; filename says **v18**; provenance comment refers to **v15** | **Confirmed**: L2, L6, L977, L2781. The authoritative release version cannot be assumed. |
| Analysis date | 2026-09-02 (Australia/Sydney) | **Confirmed**: analysis environment date. |
| File information | 217,237 bytes; PowerShell count: 5,649 lines, 17,655 words, 211,446 characters; modified 2026-09-02T12:17:31.2824377+10:00 | **Confirmed**. Line-ending-aware tools may show locations through L5792. |
| SHA-256 | `D311909E45DE1C84E2F851B3B77425704A08EE712584508BD70AF695EDA9A550` | **Confirmed**. |
| File completeness | DOCTYPE, complete `html/head/body`, closed script/body/html, and readable embedded CSS/JS | **Confirmed** structurally; **Unknown** whether the business release package omitted anything. |
| Single-file application | Yes; CSS, JavaScript, and SVG generation are embedded; no external `src`/`href` dependency found | **Confirmed**. |
| Known missing files | No missing file referenced by code | **Confirmed**. The Outlook-cache comment is provenance, not a runtime dependency. |
| Known missing information | Product owner, target browsers, version governance, official price sources, user/project permissions, retention policy, and actual volumes | **Unknown**. |

Included: static HTML/CSS/JavaScript analysis; pages, controls, events, state, calculations, JSON Save/Load, sources, persistence paths, attempted runtime validation, and application evidence for conceptual data discovery.

Excluded: final database tables/fields, key implementation, PostgreSQL types, indexes, SQL, migrations, CMS schema, APIs, construction/cost correctness certification, cross-browser certification, and production security testing.

Limitation: no browser instance was available, so UI interaction, download/upload round trips, Storage/Network inspection, and print-window validation could not be performed. Dynamic conclusions therefore remain code-path findings rather than runtime-confirmed results.

# 2. Analysis Methods

| Method | Completion | Description |
|---|---|---|
| Static Code Analysis | **Completed / Confirmed** | Read the complete single-file structure and extracted controls, functions, constants, state, saved fields, calculations, and rendering paths. |
| Dynamic Runtime Analysis | **Attempted but unavailable / Unknown** | Browser connection was attempted, but no browser was available. |
| Scenario-Based Analysis | **Completed statically / Inferred** | Traced New, Draw/Edit, Save, Load, Report, and Reset through event handlers. |
| Data Flow Analysis | **Completed / Confirmed** | Traced UI → getters/variables → layout/WALL → costs/report/save. |
| Persistence Path Analysis | **Completed / Confirmed** | Traced `collectProjectState`, `JSON.stringify`, Blob download, FileReader, `JSON.parse`, and `applyProjectState`. |
| Traceability Analysis | **Completed / Confirmed** | Significant findings map to HTML IDs, functions, or source locations; runtime findings remain Unknown. |

# 3. Primary Purpose and User Tasks

The application generates a schematic double-skin brick wall, brick counts, material quantities, cost estimates, and a printable report from wall dimensions, brick type, mortar, bond, cut-brick, and capping selections. It saves and restores inputs as JSON. **Confirmed**.

Likely users are owners, estimators, or design-support staff planning a brick wall. This is **Inferred** because the code has no identity, role, or qualification model. The application explicitly states that drawings are schematic and not for construction, and estimates are for planning only. **Confirmed** (L1197 and print disclaimer).

Main functional areas:

1. Start: New Project; Existing Project is marked Coming soon.
2. Designer: wall parameters, brick type, bond, cuts, capping, SVG wall, summary, brick count, zoom, and geometry diagnostics.
3. Materials & Costs: override inputs, derived quantities, material/labour costs, and GST-inclusive total.
4. Reports: current wall drawing, design/cost summaries, and browser print/export.
5. Save Project: project naming, JSON download, and JSON file loading.

Outputs are the schematic SVG, actual dimensions, brick counts, material/cost estimate, printable HTML report, and a JSON project file containing restorable inputs. **Confirmed**.

# 4. Application Workflows

| Workflow | Trigger/UI | JavaScript | Data effect | Save/load |
|---|---|---|---|---|
| New Project | Start-screen `NEW PROJECT` | inline handler → `showPage(1)` | Hides overlay and shows Designer; does not clear state | No |
| Create Design / Draw | Designer `Draw Wall` | `drawWall` → `calculateLayout` → `renderWall` → `recalculateCosts` | Reads inputs and rebuilds `lastLayout`, SVG, counts, `WALL`, and costs | No auto-save |
| Edit Design | Change input and Draw again | Same | Full recalculation, not local entity editing | Inputs may later be saved |
| Add/Remove/Copy Item | No independent UI | None | Brick graphics are generated, not user-managed persistent items | N/A |
| Delete | No project/item delete function | None | N/A | N/A |
| Materials override | Page 2 `u_*` inputs | `oninput=recalculateCosts()`, `userOrDefault` | Blank uses built-in default; nonblank zero overrides; updates derived quantity/cost DOM | 34 related values are saved |
| View Report | nav3 | `showPage(3)` → `populateReport` | Clones current SVG and reads `WALL` and cost DOM | No |
| Print/Export Report | Two `printReport` buttons | `printReport` | Builds temporary HTML; uses `window.open` + print, falling back to `window.print` | Browser may save PDF; no application PDF object |
| Suggest project name | `Suggest a name` | `suggestProjectName` | Builds `sp_name` from actual size and bond | Saved as root projectName |
| Save | `Save Project to File` | `saveProject` → `collectProjectState` → `JSON.stringify` → Blob/anchor | Creates appName, appVersion, projectName, savedAt, state | Downloads `.json`; no local database |
| Load / Import | `sp_load_file` + `Load Project` | `loadProject` → FileReader → `JSON.parse` → `applyProjectState` | Overwrites listed inputs/radios, triggers change, redraws/recalculates | Restores a selected JSON file |
| Reset | Start navigation | `showStartScreen` | Only displays overlay; does not reset values | No |
| Diagnostics | `Run Wall Integrity Test` | `runWallIntegrityTest`, `checkLayoutIntegrity` | Snapshots some inputs, sweeps lengths/bonds/bricks, restores them; failures stay in DOM memory | No |
| Zoom/Fit | toolbar | `zoom`, `fitToWindow` | Changes `currentScale` and redraws SVG | No |

# 5. External Dependencies and Data Sources

| Source | Location | Purpose | Accessible | Completeness impact |
|---|---|---|---|---|
| Embedded HTML/CSS/JS | Whole file; script L2146-L5789 | UI, algorithms, state, save/load | Yes | None missing |
| External JavaScript/CSS | Not found | None | N/A | None |
| Imported JSON project | `sp_load_file` L2117-L2127; `loadProject` L2802-L2835 | Restore inputs | Structure confirmed; no sample file | Runtime round trip unverified |
| Other JSON/CSV/config | Not found | None | N/A | None |
| API/fetch/XHR | Not found | None | N/A | None |
| Images/product assets | No URLs/files; wall is generated SVG | Visual output | Yes, as code | None |
| LocalStorage / SessionStorage / IndexedDB | Not found | None | N/A | None |
| URL parameters | No `URLSearchParams`/`location.search` | None | N/A | None |
| Browser time | `new Date().toISOString()` L2783; locale formatting on load | Saved time/display | Runtime supplied | Clock/time-zone trust unverified |
| Blob/Object URL/download | L2787-L2797 | Browser JSON download | Code accessible | Actual download unverified |
| Print window | L2552-L2587 | HTML report/browser printing | Code accessible | Popup/print policy affects runtime |
| Outlook cache provenance | v15 file URL in L2 comment | Provenance only; not read at runtime | Original location unconfirmed | Version traceability question |

# 6. Page Inputs and UI Control Inventory

All items below are **Confirmed**. Save: `V` = `state.values`; `R` = `state.radios`; `N` = not saved; Root = root saved object.

## 6.1 Designer and Diagnostic Controls

| Control/ID or name | Type; data; example/values | JS/Event | Save | Evidence |
|---|---|---|---|---|
| `WL` Length | number; mm; 1235; 100..50000 | `getWallLength`; read on Draw | V | L1024, L2205, L2658 |
| `WH` Height | number; mm; 1750; 100..10000 | `getWallHeight` | V | L1027, L2206, L2659 |
| `MJ` Mortar | number; mm; 10; 0..30 | `getMortarThickness` (0 falls back to 10) | V | L1030, L2200, L2657 |
| `BT` Brick Type | select; code; s/l/u/m/r | `getBrick`/`getBrickLabel` | V | L1035-L1041, L2199 |
| `cu` Allow cuts | radio; y/n; default n | `getCutsAllowed`; change alters `AR` opacity | R | L1048-L1050, L2201-L2227 |
| `ad` Adjust direction | radio; s/l; default s | `getLengthAdjust` | R | L1055-L1058 |
| `bond` | radio; 7 values; default stretcher | `getBondType`, layout/minimum length | R | L1063-L1076 |
| `cc` Include capping | radio; y/n; default n | `getHasCapping`; change shows options | R | L1083-L1085, L2210-L2223 |
| `cco` Orientation | radio; flat/vertical/soldier | `getCappingOrientation` | R | L1091-L1096 |
| `ccoff` End offset | radio; y/n; default n | `getCappingOffset` | R | L1102-L1104 |
| `sk_hidden` | hidden; integer-like string; 2 | `getSkinCount` always returns 2; field not read | V | L1113, L2207, L2667 |
| `C1,C2,C6,C3,C4,C5,CHR` | hidden; hex colours such as `#C1440E` | `renderWall` colour map | V | L1115-L1121, L4997-L5008 |
| `DIAG_MIN/MAX/STEP/MORTAR` | number; 600/10000/1/17 | `runWallIntegrityTest` | N | L928-L940, L4830-L4836 |
| `DIAG_ALLBONDS/ALLBRICKS/CUTS_ONLY` | checkbox; boolean; cuts default true | `runWallIntegrityTest` | N | L945-L959, L4834-L4836 |
| Draw button | button | `drawWall()` | N | L1123 |
| Diagnostic modal controls | buttons | `openDiagModal`,`closeDiagModal`,`runWallIntegrityTest`,`loadFailingWall` | N | L911, L961, L1124, L4951 |
| Zoom +/−/Fit | buttons | `zoom(1.25)`,`zoom(0.8)`,`fitToWindow` | N | L1193-L1196 |

## 6.2 Materials & Costs Inputs

All are number inputs with `oninput=recalculateCosts()`. Blank uses the code default; all enter Save as V.

| ID | Business meaning; unit | Code default | Evidence |
|---|---|---:|---|
| `u_side_clear` | side clearance; mm | 1500 | L5552 |
| `u_end_clear` | end clearance; mm | 2000 | L5553 |
| `u_footing_ext` | footing extension; mm | 100 | L5566 |
| `u_footing_ratio` | wall height : footing depth ratio | 4 | L5573 |
| `u_mesh_inset` | mesh inset; mm | 50 | L5583 |
| `u_mesh_layers` | mesh layers; count | 2 | L5591 |
| `u_roadbase_depth` | road-base depth; mm | 100 | L5594 |
| `u_compact_factor` | compaction; percent; HTML 0..100 | 10 | L5602 |
| `u_steel_len` | reinforcing bar length; m | 8 | L5611 |
| `u_brick_contingency` | brick contingency; percent | 10 | L5619 |
| `u_mortar_contingency` | mortar contingency; percent | 10 | L5635 |
| `u_mix_cement` / `u_mix_lime` / `u_mix_sand` | mortar mix parts | 1 / 0 / 4 | L5643-L5645 |
| `u_cost_rb` / `u_cost_steel` / `u_cost_conc` / `u_cost_brick` | road base, steel, concrete, brick unit costs | 50 / 18 / 400 / 2.00 | L5667-L5670 |
| `u_cost_cement` / `u_cost_lime` / `u_cost_sand` | material cost per kg | 0.50 / 1.10 / 0.10 | L5671-L5673 |
| `u_allow_site` / `u_allow_waste` | site/waste allowances; $ | 0 / 0 | L5701-L5702 |
| `u_lab_prepare` / `u_lab_trench` / `u_lab_compact` / `u_lab_pour` / `u_lab_lay` | labour rates | 0 / 50 / 20 / 50 / 2 | L5703-L5707 |
| `u_min_prepare` / `u_min_trench` / `u_min_compact` / `u_min_pour` / `u_min_lay` | minimum charges; $ | 0 / 500 / 0 / 500 / 1000 | L5709-L5713 |

HTML `min` is 0 for these controls; `u_compact_factor` also has max=100, and several currency items have step=0.01. **Confirmed**. `cv_*` and `rp_*` values are derived display outputs and are not saved.

## 6.3 Save/Load and Navigation Controls

| Control | Type/example | JS/Event | Save | Evidence |
|---|---|---|---|---|
| `sp_name` | text; “Foley Residence …” | `suggestProjectName`,`saveProject`; assigned after Load | Root `projectName` | L2052-L2065, L2771-L2785, L2814 |
| `sp_load_file` | upload; accepts JSON | `loadProject`/FileReader | N | L2117-L2129 |
| Save / Load buttons | buttons | `saveProject()` / `loadProject()` | Execute persistence | L2085-L2087, L2128-L2130 |
| `nav0..nav4` | buttons | `showStartScreen`,`showPage(1..4)` | N | L999-L1013 |
| Print buttons (2) | buttons | `printReport()` | N | Static inventory; L2484 onward |

No slider, dedicated toggle, textarea, dynamically generated input, or item add/remove/copy control was found. Conditional areas are `AR` (opacity only) and `CC_OPTIONS` (display).

# 7. JavaScript Data Object Inventory

| Name | Type/example | Creation, reads, changes, functions | Save/Load | Evidence |
|---|---|---|---|---|
| `BRICK_TYPES` | map; `s:{l:230,w:110,h:76}`; 5 entries | Created L2150; read by `getBrick`, layout, diagnostics, minLength; not changed | Only selected code saved | L2150-L2156 |
| `BRICK_NAMES` | map; s→Standard; 5 entries | Labels/diagnostics read | No | L2157-L2163 |
| `NAMES` | 7 bond code→label entries | Read by report, name suggestion, warnings | Only bond code saved | L5158-L5166 |
| `ALL_BOND_VALUES` | array; 7 codes | Iterated by diagnostics | No | L4752-L4760 |
| `PROJECT_VALUE_IDS` | array; 45 IDs | Symmetrically collected/applied | Defines values scope | L2655-L2701 |
| `PROJECT_RADIO_GROUPS` | array; 6 names | Symmetrically collected/applied | Defines radios scope | L2702 |
| `WALL` | mutable state; dimensions/counts/ready | Created L2183; updated L5473-L5500; read by costs/report | Rebuilt after Load, not directly saved | L2183-L2195 |
| `lastLayout` / `currentScale` | object/null; number 1 | Drawing/zoom/fit/render | No; derived/UI state | L2178-L2179, L5764-L5773 |
| `layout` and brick geometry records | calculated object and `{x1,x2,y1,y2,t,...}` items | Created by layout/row/capping builders; read by render/count/integrity | No; recalculable | L2948 onward; L4740-L4742 |
| `counts` / `countRows` | category count object / summary rows | Built from layout during Draw | No; derived | L5408-L5440 |
| `state` | `{values:{},radios:{}}` | `collectProjectState`; read by `applyProjectState` | Core project state | L2704-L2744 |
| `project` | `{appName,appVersion,projectName,savedAt,state}` | Created by Save; partially read by Load | JSON root | L2779-L2785 |
| Diagnostic `snap` / `failures` | input snapshot / failure array | Test creation, restoration, result rendering | No | L4778-L4825, L4846-L4914 |
| `geomMinCache` | key `bond|brick|mortar` | Filled/read by `getGeomMinLength` | No | L5240-L5255 |
| `colorMap` | brick category→hidden colour | Recreated by `renderWall` | Source colours saved | L4997-L5008 |
| Cost locals | numeric quantities and totals | Created by `recalculateCosts`, written to DOM | No; recalculable | L5534-L5755 |
| Defaults/presets | dimensions, colours, costs, construction values, densities, GST rule | HTML/code constants | Reference/config; not independently versioned | L2150-L2163, L5514-L5754 |
| Sample/test data | diagnostics 600..10000/17; sample project name | UI examples/test parameters | No | L928-L959, L2055 |

All internal `lastLayout` fields are derived geometry produced by multi-branch algorithms and should not be directly treated as database fields at this stage.

# 8. Persistence Behaviour Overview

- Save requires a nonblank name, collects 45 value IDs and 6 radio groups, creates project metadata, serialises JSON, and downloads an `application/json` Blob through a temporary Object URL. **Confirmed**.
- Load reads a selected `.json` with FileReader, parses it, applies only `project.state`, restores name/time display, then redraws and recalculates. **Confirmed**.
- Values and radios use the same lists in collect/apply and are field-level symmetric. Root `appName`/`appVersion` are not validated or used; `savedAt` is display-only; unknown fields are ignored; missing fields retain current UI values. **Confirmed**.
- No auto-save, LocalStorage, SessionStorage, IndexedDB, or API request/response was found. **Confirmed**.
- No schemaVersion, migration, or version branch exists. Missing-field tolerance provides weak compatibility but is not an explicit version policy. **Confirmed/Inferred**.
- Not persisted: `WALL`, layout/SVG, derived costs, diagnostic results, zoom, current page, collapse state, diagnostic controls, file input, and status text. Most are recalculable or temporary UI state. **Confirmed**.
- Save claims every input, but `sp_name` is at the root and diagnostics are excluded. `sk_hidden` is saved although the calculation getter always returns 2. **Confirmed**.

# 9. Runtime Validation

A browser connection was attempted, but no browser instance was available.

| Scenario | Planned steps | Actual result | Static expectation |
|---|---|---|---|
| Create blank design | New Project → Designer | **Unknown: not run** | Opens current defaults; does not clear state |
| Change primary inputs | Change WL/WH/MJ/BT/bond | **Unknown: not run** | Draw fully recalculates |
| Add/edit/delete design element | Search for item controls | **Not applicable** | Individual bricks are not editable objects |
| Save/export | Name, download JSON, print report | **Unknown: not run** | JSON Blob and print HTML |
| Reload/import | Select saved JSON | **Unknown: not run** | Restore state, redraw, recalculate |
| Compare before/after | Compare JSON/UI/WALL | **Unknown: not run** | Listed fields symmetric; outputs recalculated |
| Storage/Network | Inspect Local/Session/IDB/network | **Unknown: not run** | Static code predicts no persistence/request |

**Current conclusions are based only on static code analysis; runtime validation has not been completed.**

# 10. Application Report Conclusion

Confirmed: four-page flow, double-skin design parameters, five brick types, seven bonds, cuts and capping, SVG/count/material/cost derivation, diagnostic sweep, print report, and JSON save/load.

Confirmed sources: HTML defaults, JavaScript reference/configuration constants, user UI input, imported local JSON, and browser time. No API, external configuration, LocalStorage, SessionStorage, or IndexedDB exists.

No referenced dependency is missing, but downloads, import, print, live DOM behaviour, and Storage/Network inspection remain unverified because no browser was available. No functional module or persistent candidate remains unmapped; low-level geometry/calculation variables were grouped as derived data rather than promoted to business attributes.

**Analysis completeness: Static analysis complete, runtime verification missing**.
