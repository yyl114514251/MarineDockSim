# MarineDockSim 构建规格（构建代理必读）

> 本文件是构建单文件 HTML 的硬约束。数据来自同目录 `01-海事仿真市场与政策调研报告.md` 与 `02-船型参数靠泊规范与事故案例技术报告.md`，构建前必须 Read 这两份报告，把其中的船型/港口/事故/阈值数据以 JS 常量形式内嵌。

## 1. 交付物
- 单个文件：`MarineDockSim.html`，保存到 `C:\Users\刘亦扬\Doubao\chats\2026-09-12\new-chat\MarineDockSim\MarineDockSim.html`
- 双击即用，CSS/JS 全部内联，**无任何外部运行时依赖**（不引 ECharts/Leaflet/jQuery；图表与地图全部手写 Canvas/SVG）。字体用系统字体栈，不引外部字体。
- 不使用 `type="module"`、不使用 `eval`/`new Function`、不使用 `innerHTML` 插入用户输入（一律 `textContent`）。
- `<head>` 内含 CSP meta：`<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self' https://*;">`（允许用户配置的 https API 端点）。

## 2. 视觉风格（深色科技风）
- 背景：`#0a0e1a` 主背景，`#111827` 卡片，`#1e293b` 悬浮层。
- 主强调色：青蓝 `#22d3ee`，次强调：琥珀 `#f59e0b`（警告）、红 `#ef4444`（危险）、绿 `#10b981`（安全）。
- 字体：`'Segoe UI', system-ui, -apple-system, sans-serif`；数据/仪表盘用 `'Consolas','Monaco',monospace`。
- 圆角 8px，细边框 `1px solid rgba(34,211,238,0.15)`，卡片 hover 有微光。
- 不用 emoji，图标用内联 SVG（24x24 viewBox）。
- 有 favicon（内联 SVG data URI，船锚/舵轮图案）。

## 3. 页面结构（hash 路由，单页切换）
顶部导航栏（固定）：Logo「MarineDockSim」+ 6 个入口：总览看板 / 靠泊仿真 / 船型库 / 港口场景 / 事故案例 / 报告导出 / 设置。
所有视图在一个 `<main id="view">` 内切换，每个视图一个 `<section data-view="xxx">`，JS 控制显示。

### 3.1 总览看板 (#dashboard)
- 市场概览卡片 4 张：全球海事仿真市场规模(2024)、CAGR、欧洲/亚太占比、全任务模拟器单套造价区间——数据来自报告1，卡片右下角标注来源。
- 后舷靠泊流程 5 步时间轴：进港 → 减速 → 调整艏向 → 尾端对准 → 贴靠护舷。
- 系统能力矩阵小表：MMG 三自由度 / 风流载荷 / 拖轮推力 / 风险雷达 / 报告导出。
- 快捷入口：「开始仿真」按钮跳 #simulation。

### 3.2 靠泊仿真 (#simulation) —— 核心
布局：左侧控制面板（320px）+ 中央 Canvas 主视图 + 右侧仪表盘（300px）。窄屏堆叠。

**左侧控制面板：**
- 船型选择下拉（来自船型库，3+ 型）。
- 港口场景选择下拉（来自港口库，4+ 个）。
- 环境参数：风速(m/s)、风向(度)、流速(kn)、流向(度)——数字输入，带范围校验（风速 0-30，流速 0-5）。
- 初始状态：初始距离泊位(m)、初始横向偏距(m)、初始艏向角(度)、初始航速(kn)。
- 控制输入（实时可调）：主机转速 RPM 滑块（0-100% MCR）、舵角滑块（-35°~+35°）、拖轮推力滑块（0-100kN，左右各一或合成横向推力）。
- 按钮：开始 / 暂停 / 重置 / 加速（1x/2x/5x）/ 自动靠泊（规则引擎自动给控制量）。
- 时间步长显示、仿真耗时显示。

**中央 Canvas（800x600 自适应）：**
- 俯视 2D 视图：泊位（码头线+护舷方块）、防波堤、航道边界、船舶轮廓（按真实长宽比缩放，船首三角标记）、风流矢量箭头（青色风、蓝色流）、拖轮位置与推力箭头、船舶尾迹线（淡色）。
- 比例尺与指北针。
- 泊位附近显示"安全区"半透明框与"危险接近"提示。
- 可鼠标拖拽平移、滚轮缩放（简单实现）。

**右侧仪表盘：**
- 实时数值（等宽字体）：航速(kn)、横移速度(m/s)、艏向角(°)、艏摇角速度(°/s)、距泊位法向距离(m)、法向接近速度(m/s)、主机推力(kN)、舵力(kN)、风载荷(kN)、流载荷(kN)。
- 趋势曲线 Canvas（手写）：最近 60 秒的航速 / 横移速度 / 艏向角 三条折线，不同颜色。
- 靠泊风险雷达图（手写 Canvas 六边形雷达）：6 维——法向速度、横移速度、艏摇角速度、风载荷、流载荷、护舷能量，每维 0-1 归一化，绿色安全区/红色危险区叠加。
- 风险等级指示灯：安全(绿)/注意(黄)/危险(红)，基于阈值实时判定。

