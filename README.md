# 赛博小雅直播版 · 组件版数字人（Vidu s2）

> **本项目永久免费开源。** 唯一官方来源：`https://github.com/<组织>/<仓库>`
> 任何付费售卖、代下载、代部署均未经授权，欢迎举报（举报邮箱/群：`<待填>`）。
>
> 如果你是在某个平台花钱买的这份代码 —— 你被收智商税了，请直接来这里取免费的。

Vidu 只负责**数字人渲染与推流**；LLM、TTS、弹幕处理、歌单全部跑在你自己的电脑上。

## 这是干什么的

一套可以在本机跑起来的「数字人直播间」：

- 浏览器里的**舞台页**（`/stage`）显示数字人画面、字幕、弹幕事件、动作特效
- 本地 **FastAPI 桥**（默认 `:8795`）负责会话生命周期、说话管线、动作调度
- 说话走 **LLM → 分句 → TTS（OmniVoice）→ 裸 PCM 实时喂给 Vidu**
- 弹幕从抖音 / B站 / YouTube / 视频号 采集进来，可以复读、可以翻译成动作指令
- 支持点播歌单（带歌词字幕）、快捷指令面板、自动录播

## 一句话架构

```
舞台页(浏览器) ──文字──▶ 桥(FastAPI :8795) ──type9──▶ Vidu WS
                          │  ▲
              LLM 流式 ◀──┘  └── PCM s16le/24kHz/mono 二进制帧(48B/ms 实时节奏)
                          │
        本机 TTS(OmniVoice :9877) / audio 目录点播文件 ──▶ ffmpeg 统一解码
Vidu: 渲染口型+动作 ──▶ RTC 频道(托管/自备) ──▶ 舞台页订阅看画面听声音
```

## 快速开始

```powershell
# 1) 建环境(Python 3.12)
uv venv .venv --python 3.12
uv pip install -r requirements.txt --default-index https://pypi.tuna.tsinghua.edu.cn/simple

# 2) 复制配置并填写你自己的 key
copy config.example.json config.json
#   vidu.api_key        ← 你的 Vidu API Key
#   avatar.avatar_id    ← 你上传到 Vidu 的形象 ID(或 image_uri)
#   rtc.artc.app_id / app_key  ← 仅自备 RTC 模式需要
#   llm.key_file        ← 存 DeepSeek(或兼容) API Key 的文件路径

# 3) 起服务
.venv\Scripts\python.exe -m src.main        # 桥 + 舞台页 http://127.0.0.1:8795/stage
```

一键启动脚本：双击 `启动-组件版全链路.cmd`（会读环境变量，缺的组件自动跳过）。

### 可选组件（都可不要）

| 组件 | 作用 | 不装的后果 |
|---|---|---|
| OmniVoice（WSL2 :9877） | 克隆音色的 TTS | 没有说话声音，其他功能正常 |
| 弹幕采集器（multi-danmaku-tts） | 抖音/B站/YouTube 弹幕 | 只能手动打字互动 |
| 点播歌单（audio/ 目录） | 放歌 + 歌词字幕 | 没有点播 |

用环境变量指路：`DSH_OMNIVOICE_CMD`、`DSH_DANMAKU_ROOT`、`DSH_VOICE_BRIDGE_CFG`。

## 需要你自己准备的素材

仓库**不附带**以下内容（这是作者的核心资产，也是防止有人拿去做二次售卖的边界）：

- 人物形象（图片/视频，上传到 Vidu 换取 `avatar_id`）
- 克隆音色的参考音频 + 参考文本
- 你的人设提示词（`llm.system_prompt`）、动作库、歌单
- 直播账号的 Cookie / API Key

## 目录结构

```
src/            桥与引擎(bridge/engine/llm/tts/vidu/dm_router)
stage/          舞台页(单文件 HTML)
tools/          自检、回放、歌词打轴、动作库同步等脚本
audio/          点播目录(歌不入库)
config.example.json  配置模板(不含任何密钥)
```

## 许可证

- **代码**：[GNU AGPL-3.0](./LICENSE) —— 你可以自由使用、修改、分发；但分发/提供网络服务时必须同样开源。
- **素材与品牌**：见 [NOTICE](./NOTICE)，不在 AGPL 授权范围内。
- 第三方依赖各自的许可证见 `requirements.txt` 对应项目。

## 关于倒卖

本项目**代码免费、永久免费**。我们不会在任何平台出售它。

如果你看到有人在卖，那不是官方行为，欢迎把链接发到 `<举报渠道>`。同时提醒买家：盗版没有更新、没有答疑、出了事也没人管。
