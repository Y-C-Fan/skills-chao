---
name: youtube-obsidian
description: "把 YouTube 课程/讲座转成 Obsidian 阅读笔记：meta+封面+字幕去重+均分截图3张+分节总结。用户丢一个 YouTube URL 并要求存 Vault、做课程笔记时使用。不再生成 Framework & Mindset 段。"
---

# YouTube → Obsidian v1（简化版：封面 + 均分截图 + 阅读版 Prompt）

> 状态：2026-09-10 已去 Framework & Mindset。正文只保留分节总结 + Screenshots + Transcript + 不确定性备忘。不重复造轮子：转写复用 `baoyu-youtube-transcript`，截图链路改编自 `tga-cheetung/yt-digest`（MIT）。

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
 → 2. transcript（baoyu skill，优先 zh>en；被 bot 拦则走 cookies，见下）
 → 3. 阅读版总结（课程类忠实转述，见下正文规范）
 → 4. 截图（课程类：内容小节数 N，每节 1 张 + cover；图放节上，不堆底部）
 → 5. 落盘 Clippings/Courses/ 或 Clippings/Podcasts/（YAML 全字段 + 一md一文件夹装图）
```

## 正文规范（2026-09-10 起：去 Framework；课程少 AI）

- 正文结构只保留：Overview + 分节总结（一节500字左右）+ Transcript + 转录不确定性备忘。**不要 `## Screenshots` 底部画廊，图全部分到各节上边。**
- 禁止生成 `## Framework & Mindset` 一/二/三。理由：冗余，信息在分节里已覆盖。
- **课程类少用 AI 发挥：** 忠实转述，保留课程知识（定义原话、数字、论文/模型名、人名、课堂案例）；只整理冗余（字幕滚动重复、口头禅、同义反复），不造新框架、不引申、不下新结论。宁可长一点贴课程，也不要短而滑。
- 一集一文件夹，md 和图同居：`Clippings/Courses/<SERIES>/<basename>/<basename>.md` + 同文件夹内 `cover.jpg` + `sec01.jpg…secNN.jpg`（关键核验过的幻灯片可另存 `slide-<slug>.jpg`）。正文用同目录短引用 `![[secNN.jpg]]`，YAML `cover:` 只写 `"cover.jpg"`。不许把 md 扔在系列根下、图另放别处，也不许把全系列的图堆在系列目录根下。
- 每节图放节上：`## 一、…` 上方先放 `![[sec0N.jpg]]` + 一行 `*↑ 约 M:SS 课程画面*`，再写小节正文。截图时间戳按时长 5%–95% 均分 N 张（N=内容小节数，Overview 配 cover 不占图）。

## 首验状态（2026-09-09）

- ✅ `yt-dlp` + `ffmpeg` + `bun` 就绪（按各机 `ENV-GUIDE` 路径自行配置，进 PATH）
- ✅ 两条视频 meta + cover 已落盘（Vault 内 demo 笔记验证）
- ✅ cookies + EJS 参数打通（2026-09-09 实测）：`--cookies-from-browser firefox:<profile> --js-runtimes node --remote-components ejs:github`（缺这俩会报 `n challenge solving failed / The page needs to be reloaded`）
- ✅ 两篇首验笔记已回填完（transcript + 均分 3 张 + 阅读版正文）

## 登录态：专用 Firefox 钥匙扣（2026-09-09 已打通，免手动导 Cookie）

- 原理：Chrome/Edge 新版用应用级加密，关着也读不出；专用 Firefox 档案只要平时不打开，yt-dlp 可免提权直读。**该档案只做钥匙扣，别拿来上网。**
- 过期（一般几个月）后重登一次 YouTube，登完彻底关闭。
- `cookies.txt` 留本地作备用（= 导出时的登录态快照，勿外传，勿进 git）。
- 试过但不行的：直连 InnerTube / tv client / python transcript-api（IP 被限）、PO Token 脚本（token 能发但 YouTube 仍要登录）。
- 手动字幕优先：`--write-subs --sub-langs 'zh-Hans'`；自动字幕：`--write-auto-subs --sub-langs 'en-orig,en'`
- VTT 滚动重复严重：英文用 overlap-merge去重，中文直接按 `。！？；` 换行即可

拿到新 URL 后跑（路径按自己机器改）：

```powershell
$Y='<yt-dlp>'; $C='--cookies-from-browser','firefox:<profile>'; $X='--js-runtimes','node','--remote-components','ejs:github'
& $Y @C @X --write-subs --write-auto-subs --sub-langs 'zh-Hans,zh,en-orig,en' --sub-format vtt --skip-download --write-info-json -o '<transcripts>\<id>\%(id)s.%(ext)s' '<URL>'
# 截图：360p 小片源 + 25%/50%/75% 均分 3 张
& $Y @C @X -f 'bv*[height<=360]+ba/b[height<=360]/b' --merge-output-format mp4 -o '<transcripts>\<id>\video.%(ext)s' '<URL>'
ffmpeg -y -loglevel error -ss <sec> -i video.mp4 -frames:v 1 -q:v 3 'assets-<id>-frame<n>.jpg'
```

## 落盘规范（一集一文件夹，md 和图同居）

- Course → `Clippings/Courses/<SERIES>/<basename>/<basename>.md`，图（`cover.jpg`、`sec01.jpg…`、`slide-<slug>.jpg`）和 md 放同一文件夹，正文同目录短引用 `![[secNN.jpg]]`
- 系列课程 → `Clippings/Courses/<SERIES>/` 下每集一个文件夹 + 系列根下一篇 `00-INDEX-<SERIES>.md` 做导航（索引用 `[[<basename>]]` 即可，文件名全局唯一）
- Podcast/讲座 → `Clippings/Podcasts/<basename>/<basename>.md` 同标准（截图可少于课程类，按嘉宾/话题切）
- YAML：title/author/channel/url/video_id/fetched/source/language/caption_type/duration/tags/cover（cover 只写 `"cover.jpg"`，同目录）

## v2 升级点

1. Prompt 加 `⏱️ [HH:MM:SS → HH:MM:SS]` 时间戳 grounding，截图从“均分”升级到“按小节中点截”
2. channel 监听（RSS + 定时）→ 半自动
