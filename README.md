# 晶晶 · 立达标讯招投标情报分析师（Lida Biaoxun Tender Intelligence Analyst）

**主要职责是「获取项目数据 + 项目初筛」**：针对**立达标讯平台（www.arrbid.com）**工作，自动采集该平台的采购公告，按《投标策略》四维模型做初筛，把平台每天新增的公告变成一份「值得跟进 / 不值得跟进」的清单，命中即通过企业微信推送提醒，并监控采集侧与写入侧的异常。

除日常初筛外，另有两件**按需**的事：把公司的投标经验**沉淀成结构化《投标策略》**（并做策略体检），以及对重点冲突项目做**方案层面的深度分析**。

> 职责边界：**初筛**回答「进 / 不进」，**深度分析**回答「怎么投」；投标的最终报价与投标决策仍由公司做出。

## 类型

Agent 型（单个 AI 专家），分类：销售商务（07-SalesCommerce）。数据源：立达标讯平台（arrbid.com）。

## 功能

- **立达标讯平台数据采集调度**：启停爬虫采集该平台最新公告、查询采集状态、监控附件下载异常与程序异常退出。
- **项目初筛（主职责）**：为每个待处理项目生成素材包（正文 + 附件 + 图片 OCR），按「关系单位 × 业务对口 × 资质门槛 × 采购方式」四维模型给出「推荐参加 / 考虑 / 关注（后续参加）/ 不参加」初筛结论；已过投标截止的项目走快速通道直接筛除。
- **《投标策略》梳理与体检（按需）**：访谈式把公司的业务范围、关系单位、一票否决规则、价格分取舍沉淀成结构化配置并渲染成策略文档；自动检查策略的自相矛盾、残缺与可疑过时项。
- **单项目投标深度分析（按需）**：评分结构应对、优势与劣势包装、风险与门槛、竞争格局判断、报价思路区间、待补信息清单。
- **命中推送与异常提醒**：命中项目自动生成提醒文案（含精简公告摘要）并通过企业微信推送；每批结束推送进度简报；采集侧下载异常/崩溃时主动告警。
- **写入健康监控**：项目目录与数据库若位于**网络映射盘/UNC 共享**，写入会先做写后校验；失败时不崩溃、不卡锁，而是返回告警并（磁盘恢复后）自动补写。

> 三件事的关系：**日常跑的是初筛**；策略梳理与深度分析是**按需**触发的只读作业，不改动项目队列状态，因此不会和初筛链路抢锁。

### ⛔ 策略闸门（未提供策略就不开工）

**《投标策略》没配好之前，采集与分析都会被拒绝**——留空时脚本会回退到包内占位模板，
筛出来的是对客户**无效的错误结论**，不如先把策略配好：

| 命令 | 未提供策略时 |
|---|---|
| `jrzb.py start`（采集） | **拒绝启动**，退出码 `5`（应急放行：`start --allow-without-strategy`） |
| `ztb.py next`（分析） | **拒绝发放项目**，退出码 `5`、`error: strategy_required`（应急：`next --ignore-strategy`） |
| 只读监控与分析取号外的命令 | 不受影响（`status` / `check-io` / `strategy` / `save` / `push` 等） |

**「已提供」的判据**：`strategy.json` 存在，且四维模型的四个核心字段
（`业务范围` / `关系单位` / `硬性否决规则` / `价格分规则`）**至少一个非空**——
只填了「公司名称」这类非核心字段、或空文件，仍算未提供。
`status` / `setup` 输出里都有 `strategy_gate` 块，能直接看到 `blocked` 与还缺哪几项。

## 使用示例

**先建会话**：在招投标项目里建立一个沟通任务（一个项目建一个任务，也可以长期共用一个任务），
之后都在这个任务里跟晶晶说话。下面几组是最常用的开场话术。

| 你说 | 晶晶会做什么 |
|---|---|
| **显示今天推荐跟进的项目** 〔①〕 | 整理出今天的推荐跟进项目结果，直接给你看，自由阅读 |
| **跟进衢江美食街项目** 〔②〕 | 说项目名称或项目 ID 即可；她会把该项目标记为**跟进中**，并持续记录这个项目的跟进情况。反之，说「取消跟进」也可以撤掉标记 |
| **显示跟进项目列表** 〔③〕 | 列出所有跟进中的项目，并按截止投标时间给出 **5 天倒计时**提醒 |
| **详细介绍衢江美食街项目，并在互联网上搜索成功案例** | 结合项目的招标信息做详细介绍，并借助联网检索同类成功案例做交叉对比（也可以指定在企查查等平台查） |
| **将此文件归档到衢江美食街项目，并结合投标要求，进行标书编写思路建议** | 识别文件与项目的关联性并归档，再结合该项目的投标要求给出标书编写思路 |

> 〔①②③〕同时是平台上的「试试这样问我」快捷提示（`plugin.json` 的 `quickPrompts`，
> 第一条也就是进入专家时的默认引导语）。

除上面这些日常用法外，**梳理《投标策略》**与**单项目投标深度分析**是两条**按需**入口，
同样直接说人话即可（如「梳理并保存我们公司的投标策略」「对某个项目做投标深度分析」）。
但记得先过「策略闸门」——**策略没配好，采集与初筛都会被拒绝**。

## 捆绑技能

| 技能 | 作用 |
|------|------|
| `jrzb-crawler` | 爬虫程序（`xCD.WB.Skill.JRZB.exe`）及**安装时自动下载的运行环境**的部署、启停、状态查询、下载异常与异常退出监控 |
| `ztb-analyzer` | 读取 `buzdata.db` 待办项目，按《投标策略》四维模型初筛并落库、判定推送；投标策略向导与渲染 |

