---
name: super-upload
description: 当 agent 需要用 `sau` CLI 向抖音、B 站、小红书、快手、视频号、百家号、TikTok 或 YouTube 上传视频时使用。视频号为优先验证路径，涵盖跨平台安装、微信扫码、草稿优先、持久浏览器窗口和批量任务。
---

# SuperUpload

先使用仓库的 `sau` CLI；不要先读 uploader 源码，也不要先用浏览器桥接。浏览器桥接只用于 CLI 已经打开真实页面、但某个控件变化需要人工辅助的情况。

## 安全执行原则

1. 先安装并运行 `sau --help`，再开始登录或上传。
2. 账号 cookie、`conf.py`、二维码和视频素材都是本地敏感数据：不得提交、复制到日志或发送给第三方。
3. 第一次或批量任务先保存草稿；只有用户明确授权才公开发布或定时发布。
4. 视频号单个任务可用 `--keep-open`；批量命令默认复用一个可见浏览器窗口，整批完成后保持打开。不要为每条视频反复创建和关闭浏览器。
5. 每次成功提交后检查页面确认状态或草稿列表，禁止轮询式重复点击“保存草稿”或“发表”。

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

## 选择工作流

| 目标 | 优先命令 |
| --- | --- |
| 新账号或 cookie 失效 | `sau <platform> login --account <name> --headed` |
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
