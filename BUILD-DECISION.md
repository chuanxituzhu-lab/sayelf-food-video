# sayelf · 构建决策记录

版本：1.15.0 · 日期：2026-09-20 · 记录在实现之前；本次为对 v1.14.0 的增量升级。

- **真实任务**：把美食拍摄意图与有依据的创业故事转为图片 Prompt、按时间段编排的视频 Prompt。交付离线单 HTML 与可独立使用的 Skill。
- **本次增量**：按工作流原则复用现有 Core 的唯一结果和校验器，将用户提供的两张课堂参考图蒸馏为“料理卡、工艺序列、动作级镜头、剪辑/声音/负面约束、角色与场景锚点”；新增事件/故事线索主入口，先由事件节点决定分镜数量，再由主体/动态/镜头/风格字段表达，不让固定四要素反推事件；为 UI 预留共享 GSAP Motion Layer，动效从统一入口调用，不复制新的 Skill 或动效底座；本次把现有校验、运行记录、检查点组合成 FDE 现场交付卡。
- **本次增量（v1.14.0）**：复用既有故事节点加权时间轴，把普通镜头约 2 秒设为软参考；不固定每镜 2 秒，实际时长继续由故事节点、动作完成度、情绪停顿与感官/成品关键点分配，并在元数据中记录 `timingReference='soft'` 与 `targetShotSeconds=2`。
- **本次增量（v1.15.0）**：增加本机品牌知识库适配层；品牌档案包含品牌名、创意上下文、品牌口吻、视觉规则和连续性规则，普通用户可直接调用，专业用户可保存、删除、导入和导出。档案在生成前作为显式 `brandProfile` 上下文注入 Prompt，Core 仍不读 DOM、存储或网络。
- **最接近能力**：MoneyPrinterTurbo（阶段化素材/声音）、short-video-maker（场景数组）、Story2Video（结构化场景与一致性）、ComfyUI（可复用工作流）；本次补查 StoryMind（先规划逐镜镜头）、SeedDance Studio（首尾帧串接）、VibeStudio（参考绑定与连续性）、ai-visual-director（镜头与 QC 约束）、OpenCanvas（连续性状态与 QA 选择）、Seedance shot-list continuity（参考绑定、接入/交出状态与失败后拆分动作）、FDE Guide（现场观察、价值基线、验收证据、交付与回滚），以及 GSAP 官方 `to/fromTo/timeline/context` 文档的共享补间与清理边界。它们提供机制参考，当前工作区仍无可直接复用的本地料理卡、FDE 交付卡或动效适配实现。
- **本次品牌库调研**：promptforge-v2 的本地可移植资料库与版本化存储、Cabinet 的本地可见与可携带知识库；只蒸馏“本机保存、显式导入/导出、按需注入”的机制，不引入云端向量库、账号体系或第三方实现。
- **Step 0 唯一决策**：Improve。
- **可测差异**：单文件 file:// 运行；运行依赖与自动网络请求均为 0；普通用户先填事件、菜品和场景即可生成，也可直接输入新菜名；四要素栏从普通界面隐藏，专业设置渐进展开并可作为表达覆盖；事件节点先决定分镜数量并写入 `meta.eventFirst / shotCountSource / shotCountReason`；其余既有料理卡、连续性、动态分工、比例与双 Prompt 能力保持。
- **可测差异**：在既有单 HTML 和动态角色基础上，每个结果新增一张可审阅的料理卡（食材摘要、做法、工艺序列、时间逻辑、角色/场景锚点），每个镜头新增移动/剪辑/SFX/VFX 字段；图片和视频 Prompt 同时携带这些字段，且不增加外部依赖、网络请求或渲染服务。
- **可测差异（v1.12.0）**：HTML 暴露一个 `globalThis.SayelfMotion` 共享入口，GSAP 仅在宿主已注入时复用；独立 file:// 商品不含 CDN/外链脚本，未注入时同一入口原生降级，减少动效时停用。动效不写入 Core、Skill 输出或状态机，因此不会重复建设动效底座或改变故事结果。
- **可测差异（v1.13.0）**：专业输出新增一张默认折叠的 FDE 现场交付卡，固定呈现 `目标 → 当前状态 → 验收门 → 参考/连续性 → 下一步 → 恢复点 → 交接结果`；普通路径交互数不增加，卡片只读取现有校验、运行记录和检查点，不新增网络、后台或数据上传。
- **可测差异（v1.14.0）**：12 秒切片回归必须同时满足 `targetShotSeconds=2`、`timingReference='soft'`、多数镜头落在 1.5–3.5 秒且至少存在一个非 2 秒镜头；故事节点数量测试继续证明分镜先于时长分配，不增加依赖或网络请求。
- **可测差异（v1.15.0）**：品牌档案可以在同一浏览器跨次打开后继续调用；保存/调用回归检查档案字段进入中英文 Prompt，导入/导出使用 `sayelf-food-video/brand-kb/v1`，删除后生成不再携带旧档案；不增加服务器、账号或外部请求。
- **成功证据**：核心独立执行测试、11 个示范（含 6 道川菜）、每道川菜四段工艺与器皿/场景、动态真人秀定位、品牌/创意/场景/照片文件名进入双 Prompt、图片关键帧与视频分镜逐一匹配、故事节点先决定分镜数量再分配总时长、电影化节奏与特殊镜头速度/技法、动态工作流角色随内容信号变化且手动模式保持严格校验、料理卡字段与工艺序列、逐镜移动/剪辑/SFX/VFX 与负面约束、共享动效入口及无 GSAP 原生降级、模拟宿主注入的 `context → fromTo → kill/revert` 分支、FDE 交付卡的渐进显示与七项交接字段、最新版下载链接、内嵌品牌图标与浏览器图标、画面比例枚举与自动匹配、普通/专业两条 UI 路径、边界时长、各开关、证据缺失/存在、跨镜头一致性、角度/移动方式/方向无完全重复、动作/视线/器物/声音桥转场约束、输入改动后禁止导出旧结果、离线浏览器操作、内嵌品牌图标检查、深色工作台视觉检查、导入导出回环与 ZIP 检查。
- **Minimum Core**：normalize → validate → select beats → timeline → image/video compile → output validate。只做确定性文本生成；不承诺模型画面质量或传播结果。
- **v1.11.0 增量证据**：普通 UI 不再展示四要素栏；事件输入进入统一 Core，事件节点数量改变 `shots/imageFrames` 数量；`meta.eventFirst`、`meta.eventSupplied`、`meta.shotCountSource` 与 `meta.shotCountReason` 可审阅；双 Prompt 追加事件主线与事件优先规则。未引入第三方代码、图片或在线渲染。
- **v1.13.0 增量证据**：FDE 交付卡只在专业输出中展开，目标来自事件/菜品/场景，验收来自现有结构校验，下一步来自警告与人工门，恢复点来自现有检查点；未改变 Core 输出 schema 或普通用户流程。
- **v1.14.0 增量证据**：Core 与 shipping HTML 的 `meta.timingReference`、`targetShotSeconds`、视频 Prompt 文案和独立测试同步；故事节点加权结果保留自然差异，未改变输入 schema 或普通用户路径。
- **插件边界**：菜品词库、平台编辑建议、Seedance 风格文本序列化和品牌知识库适配为可替换数据/函数；真人秀、ASMR 可关。GSAP Motion Layer 只属于 UI 共享能力，Core/Skill 不依赖它，也不把它注册为新 Skill。Core 不读 DOM、不访问网络和存储。无第三方插件加载器。
- **本地边界**：表单运算在浏览器内存；品牌档案仅由 UI 适配层写入当前浏览器本机存储，并提供显式删除/导入/导出；无账户、API key、遥测、上传。
- **数据分级**：本次上传的仓库文件明确分类为 Public；用户课堂截图、原会话和用户输入按 Unknown/本地资料处理；个人与商业细节可为 Sensitive。截图及会话原文不打包；运行时用户输入仍属于本地 Internal 数据。
- **GitHub/public release**：用户已明确授权上传到 `https://github.com/chuanxituzhu-lab/sayelf-food-video`；本次仅上传已分类为 Public 的商品文件。上传前审查文件列表、绝对路径、凭据、媒体和 ZIP 内容；不上传截图、原会话、表单或测试上下文。
- **其他外传**：GitHub 传输仅限本次用户指定仓库和 Public 商品文件；不发送截图、原会话、表单或本地测试目录。产品运行仍无外传路径。
- **状态与检查**：EMPTY/DIRTY → VALIDATING → READY 或 NEEDS_REVIEW/ERROR；只在编辑、载入、生成、导出时检查。无定时轮询；输入变更立即标旧并禁用结果导出。
- **认知边界**：Observation 为用户提交的事件描述；Inference 为剪辑安排；Hypothesis 为示范及创作建议；Fact 仅为可程序验证的时长/字段结果。用户勾选证据可用不等于机器核实事件。
- **演进/回滚**：v1.15.0 在 v1.14.0 的事件先行、软节奏、共享动效入口和 FDE 交付卡边界上增加本机品牌档案；先完成品牌保存/调用/删除/导入/导出和跨语言回归，再升级；保留旧 HTML、Skill、配置和品牌库导出即可回滚。不自动更新；后续 schema 不兼容时明确拒绝导入。
- **WebUI**：Required — 用户明确要求独立表单工具，便于拍摄者选择菜品、证据与分镜。默认 Open → Input → Execute → Result；高级故事证据与输出细节折叠。
- **最简单实现**：内嵌原生 HTML/CSS/JavaScript；不依赖服务器、CDN、模型或打包器。
- **明确不做**：视频渲染、图像生成、语音克隆、自动发布、素材上传、付费接口、爆款预测、复杂节点编辑器、桌面安装器。