## 运行环境与依赖

| 项 | 要求 |
|---|---|
| 操作系统 | **Windows**（爬虫程序为 Windows 可执行文件；分析脚本本身跨平台） |
| Python | 3.9+（PDF 附件解析需 `pymupdf`，可选） |
| 爬虫程序 | `xCD.WB.Skill.JRZB.exe` —— 运行环境约 70MB：开放平台版**不随包分发**，安装时按 `crawler/runtime-source.json` 的地址自动下载并校验 SHA256；**GitHub 完整版已内置**，解压即用。离线环境均可手工放置运行包。自带 .NET 运行时，**无需另装 .NET** |
| 数据库 | `buzdata.db`（SQLite，由爬虫采集生成，**不在本包内**，首次运行自动生成） |
| 企业微信 | **可选**。要在微信里收推送/告警才需要：启用「企业微信」连接器且授权状态为 `authorized`。未启用不影响采集与初筛 |
| 其他 | 本机需装 **Google Chrome**（爬虫通过 CDP 附着到 Chrome 采集） |

> ⚠️ **数据库不在包内**：`buzdata.db` 是采集产生的业务数据，不随包分发；
> 爬虫首次运行时会在工作目录自建。
> ⚠️ **包内不含任何账号密码**：本机个性化配置 `xCD.WB.Skill.JRZB.dll.config` 已从运行包中剔除，
> 由安装向导生成空骨架后由你填写自己的值。

## 安装（接收方）

1. 解压专家包，把 `lida-biaoxun-bidding-analyst/` 整个目录放到：

   ```
   <WorkBuddy 配置目录>/plugins/marketplaces/my-experts/plugins/
   ```

   Windows 默认即 `C:\Users\<你的用户名>\.workbuddy\plugins\marketplaces\my-experts\plugins\`
   （若配置了 `WORKBUDDY_CONFIG_DIR` 环境变量，则以该变量为准）。

2. 注册，使专家在「专家中心 - 我的专家」可见：

   ```bash
   python <expert-manager>/scripts/register_expert.py <专家目录>
   ```

   `expert-manager` 位于 WorkBuddy 安装目录下的
   `resources/app.asar.unpacked/resources/plugins/workbuddy-builtin/skills/expert-manager/`。

3. **（推荐）工作区自包含安装**：一条命令把技能、爬虫、配置全部锚定到工作区 —— 见下一节
   《📁 工作区锚定》。**做了这一步就可以跳过第 4、5 步的手工配置**，直接去做爬虫个性化向导与策略向导。

3. **部署爬虫运行环境（一条命令，首次需要联网下载运行包）**：

   ```bash
   # 解包到指定工作目录（推荐：先建好目录，如 D:\jrzb_spider）
   python skills/jrzb-crawler/scripts/jrzb.py setup --install-crawler "D:\jrzb_spider"

   # 或解包到当前配置的工作目录（默认 ~/jrzb_spider）
   python skills/jrzb-crawler/scripts/jrzb.py setup --install-crawler
   ```

   运行包（`xCD.WB.Skill.JRZB.exe` + .NET 运行时 + Playwright 驱动，约 70MB）在开放平台版里
   **不在专家包内**（平台 20MB 上传上限装不下）；GitHub 完整版已内置在 `crawler/` 下，
   命令会优先使用它、不联网。开放平台版则会按 `crawler/runtime-source.json` 里的
   地址下载并做 SHA256 校验，解包后自动生成一份空的 `app.config` 骨架。

   > ⚠️ **首次安装需能访问运行包下载地址**。若本机没有外网/内网通道，见下方
   > 《运行包的获取方式》一节改用离线方式。

   | 解包规则 | 说明 |
   |---|---|
   | 已存在文件 | **默认保留**，不覆盖你已配好的 `app.config`；需覆盖时加 `--force` |
   | `buzdata.db` | **永不写入**（属采集业务数据，首次运行时由爬虫自建） |
   | `*.dll.config` | **永不写入**（属本机个性化配置，含账号密码） |

#### 运行包的获取方式

运行包体积约 70MB（含 .NET 8 运行时与 Playwright 驱动）。**两种分发形态**：

| 形态 | 体积 | 运行包位置 |
|---|---|---|
| 开放平台版（轻量） | < 1MB | 包内**不含**，安装时按下方优先级联网下载 |
| GitHub Release 完整版 | 约 70MB | 已内置于 `<专家包>/crawler/jrzb-runtime.zip`，**解压即用、全程无需联网** |

> 判断手上是哪种：看 `<专家包>/crawler/` 下有没有 `jrzb-runtime.zip`。两种形态的
> 安装命令完全一样 —— 脚本会按下面的优先级自动挑，完整版命中优先级 2，直接解包不会联网。

安装时按以下优先级自动寻找，找到即用、都不需要就报错：

| 优先级 | 来源 | 说明 |
|---|---|---|
| 1 | 环境变量 `JRZB_RUNTIME_BUNDLE` | 直接指向本地 zip 文件（离线部署常用） |
| 2 | `<专家包>/crawler/jrzb-runtime.zip` | 手工把运行包放回这里（离线部署） |
| 3 | `<state_root>/jrzb-runtime.zip` | 上次下载的缓存，直接复用，不重复下载 |
| 4 | `crawler/runtime-source.json` 里的 `url` | 联网下载，带 SHA256 强制校验（**默认路径**） |

常用命令：

```bash
# 默认：读 runtime-source.json 下载 + 校验
python skills/jrzb-crawler/scripts/jrzb.py setup --install-crawler "D:\jrzb_spider"

