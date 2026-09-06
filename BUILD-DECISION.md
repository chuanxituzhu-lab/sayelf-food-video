# sayelf · 构建决策记录

版本：1.5.0 · 日期：2026-09-06 · 记录在实现之前；本次为对 v1.4.0 的增量升级。

- **真实任务**：把美食拍摄意图与有依据的创业故事转为图片 Prompt、按时间段编排的视频 Prompt。交付离线单 HTML 与可独立使用的 Skill。
- **最接近能力**：MoneyPrinterTurbo（主题到成片）、short-video-maker（场景数组与配置）、Story2Video（场景拆解与一致性检查）、ComfyUI（可复用工作流与可拔除节点）；本地 sayelf-visual-narrative README 描述了模型无关的视觉规格与编译边界。当前工作区没有既有实现。
- **Step 0 唯一决策**：Differentiate。
- **可测差异**：单文件 file:// 运行；运行依赖与自动网络请求均为 0；普通用户只填菜品和场景即可生成，也可直接输入新菜名；专业设置渐进展开；真人秀标题与人设按菜品自动切换；专业用户可录入品牌、创意、场景补充并在本机预览参照照片；分镜卡片的 Prompt 默认折叠，点击后可单独复制；支持自动匹配、9:16、3:4、16:9、4:5；四要素、菜品与真人秀证据共同编译；缺失冲突证据不产生冲突事实；已收录菜品使用专属工艺，自定义菜名使用川味通用四段工艺并标注建议；6–60 秒连续时间轴，首尾与用户时长精确一致；11 种菜品示范。
- **成功证据**：核心独立执行测试、11 个示范（含 6 道川菜）、每道川菜四段工艺与器皿/场景、动态真人秀定位、品牌/创意/场景/照片文件名进入双 Prompt、图片关键帧与视频分镜逐一匹配、画面比例枚举与自动匹配、普通/专业两条 UI 路径、边界时长、各开关、证据缺失/存在、跨镜头一致性、输入改动后禁止导出旧结果、离线浏览器操作、导入导出回环与 ZIP 检查。
- **Minimum Core**：normalize → validate → select beats → timeline → image/video compile → output validate。只做确定性文本生成；不承诺模型画面质量或传播结果。
- **插件边界**：菜品词库、平台编辑建议、Seedance 风格文本序列化为可替换数据/函数；真人秀、ASMR 可关。Core 不读 DOM、不访问网络和存储。无第三方插件加载器。
- **本地边界**：所有表单运算在浏览器内存；只由用户点击导入或下载本地 JSON/TXT；无自动持久化，无账户、API key、遥测、上传。
- **数据分级**：本次上传的仓库文件明确分类为 Public；用户课堂截图、原会话和用户输入按 Unknown/本地资料处理；个人与商业细节可为 Sensitive。截图及会话原文不打包；运行时用户输入仍属于本地 Internal 数据。
- **GitHub/public release**：用户已明确授权上传到 `https://github.com/chuanxituzhu-lab/sayelf-food-video`；本次仅上传已分类为 Public 的商品文件。上传前审查文件列表、绝对路径、凭据、媒体和 ZIP 内容；不上传截图、原会话、表单或测试上下文。
- **其他外传**：GitHub 传输仅限本次用户指定仓库和 Public 商品文件；不发送截图、原会话、表单或本地测试目录。产品运行仍无外传路径。
- **状态与检查**：EMPTY/DIRTY → VALIDATING → READY 或 NEEDS_REVIEW/ERROR；只在编辑、载入、生成、导出时检查。无定时轮询；输入变更立即标旧并禁用结果导出。
- **认知边界**：Observation 为用户提交的事件描述；Inference 为剪辑安排；Hypothesis 为示范及创作建议；Fact 仅为可程序验证的时长/字段结果。用户勾选证据可用不等于机器核实事件。
- **演进/回滚**：v1.5.0 固定规则；基于真实反馈提出变化，先单个示范与自定义菜名回归后升级；保留旧 HTML 与 JSON，替换文件即可回滚。不自动更新；后续 schema 不兼容时明确拒绝导入。
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

检索日期：2026-09-05。范围：GitHub 仓库 README 与公开文档；未安装运行这些仓库，功能是文档陈述，不是本次实测。搜索采用通用词 short video generator / storyboard / workflow。以下仅借鉴机制，未复制实现、提示词模板、素材或依赖代码。

| 仓库 / 来源 | 观察到的机制 | 蒸馏到本产品 | 不引入的部分 |
|---|---|---|---|
| [MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo) | 脚本、素材、声音、合成为阶段；允许自定义脚本和独立音量配置 | 输入与输出分阶段；配乐、字幕、现场声音单独控制 | 云模型、素材搜索、合成与发布服务 |
| [short-video-maker](https://github.com/gyoridavid/short-video-maker#concepts) | Scene 数组分离文本与素材查询；REST/MCP 提供创建与状态 | 使用 shots[] 作为中间结果；显式 READY/ERROR 状态 | Pexels、TTS、Remotion、服务器 |
| [Story2Video](https://github.com/approximatelylinear/story2video#how-it-works) | 场景含动作、心境、台词、时长；跨镜头风格检查；后端可换 | 先场景规格后双 Prompt；人物/餐具/空间一致性约束 | LLM、SQLite、多个云端视频后端 |
| [ComfyUI](https://github.com/Comfy-Org/ComfyUI#features) | JSON 工作流保存加载、离线核心、自定义节点 | JSON 配置回环；Core 与词库/输出适配隔离 | 节点画布、模型管理、GPU 推理 |

**差距判断（Inference）**：上述已读资料未展示“离线单 HTML + 中文美食四要素 + 真实事件证据门控 + 双提示词出口”的完整组合。该判断限于检索范围，不宣称全球首创。

**授权与许可边界**：调研不等于代码再分发许可；未复制第三方实现，未打包任何仓库。未来接入或复制时须单独核查版本许可证。sayelf 是本交付品牌，不代表与上述项目或 Seedance 官方合作。