### 3.3 船型库 (#ships)
- 卡片网格，每型船一张卡片：船名、船型标签、主尺度表（LOA/B/d/Δ/功率/航速）、适用场景说明、数据来源链接。
- 点击卡片可"在仿真中使用"跳转到 #simulation 并预选该船。

### 3.4 港口场景 (#ports)
- 卡片网格，每个港口一张：港口名、国家、泊位走向/长度/水深、护舷类型、设计靠泊速度、潮流特征、常风向、来源。
- 简易泊位示意图（SVG：码头线、护舷、水深标注、泊位走向角度）。
- 点击可"在仿真中使用"。

### 3.5 事故案例 (#accidents)
- 列表/卡片，每起：日期、船名、港口、事故简述、直接原因、教训、调查报告链接（可点击新窗口打开）。
- 顶部可按船型/事故类型筛选。
- 每起案例关联"风险提示"标签（如"高速接近""风流叠加""舵效不足"）。

### 3.6 报告导出 (#report)
- 显示当前/最近一次仿真的复盘：工况摘要表、关键极值（最大法向速度、最大横移、最大护舷能量）、风险结论、操作建议。
- 「AI 生成作业方案」按钮：无 key 时用本地规则引擎生成（基于船型/港口/环境输出结构化方案文本：进港路线、减速点、舵角序列、拖轮配置、应急预案）；有 key 时调用用户配置的大模型 API 生成。
- 「导出 Markdown」按钮：把工况+时序数据摘要+风险结论+方案生成为 `.md` 文本，用 Blob 下载（文件名 `berthing-report-YYYYMMDD-HHMM.md`）。
- 「导出 CSV 时序」按钮：导出仿真全程时序数据（时间、u、v、r、x、y、psi、控制量）为 CSV 下载。

### 3.7 设置 (#settings)
- AI 配置：API Base URL（默认空，占位符 `https://api.example.com/v1`）、API Key（password 输入，仅存 localStorage）、模型名（默认 `gpt-4o-mini` 或用户自填）。
- 「测试连接」按钮：发一个最小请求，显示成功/失败。
- 「清除本地数据」按钮：清 localStorage。
- 说明文字：key 仅存本地浏览器，不上传任何服务器；无 key 时系统自动降级为规则引擎。

## 4. 仿真引擎规格（MMG 简化模型）

### 4.1 状态向量
`state = { u, v, r, x, y, psi }` —— 纵荡速度 u(m/s)、横荡速度 v(m/s)、艏摇角速度 r(rad/s)、大地坐标 x/y(m)、艏向角 psi(rad)。

### 4.2 运动方程（随船坐标系）
```
(m + mx) * du/dt - (m + my) * v * r = X_H + X_P + X_R + X_W + X_C
(m + my) * dv/dt + (m + mx) * u * r = Y_H + Y_R + Y_W + Y_C
(Iz + Jz) * dr/dt = N_H + N_R + N_W + N_C
```
- m = 排水量(kg)，mx/my = 附加质量（用无量纲系数 mx'=mx/(0.5ρL³) 等估算，取报告给的典型范围中值）。
- Iz = (1/12)m(Lpp²+B²) 近似，Jz 用无量纲系数估算。

### 4.3 各力模型（简化但物理自洽）
- **裸船体水动力 X_H/Y_H/N_H**：线性+非线性阻尼。`X_H = -X_u u|u| - X_vv v²`（简化）；`Y_H = Y_v v + Y_r r + Y_vvv v³`；`N_H = N_v v + N_r r + N_vvv v³`。系数用无量纲典型值（Y_v'≈-0.3, N_r'≈-0.01 等，从 MMG 文献取），按 `0.5ρL²d` 等量化。
- **螺旋桨推力 X_P**：`T = ρ n² D⁴ K_T(J)`，n=RPS（由 RPM%×MCR 对应转速换算，简化为 n = n_max × throttle），K_T(J)=K_T0(1-J/J0) 线性近似，J=u/(nD)。推力作用于船尾 x=-Lpp/2。
- **舵力 X_R/Y_R/N_R**：`F_N = 0.5 ρ A_R v_R² f_α sin(δ)`，v_R≈sqrt(u²+（舵处伴流修正）²) 简化为 u×0.8；f_α≈2.5（矩形舵升力线斜率）。Y_R=F_N cosδ，X_R=-F_N sinδ（小角度），N_R = x_R × Y_R（x_R≈-Lpp/2 舵位置）。
- **风载荷 X_W/Y_W/N_W**：`F_Wx = 0.5 ρ_a V_RW² C_X(ψ_w) A_T`，`F_Wy = 0.5 ρ_a V_RW² C_Y(ψ_w) A_L`，`N_W = 0.5 ρ_a V_RW² L C_N(ψ_w) A_L`。风力系数用简化三角函数近似（迎风 C_X 大、横风 C_Y 大），A_T/A_L 来自船型数据。
- **流载荷 X_C/Y_C/N_C**：将流速矢量转到随船坐标系，视为均匀来流产生的等效低速阻尼力，`F_C ≈ 0.5ρ d L |V_rel| V_rel × C`（C 取 0.5 量级），方向与相对流速反向。简化实现即可。
- **拖轮推力**：用户滑块给定横向力 F_tug(kN)，作用于船中或船首，产生 Y_tug 与 N_tug = x_tug × F_tug。