# 临时换地址（优先于 runtime-source.json）
python skills/jrzb-crawler/scripts/jrzb.py setup --install-crawler --runtime-url "https://.../jrzb-runtime-2.9.0.zip"

# 强制指定校验值 / 完全离线（只用本机已有文件）
python skills/jrzb-crawler/scripts/jrzb.py setup --install-crawler --runtime-sha256 "<HASH>"
python skills/jrzb-crawler/scripts/jrzb.py setup --install-crawler --no-download
```

> 校验不通过（SHA256 不匹配）会立刻删除下载的临时文件并报错，不会解包半个包。
> 环境变量 `JRZB_RUNTIME_URL` / `JRZB_RUNTIME_SHA256` 可覆盖清单里的同名配置。

> **手工下载（不装专家包、只想单独部署爬虫的使用者）**：
> 运行包文件名 `jrzb-runtime-2.9.0.zip`，下载地址：
> `http://116.62.68.63/tools/jrzb-runtime-2.9.0.zip`
> （同一地址也写在 `crawler/runtime-source.json` 的 `url` 字段，换版本时以该文件为准）。
> 下载后请核对 SHA256 是否等于
> `81b71538a22aa99b2d5950097565af09de947189542eee852dde42f793f94864`，Windows 下可执行
>
> ```bat
> certutil -hashfile jrzb-runtime-2.9.0.zip SHA256
> ```
>
> 下载后有两种用法：① 直接解压到目标目录（如 `D:\jrzb_spider`），再
> `python skills/jrzb-crawler/scripts/jrzb.py config --work-root "D:\jrzb_spider"`；
> ② 把 zip 重命名为 `jrzb-runtime.zip` 放进 `<专家包>/crawler/`，之后
> `setup --install-crawler` 会自动用它，不再联网。
> 解压出来的就是完整运行环境（自包含 .NET 8 + Playwright 驱动），**无需另装 .NET**。

4. **配置本机路径**（关键，否则脚本找不到数据）：

   ```bash
   # 方式一：给出项目根目录（内含爬虫程序与 buzdata.db）
   python skills/jrzb-crawler/scripts/jrzb.py  config --work-root "D:\你的\项目根目录"
   python skills/ztb-analyzer/scripts/ztb.py   config --work-root "D:\你的\项目根目录"

   # 方式二：分别指定数据库与程序
   python skills/ztb-analyzer/scripts/ztb.py   config --db "D:\...\buzdata.db"
   python skills/jrzb-crawler/scripts/jrzb.py  config --exe "D:\...\xCD.WB.Skill.JRZB.exe"
   ```

   配置写入共享文件：优先 **`<工作区>/.workbuddy/zb-tender.json`**（工作区自包含安装会写这里），
   否则退回 `~/.workbuddy/zb-tender.json`。两个技能共用同一份。

5. **配置爬虫个性化参数（安装向导，必做）**：

   爬虫自己还有一份个性化配置 `xCD.WB.Skill.JRZB.dll.config`（与 exe 同目录），
   里面的招标网站账号密码、Chrome 路径、用户数据目录、工作目录等**绑定具体使用者环境**，
   每台机器都不同。先查看需确认的项，再逐项写回：

   ```bash
   python skills/jrzb-crawler/scripts/jrzb.py setup          # 列出必问项 + 本机体检
   python skills/jrzb-crawler/scripts/jrzb.py setup \
     --set "登录账号=13800000000" --set "密码=******" \
     --set "ChromePath=D:\Google\Chrome\Application\chrome.exe" \
     --set "UserDataDir=D:\ChromeProfile" --set "工作目录=D:\你的\项目根目录"
   ```

   写回只改 `value`、保留注释，并自动备份原文件；完成后 `setup` 应显示「已配置完成」。
   **在 WorkBuddy 里首次对话时，晶晶也会主动按向导逐项询问你：先问爬虫环境与配置，再问投标策略。**

6. **配置投标策略（安装向导，必做）**：

   《投标策略》是**客户自己的**业务规则（关系单位、对口业务、一票否决、价格分取舍……），
   每家客户都不同。技能不内置任何客户的策略，需按向导配置后再初筛：

   ```bash
   python skills/ztb-analyzer/scripts/ztb.py strategy            # 列出必问项 + 当前值
   python skills/ztb-analyzer/scripts/ztb.py strategy --template  # 空白模板（照着填）

   # 方式一：逐项写
   python skills/ztb-analyzer/scripts/ztb.py strategy \
     --set "价格分规则.可接受下限=30" --set "价格分规则.不考虑低于=20" \
     --add "关系单位=某某集团及下属公司" --add "关系单位=某某市青少年活动中心" \
     --set "优势=演示视频、效果图、3D演示视频" \
     --set "硬性否决规则=纯设备或物资采购项目，无优势"

   # 方式二：把模板填好后整体导入（长文本推荐，避免命令行转义问题）
   python skills/ztb-analyzer/scripts/ztb.py strategy --set-file 我的策略.json

   # 渲染成《投标策略》（初筛时读它）/ 打印查看
   python skills/ztb-analyzer/scripts/ztb.py strategy --render
   python skills/ztb-analyzer/scripts/ztb.py strategy --print
   ```

   配置写入 `<state_dir>/strategy.json` 并渲染出《投标策略》markdown；
   未配置时回退到包内 `references/投标策略.txt`（**仅作示例/兜底**，切勿当作自己的策略）。
   `--reset` 可清除配置。

