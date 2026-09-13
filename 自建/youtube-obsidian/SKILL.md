---
name: youtube-obsidian
description: "把 YouTube 课程/讲座/播客转成 Obsidian 阅读笔记：meta+封面+字幕去重+分节总结一次展开（课程类加均分截图，播客类仅封面）。用户丢一个 YouTube URL 并要求存 Vault、做课程笔记时使用。不再生成 Framework & Mindset 段。"
---

# YouTube → Obsidian v1.3（阅读版：封面 + 分节总结一次展开 + 中文优先；课程类加均分截图，播客类仅封面）

> 状态：2026-09-10 已去 Framework & Mindset。正文只保留 Overview + 分节总结 + Transcript + 不确定性备忘。不重复造轮子：转写复用 `baoyu-youtube-transcript`，截图链路改编自 `tga-cheetung/yt-digest`（MIT）。
> v1.1（2026-09-13，三条实测决策）：① 转写第一次就带 Firefox 钥匙扣 cookies，不再裸跑；② 播客/访谈/讲座类只要 `cover.jpg`，不做均分截图；③ 分节总结一次展开写全，不做"先短版再展开"两档。
> v1.2（2026-09-13，语言规范）：正文是写给人读的完整中文，见下 `## 语言规范`。
> v1.3（2026-09-13：原则代替白名单 + 数字规则）：白名单太僵化，改成可读性判断；数字和单位一律阿拉伯数字（起因：v1.2 重写把 1MHz 写成"一兆赫兹"，矫枉过正）。

## GitHub 去重结论（A 为主、C 为辅）

| 候选 | 结论 |
|---|---|
| `tga-cheetung/yt-digest`（Claude Skill：TL;DR + 时间戳 takeaways + 幻灯片截图 + transcript，`yt-dlp + ffmpeg + parse_vtt.py`，`vault/research/youtube/<slug>.md`） | ✅ 主参考。截图脚本和 vault 落点直接改编，本方案只把它的“takeaways”换成你的“阅读版 Prompt（分节500字+，不要 Framework & Mindset）”，截图从“关键时刻”简化为“均分3张” |
| `jdmonaco/ytcapture`（已并入 vidflow：按 interval 截帧 + 去重 + Obsidian wikilink） | 参考去重阈值（`dedup 0.85`），v2 升级用 |
| `mohsinkhadim59/youtube-obsidian-mcp`（MCP，需手动给 timestamps） | ❌ 不用（和“自动”相悖） |
| `JimmySadek/youtube-fetcher-to-markdown`（纯字幕归档，无截图） | 只参考 frontmatter 字段 |
| `davisbuilds/podsave`（AssemblyAI + OpenAI 付费链） | ❌ 太重，不用 |

## 管道（手动丢 URL 模式）

```
URL
 → 1. meta（oEmbed，无需登录）+ cover（i.ytimg.com 直链，无需登录）
 → 2. transcript（baoyu skill，优先 zh>en；第一次就带 Firefox 钥匙扣 cookies，见下——裸跑必被 bot 拦，白白多一轮）
 → 3. 阅读版总结（一次展开写全，不分短版/长版两档；课程类忠实转述，见下正文规范）
 → 4. 截图（只做课程类：内容小节数 N，每节 1 张 + cover，图放节上不堆底部；播客/访谈/讲座类跳过，只要 cover）
 → 5. 落盘 Clippings/Courses/ 或 Clippings/Podcasts/（YAML 全字段 + 一md一文件夹装图）
```

## 正文规范（2026-09-10 起：去 Framework；课程少 AI。2026-09-13 起：播客免截图；总结一次展开）

