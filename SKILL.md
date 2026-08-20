---
name: super-upload
description: 当 agent 需要用 `sau` CLI 向抖音、B 站、小红书、快手、视频号、百家号、TikTok 或 YouTube 上传视频时使用，也覆盖即梦（Jimeng）批量视频生成的上游生产工作流。视频号为优先验证路径，涵盖跨平台安装、微信扫码、草稿优先、持久浏览器窗口和批量任务。
---

# SuperUpload

先使用仓库的 `sau` CLI；不要先读 uploader 源码，也不要先用浏览器桥接。浏览器桥接只用于 CLI 已经打开真实页面、但某个控件变化需要人工辅助的情况。

## 安全执行原则

1. 先安装并运行 `sau --help`，再开始登录或上传。
2. 账号 cookie、`conf.py`、二维码和视频素材都是本地敏感数据：不得提交、复制到日志或发送给第三方。
3. 第一次或批量任务先保存草稿；只有用户明确授权才公开发布或定时发布。
4. 视频号首次扫码写入账号专属持久化 Chrome Profile；单个任务可用 `--keep-open`，批量命令默认复用同一 Profile 和可见窗口。不要为每条视频反复创建、关闭或切换浏览器。
5. “保存草稿”只点击一次；等待 10 秒后进入草稿箱按主标题核验，成功后随机等待 10–20 秒再处理下一条。禁止轮询式重复点击。
6. 所有批量上传必须接入 SuperMedia 统一账本：上传前按成品、源作品和目标账号
   查重并预约；完成或中断后回写状态。没有唯一 `source_key` 的文件进入 HOLD。
7. 「位置」默认不显示地址：上传页「短标题」后的「位置」字段保持为空，不附加任何
   地理位置；只有用户明确要求时才填写。
8. 上传前先做 VFR 检测修复（封面生成依赖恒定帧率）；视频标注选择「含AI生成内容」；
   不自动选择合集；标签不含比例描述；定时发表只能设未来 10 天，长排班按窗口分批。
9. 自我改进只能追加隔离的学习候选，不得在上传任务中改写本 Skill、脚本、模板、
   选择器、默认节奏或安全门禁。运行状态与账号经验不得泛化为全局规则。

## 最短路径：视频号

```bash
cd <repo-root>
uv run sau tencent login --account main --headed
uv run sau tencent check --account main
uv run sau tencent upload-video --account main --file "/absolute/path/video.mp4" --title "标题" --desc "文案" --tags "标签1,标签2" --draft --headed --keep-open
```

同一账号的多条视频使用 JSON 清单：

```bash
uv run sau tencent upload-video-batch --account main --manifest batch.json
```

`upload-video-batch` 默认可见、默认保留窗口、默认将每项保存草稿；关闭窗口才结束会话。相对素材路径以 `batch.json` 所在目录为准。无人值守时才显式使用 `--headless --close-when-done`。

普通 Chrome 的已登录标签不能替代 `sau` 的持久 Profile：浏览器桥接可辅助检查页面，但通常不能向视频号子页面注入本地文件。需要自动上传时，首次运行 `sau tencent login --account <name> --headed` 扫码一次，后续复用该账号 Profile。

## 模板 A：目录一键草稿

用户要求“账号 A、指定目录、批量草稿、保留窗口”时，读取 `references/template-a.md` 并运行 `scripts/template_a.py`。脚本先检查账号专属持久 Profile、生成可审阅清单，再以一个可见窗口执行；每次单击保存后等待 10 秒进入草稿箱按标题核验，默认相邻已确认草稿间隔为 10–20 秒，并在整批确认后回到平台主页。不要添加定时刷新、点击其他区域或其他模拟在线行为。

模板 A 默认从素材路径找到 `Video_Download`，调用 `superdown88` 的
`media_asset_catalog.py` 完成 preflight、reservation 和结果回写。相同成品发往
同一账号始终阻止；同源不同 Remix 需要用户明确允许。直接使用 `sau` 时也必须
按 `references/media-lineage.md` 手工执行同样门禁。

## 定时发表计划模板

用户指定某视频号账号每天发表数量并授权定时发表时，按
`references/schedule-templates.md` 模板一排期：固定节点上午 9:00、中午 12:00、
下午 3:00、晚上 8:00，按序取前 N 个；N 超过 4 时停止并要求人工调整。定时发表属
公开发布，必须先取得用户明确授权；位置默认不显示地址。

## 即梦（Jimeng）批量视频生成（上游生产）

即梦是视频生产环节（上传前的上游）；`sau` 不支持即梦，用 Kimi WebBridge 控制已登录
即梦的 Chrome，运行 `scripts/jimeng_video_batch.py`（plan/prepare/generate/poll/
run-one/run-pipeline/run-batch-conv）。生成视频回写来源文件夹并更新 `_ACCOUNT_BOOK.csv`，
发布再走本 Skill 上传流程。详细目录约定、命令与避坑见 `references/jimeng-video-batch.md`。

即梦环节铁律：批量前先出积分计划并取得授权才自动点「生成」；提交后校验数字 workspace
与积分扣减；下载前 MD5 查重；图片用 base64 注入、不点击上传槽；同会话批量按唯一提示词
标记定位轮次；断点状态文件防重复提交；失败条目隔离并保留错误文件。

## 选择工作流

| 目标 | 优先命令 |
| --- | --- |
| 新账号或视频号 Profile 失效 | `sau <platform> login --account <name> --headed` |
| 检查登录状态 | `sau <platform> check --account <name>` |
| 单条视频 | `sau <platform> upload-video ...` |
| 视频号多条视频 | `sau tencent upload-video-batch --manifest <json>` |
| 页面控件变更后的辅助操作 | 保留 CLI 已打开的浏览器，再用可控制真实浏览器的桥接工具 |

二维码出现时，能展示本地图片的 agent 必须直接展示二维码，不要只返回文件路径。扫码、短信和二次验证必须由账号所有者完成。

## 平台与 agent 参考

- 安装、浏览器依赖和操作系统差异：`references/install.md`
- 视频号登录、草稿、批量与避坑：`references/wechat-channels.md`
- Codex、Claude Code、OpenClaw、Kimi WebBridge 等调用约定：`references/agents.md`
- 驱动、二维码 iframe、cookie 与重复提交故障：`references/troubleshooting.md`
- 模板 A 的一键调用与逐条文案 JSON：`references/template-a.md`
- 定时发表计划模板（模板一：每日 N 条、四节点 9/12/15/20）：`references/schedule-templates.md`
- 三个 Skill 的资产血缘、查重、预约和回写：`references/media-lineage.md`
- 学习候选、人工审批、核心保护和回滚：`references/controlled-evolution.md`
- 即梦批量视频生成与避坑：`references/jimeng-video-batch.md`

修改任何核心文件前必须另开维护任务，预先提交人工批准记录，并运行
`python3 scripts/controlled_evolution_guard.py --approval-file ...`。不得伪造批准，也不得
因某条经验重复出现而自动晋升。