7. **核对配置生效**：

   ```bash
   python skills/jrzb-crawler/scripts/jrzb.py config
   python skills/ztb-analyzer/scripts/ztb.py  config
   ```

   检查 `resolved` 里的 `crawler_exe` / `db` 指向正确、`*_exists` 为 `true`，
   `crawler_app_config.needs_setup` 与 `strategy.needs_setup` 均为 `false`。

## 📁 工作区锚定（一条命令，推荐）

**目标**：整套东西（技能 + 爬虫程序 + 数据库 + 状态/日志/策略）都落在**工作区**里，
做到「装到哪台电脑，就落在那个工作区，并长期在那里运行」，工作区可直接打包带走。

```bash
# 在专家包根目录执行（自动识别当前工作区）
python install.py

# 或显式指定工作区
python install.py --workspace "D:\我的工作区"

# 其他开关
python install.py --dry-run      # 只打印计划，不落盘
python install.py --no-skills    # 不把技能复制到工作区（避免与专家包内技能重复）
python install.py --force        # 覆盖工作区里已存在的技能文件
```

它做三件事（**不覆盖**已存在文件；`buzdata.db` 与含账号密码的 `app.config` 永不写入）：

| # | 动作 | 落到哪 |
|---|---|---|
| ① | 复制技能到**项目级** | `<工作区>/.workbuddy/skills/{jrzb-crawler,ztb-analyzer}` |
| ② | 解包爬虫运行环境 | `<工作区>/jrzb_spider/`（程序 + 后续自建 `buzdata.db`） |
| ③ | 共享配置（含 `work_root` / `state_root`） | **`<工作区>/.workbuddy/zb-tender.json`**，两个值都指向工作区内 |

做完后工作区是**自包含**的，一个目录装下所有东西：

```
<工作区>/
├─ jrzb_spider/                 爬虫程序 + .playwright 驱动 + buzdata.db（首次运行自建）
└─ .workbuddy/
   ├─ zb-tender.json            共享配置（work_root / state_root 都指向本工作区）
   ├─ skills/                   项目级技能（jrzb-crawler / ztb-analyzer）
   ├─ jrzb-crawler/             爬虫状态、日志、last_run、告警
   └─ ztb-analyzer/             锁、素材包、strategy.json、渲染出的《投标策略》
```

**工作区是怎么识别的**：读环境变量 `WORKBUDDY_PROJECT_DIR` → `CODEBUDDY_PROJECT_DIR` →
`CLAUDE_PROJECT_DIR`，取第一个真实存在的目录；都取不到就要显式 `--workspace`。

> **脚本默认值也已经改为工作区优先**（不只是 install.py）：
> - `setup --install-crawler` 不指定目录时 → **`<工作区>/jrzb_spider`**（原来是 `~/jrzb_spider`）；
> - 未配置 `state_root` 时默认 → **`<工作区>/.workbuddy`**（原来是 `~/.workbuddy`）；
> - **共享配置查找顺序**：`ZB_CONFIG` 环境变量 → **`<工作区>/.workbuddy/zb-tender.json`（存在才用）**
>   → `~/.workbuddy/zb-tender.json`。所以工作区装好后，技能会**自动**认工作区内的配置，
>   不需要额外设置环境变量，工作区搬走即用。
> - 兼容性：只有在**工作区里确实存在** `jrzb_spider` / 工作区配置时才优先工作区，
>   因此不会打断「按旧方式装在主目录」的既有环境；已显式配置过的机器完全不受影响。
>
> ⚠️ `install.py` **不会创建定时任务**——定时任务只能由 WorkBuddy 平台能力创建，
> 脚本/文件操作一律禁止（见下一节）。

## ⏰ 定时任务（首次安装自动创建，并在前置条件满足后立即执行首轮）

**专家包格式里没有定时任务**——这不是安装失败：

- 专家包格式（`plugin.json` 规范）里**没有** automation / schedule 字段；`validate_expert` 反而
  明确禁止 `hooks/`、`commands/`、`.lsp.json` —— 插件只承载 `agents/` + `skills/` + `avatars/` + `README`；
- 定时任务是**本机用户级资源**，存在 WorkBuddy 本机数据库里，**不属于任何包格式**，也不会随包迁移；
- 定时任务只能通过 WorkBuddy 自身的调度能力创建，**不能用脚本/shell 安装**
  （平台铁律：增删改定时任务必须走平台工具）。

**但安装时它是自动的**：新环境走完安装向导后，专家会**自己检查并创建**下面两条任务
（先用平台能力查重，已存在同名任务则跳过，只确认其为启用状态），**不需要用户额外提要求**。

### 前置条件（三项全部满足才创建 + 立即执行）

| # | 条件 | 判断方式 |
|---|---|---|
| 1 | 爬虫程序就位且个性化配置完成 | `jrzb.py config` → `crawler_exe_exists=true`；`jrzb.py setup` → `needs_setup=false` |
| 2 | **《投标策略》已提供** | `ztb.py status` → `strategy_gate.blocked=false` |
| 3 | 写入路径可写（网络盘场景） | `ztb.py check-io` → `alert=false` |

任一不满足 → **不创建**，只把「还差什么、怎么补」告诉用户；补齐后立即创建并立即跑首轮。

> **推送群不在这三项里**：没配推送群照样创建定时任务、照样跑（只是不发推送）。
> 推送链路的自检是 `ztb.py notify --text x --dry-run`，**只在汇报里提示，不参与创建判断**。

### 创建后立即执行首轮（不等下个整点）