- 正文结构只保留：Overview + 分节总结 + Transcript + 转录不确定性备忘。**不要 `## Screenshots` 底部画廊；课程类图全部分到各节上边，播客类无节图。**
- 禁止生成 `## Framework & Mindset` 一/二/三。理由：冗余，信息在分节里已覆盖。
- **分节总结一次展开写全：** 不做"先短版再展开"两档。课程类一节 500 字左右；播客/访谈类按话题切 6–10 节、一节 400–700 字，保留争论点和语气，不过度压缩。
- **课程类少用 AI 发挥：** 忠实转述，保留课程知识（定义原话、数字、论文/模型名、人名、课堂案例）；只整理冗余（字幕滚动重复、口头禅、同义反复），不造新框架、不引申、不下新结论。宁可长一点贴课程，也不要短而滑。
- 一集一文件夹，md 和图同居：`Clippings/Courses/<SERIES>/<basename>/<basename>.md` + 同文件夹内 `cover.jpg` + 课程类另有 `sec01.jpg…secNN.jpg`（关键核验过的幻灯片可另存 `slide-<slug>.jpg`）；播客类文件夹内只有 `<basename>.md` + `cover.jpg`。正文用同目录短引用 `![[secNN.jpg]]`，YAML `cover:` 只写 `"cover.jpg"`。不许把 md 扔在系列根下、图另放别处，也不许把全系列的图堆在系列目录根下。
- 课程类每节图放节上：`## 一、…` 上方先放 `![[sec0N.jpg]]` + 一行 `*↑ 约 M:SS 课程画面*`，再写小节正文。截图时间戳按时长 5%–95% 均分 N 张（N=内容小节数，Overview 配 cover 不占图）。**播客/访谈/讲座类免截图**：talking-head 帧阅读价值低，且长片源（5 小时级）分段下载可单片段卡 5 分钟以上，成本不合算；Overview 配 `cover.jpg` 即可，如下载失败在 Overview 下留一行截图说明，不阻塞正文落盘。

## 语言规范（2026-09-13 v1.3：中文优先，可读第一，模型自行判断）

- **正文是给人读的完整中文**：每句话有主谓宾、有上下文，段落读下来像人话。**禁止把句子压成短语**（不准为省字把"他不再逐个看，让智能体先筛一遍"写成"不再逐看，智能体先筛"这类文言文电报体）。宁可长，不可干。
- **中英文取舍只看一条：中文读者读着顺不顺**。不要逐词对照的僵化白名单，Agent 按中文技术写作惯例自行判断。倾向是：叙述用中文；圈内本来就说英文的术语留英文（Agent、Loop、Terminal、PR、API、harness、prompt、token……）；模型名与产品名留英文；人名地名留原文；图标级短引每节至多一两句，且必须配中文翻译。普通词译成中文，拿不准就翻。
- **数字规则（硬的）**：带单位的量、百分比、金额、版本号、时间、确切计数，一律阿拉伯数字——`1MHz`、`64K`、`472 行`、`100%`、`$550`、`Opus 4.5`、`45 秒`、`1000 个合并请求`。模糊量和习语用中文（几十万、上万、九成、十几分钟、九十五条论纲、八十年代）。**反例**：不准把 `1MHz` 写成"一兆赫兹"，不准把 `64K` 写成"六十四"（v1.2 重写事故，矫枉过正）。
- **术语首次出现中英对照**（如"提示词（prompt）""代码审查（review）"），后文直接用中文。引文是调料不是正文：英文原话一律配中文解释，正文叙述只用中文讲。
- 转录不确定性备忘里的存疑拼写名单为例外（本来就是为了备查才列英文）。

## 首验状态（2026-09-09）

- ✅ `yt-dlp` + `ffmpeg` + `bun` 就绪（按各机 `ENV-GUIDE` 路径自行配置，进 PATH）
- ✅ 两条视频 meta + cover 已落盘（Vault 内 demo 笔记验证）
- ✅ cookies + EJS 参数打通（2026-09-09 实测）：`--cookies-from-browser firefox:<profile> --js-runtimes node --remote-components ejs:github`（缺这俩会报 `n challenge solving failed / The page needs to be reloaded`）
- ✅ 两篇首验笔记已回填完（transcript + 均分 3 张 + 阅读版正文）

## 登录态：专用 Firefox 钥匙扣（2026-09-09 已打通，免手动导 Cookie；2026-09-13 起第一次跑就带，不再裸跑）