## 八项原则逐条落实

已读取本地 sayelf-build-principles 的 SKILL.md、README.md 与 AGENTS.md；以 SKILL.md 为准。README 中残留的 “seven principles” 小标题不作为改写依据，正文实际包含 01–08。

| 原则 | 本产品落实 |
|---|---|
| 01 负熵 | 只保留输入、编译、检查、复制/导出 |
| 02 模块化/可插拔 | Core 纯函数；菜品/平台词库和导出器边界；移除 UI 仍可运行 |
| 03 Local-first | 零外部运行依赖，断网可生成 |
| 04 状态驱动 | 修改标旧、生成校验、错误阻断；不轮询 |
| 05 有边界的智能自动化 | 建议、用户陈述、示范及结构事实分开；证据不足降级 |
| 06 证据驱动演进 | 版本、聚焦验证、单示范灰度及文件回滚 |
| 07 WebUI 人机界面 | 表单→生成→结果；进阶项渐进展开 |
| 08 本地与敏感数据主权 | 无上传/遥测；用户控制本地文件；公开发布另设门禁 |

## GitHub 调研与机制蒸馏

检索日期：2026-09-19。范围：GitHub 仓库 README 与公开文档；未安装运行这些仓库，功能是文档陈述，不是本次实测。搜索采用 food video / storyboard / continuity / workflow / FDE 等通用词。以下仅借鉴机制，未复制实现、提示词模板、素材或依赖代码。