1. **采集轮**：`jrzb.py start`（后台）→ 提示用户在 Chrome 登录 → 一次 `wait-login --timeout 600`；
2. 采集轮启动成功后**再跑分析轮**：按任务 2 的规格串行处理一批（上限 25 条）；
3. 汇总两轮结果给用户（处理了哪些 id、结论、推送了哪些、进度简报是否发出）。

> 首轮若卡在等待登录（用户不在），**只报告不重试**；任务已建好，下个整点自然重来。

### 两条任务的规格（专家创建时照抄，勿自创）

> prompt 是**可移植版本**：不写死任何本机绝对路径，一律通过 `ztb.py config` / `jrzb.py config`
> 解析实际路径，因此换机可直接照搬。工作目录设为该机器的项目目录（`work_root`）。

#### 任务 1：招标爬虫-每小时启动（每小时一次）

prompt：

```text
启动「招标数据爬虫」（技能 jrzb-crawler）并监控异常。每次执行都照此进行：

1. `jrzb.py status` 查状态。
2. 若为 Running（处理中）→ 本轮直接跳过、不重复启动，只记录一句「已在处理中，跳过」。
3. 若不是 Running → 以**后台任务方式**运行 `jrzb.py start`（该命令前台阻塞到本轮采集结束，不要在前台等）。
   - 若返回**退出码 5**：说明未提供《投标策略》，采集被闸门拒绝 —— 把提示转达用户并结束本轮，
     **不要**加 `--allow-without-strategy`；
   - 启动后再跑一次 `status` 复核为 Running。
4. 用一次 `jrzb.py login-state` 确认登录态（**不要循环轮询**）。若显示需要手动登录，
   用 `jrzb.py wait-login --timeout 120` 等一次（无人值守短超时；超时脚本会自动停掉爬虫），
   并在汇报里注明「等待手动登录未完成，已放弃本轮」。
5. `jrzb.py check-download --json --send`：alert=true → 直接发到推送群（未配置则不发送，不影响本轮）
   （标题已含「【爬虫告警】」）；alert=false → 静默。脚本自身有冷却去重，不要重复发送。
6. `jrzb.py check-exit --json`：alert=true → 同法发送（正文含异常栈）；alert=false → 静默。
   企业微信连接器未启用时跳过发送，不要因此中断本轮。
7. 用一句话汇报本轮结果（已启动 / 已在处理中已跳过 / 被策略闸门拒绝，+ 是否发了告警）。

注意：无论启动还是跳过，都保持每小时触发一次的固定频率，不因「已在运行中」而暂停或延长间隔。
（脚本路径以 `jrzb.py config` 解析出的实际路径为准。）
```

#### 任务 2：招投标项目分析-每小时兜底（单轮上限 25 条）

prompt：

```text
执行「招投标项目投标策略分析」（技能 ztb-analyzer），采用**单轮串行批处理**：
一轮内连续处理多条，逐条独立完成 next → bundle → OCR → 初筛 → save → push。
绝不并行、绝不绕过锁。单轮上限 25 条。

0. 先跑一次 `ztb.py check-io` 体检写入路径（项目目录/数据库可能在网络映射盘）。
   alert=true → 推告警并结束本轮，不要继续白跑。
1. `ztb.py next`：
   - 退出码 5（error=strategy_required）→ 未提供《投标策略》，**立即结束本轮**并把配置命令转达用户；
     **不要**加 `--ignore-strategy`；
   - pending:0 → 结束本轮；
   - locked_by（退出码 4）→ 已有另一轮在进行，**立即结束本轮**，不等待、不重试；
   - 正常返回 → 记下 id。
2. `ztb.py bundle --id <id>`，用 Read 工具读素材包全文。
   - **过期快速通道**：`expired: true`（投标截止或报名截止已过）→ 返回的是精简包，
     不要读附件、不要 OCR，直接判「不参加（截止时间已过）」后 save 并继续下一条。
3. 未过期时：对 `images_to_ocr` 里每张图用 Read 工具识图并完整提取文字
   （正文为图片时，项目编号/报价起止时间/采购需求/商务要求不可跳过）。
4. 读当期《投标策略》（`ztb.py strategy --print`，或 `next` 输出里 `strategy.strategy_file` 指向的文件），
   按四维模型（业务对口 / 关系单位 / 资质门槛 / 采购方式与评分）给出
   「推荐参加 / 考虑 / 不参加 / 关注（后续参加）」，并引用策略原文条目。
5. 按 SKILL.md 规定的 summary.json 字段结构写结论，存到 `ztb.py config` 输出的 `state_dir` 下，
   文件名 `_summary_<id>.json`。
   - 跨站重复采集的同一项目 → 判「不参加」，否决规则写「重复采集：与 ID xx 为同一项目」，避免二次推送。
6. `ztb.py save --id <id> --summary "<路径>"` 落库；JSON 非法则修正重试；放弃该条则 `ztb.py abort` 释放锁。
   - save 返回 `alert: true`：`partial: true` → 推告警后继续下一条；`ok: false`（数据库写失败）→ 推告警并结束本轮。
7. `ztb.py push --id <id> --send`：达标自动发到推送群，不达标自动跳过（看返回的 `send.ok`）。
8. 累计本轮的 id / 项目简称 / 结论 / 是否推送。

停止条件（满足任一即结束本轮）：① pending:0；② 撞锁（退出码 4）；③ 已满 25 条；④ 策略闸门（退出码 5）。

9. **批次结束后用 `ztb.py notify --text-file <文件>` 发「进度简报」到同一推送群**（撞锁空转时不推，避免重复打扰）：
   先 `ztb.py status` 取 done / pending / total，然后按 markdown 发送：
   📊 招投标分析进度 · HH:MM ／ 本批 N 条（ID a–b）／ ⭐ 推荐参加：… ／ 👀 关注：… ／
   其余 M 条不参加 ／ 累计 done ／ 待办 pending ／ 库内 total
   （⭐ 与 👀 按实际结论出现，无则省略；多条时 ID 用「、」分隔）

发送目标 = `ztb-analyzer config` 的 `push_target.group_name`，由脚本按群名现场解析会话；
**未配置 / 群不可达 / 未授权时跳过并在汇报里如实注明**，不得因此中断分析（详见 SKILL.md「推送目标怎么定」）。

铁则：严格串行、不并行；每条都经过锁（next 加锁 / save 或 abort 释放）；
结论必须基于素材包与 OCR 实际内容，缺信息填「未公开」，禁止编造。
（脚本路径以 `ztb.py config` 解析出的实际路径为准。）
```