- 原理：Chrome/Edge 新版用应用级加密，关着也读不出；专用 Firefox 档案只要平时不打开，yt-dlp 可免提权直读。**该档案只做钥匙扣，别拿来上网。**
- 实测（2026-09-13）：直连 InnerTube（android/web/ios 三 client）必回 bot detected；yt-dlp 无 cookies 也必败。裸跑 baoyu 只会白白多一轮"JS 挑战 + 睡 5 秒 + 重试"。所以 baoyu 侧第一次跑就设 `YOUTUBE_TRANSCRIPT_COOKIES_FROM_BROWSER=firefox:<profile>`，yt-dlp 侧第一次跑就带全套 `@C @X` 参数。
- 过期（一般几个月）后重登一次 YouTube，登完彻底关闭。
- `cookies.txt` 留本地作备用（= 导出时的登录态快照，勿外传，勿进 git）。
- 试过但不行的：直连 InnerTube / tv client / python transcript-api（IP 被限）、PO Token 脚本（token 能发但 YouTube 仍要登录）。
- 手动字幕优先：`--write-subs --sub-langs 'zh-Hans'`；自动字幕：`--write-auto-subs --sub-langs 'en-orig,en'`
- VTT 滚动重复严重：英文用 overlap-merge去重，中文直接按 `。！？；` 换行即可

 拿到新 URL 后跑（路径按自己机器改；注意 `@C @X` 第一次就带）：

```powershell
$Y='<yt-dlp>'; $C='--cookies-from-browser','firefox:<profile>'; $X='--js-runtimes','node','--remote-components','ejs:github'
& $Y @C @X --write-subs --write-auto-subs --sub-langs 'zh-Hans,zh,en-orig,en' --sub-format vtt --skip-download --write-info-json -o '<transcripts>\<id>\%(id)s.%(ext)s' '<URL>'
# 截图只做课程类：360p 小片源 + 5%–95% 均分 N 张（N=内容小节数）。播客/访谈/讲座类跳过这步。
& $Y @C @X -f 'bv*[height<=360]+ba/b[height<=360]/b' --merge-output-format mp4 -o '<transcripts>\<id>\video.%(ext)s' '<URL>'
ffmpeg -y -loglevel error -ss <sec> -i video.mp4 -frames:v 1 -q:v 3 'assets-<id>-frame<n>.jpg'
```

## 落盘规范（一集一文件夹，md 和图同居）

- Course → `Clippings/Courses/<SERIES>/<basename>/<basename>.md`，图（`cover.jpg`、`sec01.jpg…`、`slide-<slug>.jpg`）和 md 放同一文件夹，正文同目录短引用 `![[secNN.jpg]]`
- 系列课程 → `Clippings/Courses/<SERIES>/` 下每集一个文件夹 + 系列根下一篇 `00-INDEX-<SERIES>.md` 做导航（索引用 `[[<basename>]]` 即可，文件名全局唯一）
- Podcast/讲座 → `Clippings/Podcasts/<basename>/<basename>.md` + 同文件夹 `cover.jpg`，**不做 `secNN.jpg` 均分截图**（talking-head 帧阅读价值低，长片源下载成本高；截图失败只在 Overview 下留一行说明，不阻塞落盘）
- YAML：title/author/channel/url/video_id/fetched/source/language/caption_type/duration/tags/cover（cover 只写 `"cover.jpg"`，同目录）

## v2 升级点

1. Prompt 加 `⏱️ [HH:MM:SS → HH:MM:SS]` 时间戳 grounding，课程类截图从"均分"升级到"按小节中点截"
2. channel 监听（RSS + 定时）→ 半自动

## 实测教训（2026-09-13：5h16m 播客 Lex #501 全链复盘，v1.1 决策依据）

- 裸跑 baoyu（无 cookies）：InnerTube 三 client 全 `bot detected`，yt-dlp fallback 无 cookies 也败 → 两轮纯浪费。结论：第一次就带钥匙扣。
- 360p `--download-sections` 20 秒片段：5 分钟未拖下（站侧限速 + JS 挑战 + 睡 5 秒）。talking-head 长片源截图投入产出比过低 → 播客类免截图。
- 31 万字符字幕必须分段读（约 7–8 万 token 输入），这部分是真工作省不掉；省时间的办法是别在"短版还是长版"上返工，一次展开写全。