| 仓库 / 来源 | 观察到的机制 | 蒸馏到本产品 | 不引入的部分 |
|---|---|---|---|
| [MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo) | 脚本、素材、声音、合成为阶段；允许自定义脚本和独立音量配置 | 输入与输出分阶段；配乐、字幕、现场声音单独控制 | 云模型、素材搜索、合成与发布服务 |
| [short-video-maker](https://github.com/gyoridavid/short-video-maker#concepts) | Scene 数组分离文本与素材查询；REST/MCP 提供创建与状态 | 使用 shots[] 作为中间结果；显式 READY/ERROR 状态 | Pexels、TTS、Remotion、服务器 |
| [Story2Video](https://github.com/approximatelylinear/story2video#how-it-works) | 场景含动作、心境、台词、时长；跨镜头风格检查；后端可换 | 先场景规格后双 Prompt；人物/餐具/空间一致性约束 | LLM、SQLite、多个云端视频后端 |
| [ComfyUI](https://github.com/Comfy-Org/ComfyUI#features) | JSON 工作流保存加载、离线核心、自定义节点 | JSON 配置回环；Core 与词库/输出适配隔离 | 节点画布、模型管理、GPU 推理 |
| [StoryMind](https://github.com/LinHao-city/StoryMind) | 先由 Storyboard Director 逐镜决定景别、运镜、灯光和角色描述，再进入生成 | 将“先规划再生成”落实为料理卡、工艺序列与逐镜字段 | LLM 导演、素材检索、视频渲染 |
| [SeedDance Studio](https://github.com/amo613/seedance-storyboard) | 多场景分镜、首尾帧控制、上一镜末帧串接、场景级音频开关 | 将首/尾状态和器物/动作承接写进可交接的连续性提示 | OpenRouter、ffmpeg、云端生成 |
| [VibeStudio](https://github.com/vericontext/vibestudio) | 参考图目录、`@Image` 绑定、上一镜视频/末帧连续性、逐镜声音设计 | 保留本地参考说明、角色/场景锚点和声音字段，交给外部工具时再绑定 | Provider CLI、生成资产库、云模型 |
| [AI Visual Director](https://github.com/jijiutong/ai-visual-director) | 镜头技法库、角色/场景一致性与多项 QC | 增加移动/剪辑/声音/负面约束和可读的检查字段 | 大型样式库、模型路由、复杂图形编辑器 |
| [OpenCanvas](https://github.com/robinrheem/opencanvas) | 连续性状态记忆、规划/检索/生成/QA 选择 | 用轻量角色/场景锚点与输出验证保持连续性 | 持久视觉记忆、模型生成与复杂多代理选择 |
| [Seedance shot-list continuity](https://github.com/Emily2040/seedance-2.0/blob/main/references/shot-list-continuity.md) | `@Image`/`@Audio` 参考、接入/交出状态、失败后拆分动作 | 把参考绑定、动作承接、声音桥和失败兜底写入 Skill 契约 | 特定模型提示模板、在线生成与重试服务 |

**差距判断（Inference）**：上述已读资料未展示“离线单 HTML + 中文美食四要素 + 真实事件证据门控 + 双提示词出口”的完整组合。该判断限于检索范围，不宣称全球首创。

**授权与许可边界**：调研不等于代码再分发许可；未复制第三方实现，未打包任何仓库。未来接入或复制时须单独核查版本许可证。sayelf 是本交付品牌，不代表与上述项目或 Seedance 官方合作。