### 4.4 积分与数值鲁棒
- **RK4 积分**，dt=0.05s（物理时间），渲染每帧调用 N 步以匹配加速比。
- **每步后 clamp**：u∈[-15, 15] m/s，v∈[-5,5]，r∈[-0.5,0.5] rad/s。任何力/速度出现 NaN/Infinity 时，该分量重置为 0 并记录警告（不崩溃）。
- **坐标变换**：大地速度 `dx/dt = u cosψ - v sinψ`，`dy/dt = u sinψ + v cosψ`。psi 归一化到 [-π,π]。
- **靠泊检测**：当船尾（x=-Lpp/2 处）距码头线法向距离 < 0.5m 且法向速度 < 0.05m/s 时判定"靠泊成功"；若法向速度超过阈值则判定"碰撞/危险靠泊"并记录护舷能量 `E=0.5 M_eff v_n²`。

### 4.5 自动靠泊规则引擎
- 分段控制：远距离（>200m）保持航向对准泊位延长线，中速；中距离（50-200m）减速至 2-3kn，微调艏向；近距离（<50m）减速至 <1kn，拖轮横向修正，使法向速度 < 0.1m/s。
- 简单 PID 式舵角控制：目标艏向 = 泊位走向 + 小角度修正，舵角 = Kp×(psi_target - psi) + Kd×r，clamp 到 ±35°。
- 主机根据距离自动减速：throttle = clamp(distance/200, 0, 1) × 0.3（靠泊阶段低油门）。

## 5. 数据内嵌规格
构建代理 Read 两份报告后，把数据整理为以下 JS 常量（写在 `<script>` 顶部，注释标明来源）：
```js
const SHIP_LIBRARY = [ {id, name, type, LOA, Lpp, B, D, d, displacement_ton, power_kw, service_speed_kn, rudder_area_m2, prop_dia_m, blades, A_T, A_L, mx_coef, my_coef, Jz_coef, n_max_rps, source} ];
const PORT_LIBRARY = [ {id, name, country, berth, heading_deg, length_m, depth_m, fender_type, design_berth_speed_ms, current_kn, wind_dir, source} ];
const ACCIDENT_CASES = [ {id, date, ship, type, port, summary, cause, casualties, url, tags} ];
const SAFETY_THRESHOLDS = { normal_approach_ms: {roro:0.2, ferry:0.2, psv:0.15, tanker:0.1}, transverse_ms:0.3, yaw_rate_dps:5, wind_limit_ms:12, current_limit_kn:2, source };
const MARKET_DATA = { market_size_2024_usd, cagr, europe_share, apac_share, simulator_cost_range, sources:[] };
```
所有数据点在 UI 展示处标注来源（卡片底部小字）。

## 6. 安全与交互
- 所有数字输入 `type="number"` 带 min/max，JS 二次校验，非法值 toast 提示并拒绝。
- 用户输入（API key、自定义船名等）只用 `textContent` 渲染，不拼 HTML。
- 所有按钮有真实功能，无僵尸按钮。
- 响应式：≥1024px 三栏仿真布局；768-1024px 两栏；<768px 单栏堆叠，Canvas 宽度 100%。
- 仿真 Canvas 用 `devicePixelRatio` 适配高清屏。

## 7. 自检要求（构建完成后必须执行）
1. 用 Python 写一个轻量语法检查：提取 `<script>` 内容，用 `node --check` 或简单括号匹配验证无语法错误（若 node 不可用则跳过）。
2. 运行 `python C:\Users\刘亦扬\AppData\Local\Doubao\User Data\Default\.doubao\agent_mode\workspace\.skills\html\scripts\shot.py <html_path>` 生成桌面+移动截图，Read 截图确认：无布局溢出、无空白页、深色风格正确、仿真视图三栏成立。
3. 确认文件中无 `http://` 外部脚本引用、无 `eval(`、无 `new Function`、无硬编码 key。
4. 在最终回复中报告：文件路径、文件大小、内嵌船型/港口/事故数量、自检截图是否通过、有无未实现项。

## 8. 禁止事项
- 禁止引用任何外部 JS/CSS CDN。
- 禁止用 AI 生成图片当船型/港口照片（用 SVG 示意图即可）。
- 禁止编造报告中没有的数据；报告中"未公开"的字段在 UI 显示"—"并标注。
- 禁止把 API key 写死在代码里。