> ⚠️ **定时任务是有状态的本机资源**：换机器、换工作目录、改过 `work_root` 之后，
> 都要回头确认这两条任务是否仍指向正确的项目目录，必要时重建。
> 调度器**最小触发粒度是「小时」**（`FREQ=MINUTELY` 不被支持），所以做不到分钟级；
> 需要"有数据就处理"的实时性时，用**长轮询 worker**（见两个技能的 SKILL.md），
> 它和每小时兜底共用同一把锁，撞锁即安全跳过。

### 路径配置说明

| 配置键 | 含义 | 默认值 |
|---|---|---|
| `work_root` | 项目根目录：内含爬虫程序与 `buzdata.db` | `<工作区>/jrzb_spider`（工作区不可识别时退回 `~/jrzb_spider`） |
| `state_root` | 状态根目录：锁/素材包/日志/告警状态 | `<工作区>/.workbuddy`（同上退回 `~/.workbuddy`） |
| `db` | 直接指定 `buzdata.db`（优先于 `work_root`） | `<work_root>/buzdata.db` |
| `crawler_exe` | 直接指定爬虫程序路径 | `<work_root>/xCD.WB.Skill.JRZB.exe` |
| `state_dir` | 本分析技能的状态目录 | `<state_root>/ztb-analyzer` |
| `crawler_run_dir` | 爬虫运行日志/状态目录 | `<state_root>/jrzb-crawler` |
| `site_packages` | 额外 site-packages（PDF 解析用） | 当前解释器自带 |
| `wecom_push_group` | **企业微信推送目标群名称**（推荐；命中提醒/进度简报/告警都发这里） | 未设置 |
| `wecom_chat_id` | 会话标识（**仅作记录，不能用于发送**：平台要求每次现场解析） | 未设置 |

优先级：**命令行参数 > 环境变量 > 配置文件 > 内置默认**。
可用环境变量：`ZB_CONFIG`、`ZB_WORK_ROOT`、`ZB_STATE_ROOT`、`ZTB_DB`、`ZTB_STATE`、`JRZB_EXE`、`JRZB_DB`、`JRZB_RUN_DIR` 等。

**共享配置文件本身的位置**：`ZB_CONFIG` → **`<工作区>/.workbuddy/zb-tender.json`（存在才用）**
→ `~/.workbuddy/zb-tender.json`。做一次工作区自包含安装后，配置就落在工作区里，技能自动认它。

### 爬虫个性化配置（`xCD.WB.Skill.JRZB.dll.config`）

与 exe 同目录，由 `jrzb.py setup` 维护。**这些键每台机器都不同，安装时必须向使用者确认**：

| 分组 | 键 | 说明 | 必填 |
|---|---|---|---|
| 浏览器 | `ChromePath` | 本机 chrome.exe 完整路径 | 否（有默认） |
| 浏览器 | `UserDataDir` | 独立 Chrome 配置目录（**登录态存这里**） | 否 |
| 浏览器 | `DebuggingPort` / `Playwright_CDP_Endpoint` | CDP 端口与连接地址，两者端口需一致 | 否 |
| 采集 | `工作目录` | 爬虫写入的根目录，通常等于 `work_root` | 是 |
| 采集 | `分页页面爬取数` | 每关键词翻页数（越大越慢越全） | 否 |
| 登录 | `LoginTimeoutSeconds` | **手动登录等待上限（秒）**，超时判失败并停止抓取 | 否（建议 600） |
| 登录 | `LoginUrl` / `SkipLogin` | 登录门户与门禁，一般用默认 | 否 |
| 企业版 | `企业版`、`企业版_corpid`、`企业版_aiapi`、`企业版_aiapi_xkey`、`企业版_拷贝到AppData的路径`、`企业版_附件的读取路径` | 仅启用企业版（BPM）时需要 | 否 |

> **向导不再询问「登录账号 / 密码」**：爬虫程序不读取这两个键（已在 exe 内验证为 0 引用）。
> 旧配置里若还留着账号密码，执行 `jrzb-crawler setup --manual-login` 一键清空，避免密码明文留在文件里。

`jrzb.py setup` 会把「仍是打包作者示例值 / 未填写」的必填项标出来，写回时自动备份原文件。

### 登录方式（手动登录，推荐）

爬虫打开登录门户后，**等待用户在打开的 Chrome 窗口里手动输入账号密码**，用页面文案判断登录态，
超过 `LoginTimeoutSeconds` 判失败并停止抓取。登录态保存在 `UserDataDir`，**密码不落盘到配置文件**。

```bash
python skills/jrzb-crawler/scripts/jrzb.py setup --manual-login [--timeout 600]  # 一键切到手动登录模式
python skills/jrzb-crawler/scripts/jrzb.py login-state                          # 单次探测登录状态
python skills/jrzb-crawler/scripts/jrzb.py wait-login --timeout 600             # 一次性等到登录完成
```

> ⚠️ **等待登录不要循环轮询**：反复调用 `status` 会持续消耗额度。正确姿势是**一次** `wait-login`
> （服务端等待，不产生额外消耗）；**自身超时会自动停掉爬虫**（可用 `--no-stop-on-timeout` 关闭），
> 避免空转。它只会认本次调用前 5 秒内更新过的日志，不会把上一轮的旧日志误判成本轮已登录。
> 无人值守的定时任务里，`wait-login` 的 `--timeout` 建议 ≤120 秒，本轮等不到就放弃、下轮再试。

### 投标策略配置（`strategy.json`）

由 `ztb.py strategy` 向导维护，**每家客户都不同，安装时必须按客户情况配置**。
按四维初筛模型组织：

| 维度 | 字段 | 说明 | 必填 |
|---|---|---|---|
| 业务对口 | `业务范围` | 按板块列出对口项目类型（判断「业务是否对口」的依据） | 是 |
| 业务对口 | `优势` / `劣势` | 可转为加分项的能力 / 命中即倾向不参加的能力 | 否 |
| 关系单位 | `关系单位` | 有关系、可优先参加的单位名单（含下属/分支） | 是 |
| 关系单位 | `关系单位豁免项` | 对关系单位可放宽的否决规则 | 否 |
| 资质门槛 | `硬性否决规则` | 命中任一即判「不参加」 | 是 |
| 资质门槛 | `门槛底线` | 可接受的最低资质要求 | 否 |
| 采购方式与评分 | `价格分规则` | 价格分可接受下限 / 不考虑线 | 是 |
| 采购方式与评分 | `期望评标结构` | 期望的商务分/价格分/主观分比例 | 否 |
| 采购方式与评分 | `偏好规则` | 出现即倾向参加的特征 | 否 |
| 基本信息 | `公司名称` / `备注` | 抬头与补充背景 | 否 |

配置好后由 `ztb.py strategy --render` 渲染成《投标策略》markdown，初筛时读它。
包内 `skills/ztb-analyzer/references/strategy.template.json` 是空白模板可参考格式。

### 企业微信推送目标（群聊，一次推送多人可见）｜**非必填**

> ⚠️ **这是可选配置，不阻塞任何流程**：没配推送群、群不可达、企业微信未授权时，
> **采集与初筛照常运行**，只是不发推送。它不是策略闸门的一部分，也**不是**创建定时任务的前置条件。

**命中提醒、进度简报、采集/写入故障告警全部发到同一个群** —— 把要收到通知的人拉进这个群、
再把机器人也放进群即可，**一次推送群里所有人都能看到**，不需要逐个私聊。

```bash
python scripts/ztb.py config                              # 看 push_target（是否已配置 + 发送方法）
python scripts/ztb.py config --push-group "招投标情报群"   # 保存目标群名称（可选）
python scripts/ztb.py config --clear-push-group           # 清空
```

**包内已内置发送命令**，不需要手工拼接口调用：

```bash
python scripts/ztb.py push --id <id> --send        # 结论达标 → 直接发到推送群（不达标自动跳过）
python scripts/ztb.py notify --text "<简报/告警>"   # 发任意 markdown（进度简报、故障告警）
python scripts/ztb.py notify --text-file brief.md  # 长文本用文件
python scripts/ztb.py notify --text "x" --dry-run  # 只测推送通道是否可达，不发送
```

爬虫侧同理：`jrzb.py check-download --send` / `jrzb.py check-exit --send`（告警即发），
以及通用的 `jrzb.py notify --text "..."`。

**为什么存群名而不是会话 ID**：平台规定每次发送前必须重新拉一次会话列表来确定目标，
缓存的会话 ID 不可复用。所以每次发送前脚本会执行
`wecom-cli message aibot sessions list` → 按 `chat_name` 精确匹配群名 → 取该会话发送。

| 约束 | 说明 |
|---|---|
| 可发送范围 | ① 授权人本人；② 机器人**最近有消息往来**的单聊/群聊（列表最多 20 个） |
| 群前提 | 群里必须有**机器人**和**要通知的人**，且机器人近期在群里发过/收过消息 |
| 不能用什么 | 通讯录里的同事 ID、用户口述的 ID、历史保存的会话 ID —— 都不能直接作为发送目标 |
| 解析不到 | 专家会如实告诉你「目标群不在最近会话中」，提示先在群里 @ 一下机器人，不会静默失败 |
| 授权 | 需 `wecom-cli auth show --status` 为 `authorized`；否则推送与告警都发不出去 |

> 想按人分流（比如只让某个人收到某类项目）目前**做不到** —— 平台只允许发到「授权人本人」或
> 「机器人近期有往来的会话」。要分流只能建多个群、分别跑一套实例。

## 写入健康（结果落在网络盘时怎么办）

项目目录（`openclaw_file_path`）与 `buzdata.db` **可能位于网络映射盘或 UNC 共享**上。这类介质在掉线、
休眠、盘符未映射、权限变化、空间不足时会写不进去，甚至「写成功但数据没真正落盘」。因此：

- **写后校验**：关键写入都是「写 → flush+fsync → 读回比对」，不一致即判失败，不会静默丢数据；
- **失败不崩溃、不卡锁**：写入失败会返回 `alert: true` 的结构化 JSON，并保证释放锁（不会把后续分析卡死 20 分钟）；
  - **数据库**写失败 → 整体判失败（`pre_add` 保持 1），磁盘恢复后自动重试；
  - **项目目录** `summary.json` 写失败 → 结论仍入库（按完成处理），失败项记入待补写清单；
- **主动体检 / 补写**：

  ```bash
  python skills/ztb-analyzer/scripts/ztb.py check-io    # 体检各关键路径的真实可写性 / 盘类型 / 耗时
  python skills/ztb-analyzer/scripts/ztb.py resync      # 磁盘恢复后补写欠账（--id N 或 --all）
  ```

- `status` / `config` 输出里都有 `io` 块（`*_drive`、`has_network_path`、`pending_resync`、`last_io_alert`）。
- 「晶晶」会在每批分析前跑 `check-io`，并把 `alert: true` 的写入故障通过企业微信告警给你
  （同一故障 30 分钟内不重复打扰）。**若需收到这类告警，必须启用「企业微信」连接器。**

## 头像

头像位于 `avatars/expert.png`（512×512 PNG，≤500KB）。如需替换：PNG/JPG，512×512 px，单张 ≤ 500KB。

> 2.0.0 版本曾短暂改为专家团、额外生成了 3 张成员头像，已一并归档到
> `专家包导出\_archive\team-2.0.0\avatars\`，未删除，需要时可取回。

## 打包分享（维护者）

```bash
# 0) 运行包有更新时：重出爬虫包并上传服务器，同时更新 crawler/runtime-source.json
#    （version / size / sha256 三个字段；url 一般不用改）
python <本目录外的>pack_crawler_runtime.py            # 产出 jrzb-runtime-<版本>.zip
python <expert-manager>/scripts/validate_expert.py <专家目录>   # 校验
python <expert-manager>/scripts/register_expert.py <专家目录>   # 注册
python <expert-manager>/scripts/package_expert.py <专家目录> <输出目录>   # 打包为 .zip
```

> **为什么运行包不放进专家包？**
> 官方开放平台对专家包有 **20MB 上传上限**，而运行包（.NET 运行时 + Playwright 驱动）
> 压缩后仍有约 70MB。因此拆成两个产物：
> 1. **专家包**（<1MB）：技能、agent、头像、`crawler/runtime-source.json` —— 提交平台审核；
> 2. **爬虫包**（`jrzb-runtime-<版本>.zip`，约 70MB）：上传到自有服务器，把下载地址填进
>    `crawler/runtime-source.json` 的 `url`。
>
> 打包时还会**强制剔除** `buzdata.db`（业务数据）与 `xCD.WB.Skill.JRZB.dll.config`
> （本机账号密码），保证分发包不泄露隐私。运行包整体仍打成一个 zip，
> 是为了兼容官方 `package_expert.py` **跳过隐藏目录**的规则（Playwright 驱动位于
> 隐藏目录 `.playwright/`，散文件会被漏掉）——离线自备时也照这个结构放。

#### GitHub 发布（完整包）

GitHub 上的「完整包」= 轻量专家包 + 运行包内嵌在 `crawler/jrzb-runtime.zip`，
用户下载解压即用、**全程无需联网**（脚本按《运行包的获取方式》优先级 2 命中包内运行包）。
构建命令与发布步骤见 `dist/GitHub发布指引.md`。

> **GitHub 的体积红线**（决定了 70MB 的运行包只能挂 Release，不能进仓库）：
> 网页上传单文件 **25 MiB**；`git push` 单文件 >50 MiB 告警、**>100 MiB 直接阻断**；
> Release 附件单文件上限 **2 GiB**（不限总量与带宽）；仓库建议 <1 GB。
> 完整包约 70MB → git 推送可行（< 100 MiB，会告警一次），但**推荐挂 Release 附件**，
> 仓库本体只放源码树（< 1MB），clone 才快。

### 包结构要点（改动时别踩）

| 项 | 约束 |
|---|---|
| `expertType` | `"agent"`（单个 AI 专家）；多角色协作才是 `"team"` |
| `name` / `plugin` / 目录名 | 三处必须一致，当前为 `lida-biaoxun-bidding-analyst`。小写字母+连字符，平台按 `name` 做**同类型资产唯一性校验** |
| `agentName` | `bidding-strategist`，必须有同名 `agents/bidding-strategist.md`。**与 `name` 无关**，不必跟着改 |
| `agents/*.md` | 每个 MD 的 frontmatter `name` 必须等于文件名（去掉 `.md`）；**不得**出现 `tools:` 字段 |
| `tags` / `quickPrompts` | 各**必须恰好 3 个**；`defaultInitPrompt` 须与 `quickPrompts[0]` 一致 |
| `displayDescription.zh` | 建议 40–50 字（超出只告警） |

> 专家包目录名与 `plugin.json` 的 `name` / `plugin` 是同一套**唯一标识**，三处必须一致。
> **改名会被视为新专家**，原专家在专家中心会丢失，所以只在被平台驳回这类不得不改的场景才动它
> （2.9.0 就因 `bidding-strategist` 被平台判定占用，改为 `lida-biaoxun-bidding-analyst`）。
> `agentName` 不在此列——它只要求对应 `agents/<agentName>.md`，与 `name` 解耦。

> 历史记录：2.0.0 曾改造为 Team 型专家团（主理人 + 2 名团员），因调度开销偏大，
> 2.1.0 已回退为单专家并把策略梳理/深度分析并回同一角色。团队版的 agent MD 与头像
> 归档在 `专家包导出\_archive\team-2.0.0\`，可追溯。
