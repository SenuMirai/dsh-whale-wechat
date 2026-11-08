# dsh-wechat-group · 她在你的微信里

把 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 里的陪伴角色接进**微信**：她能读到你发的消息、能回、会 @ 人、会引用、会发语音，而且**发送走后台通道，不抢你的鼠标和焦点**。

从本项目的 QQ 版 [dsh-whale-qq](https://github.com/SenuMirai/dsh-whale-qq) 拆出来改写。记忆、人设、日记全部独立，不和 QQ 那套混。

## 🐋 这是「DSH Whale 系」的一员

一套让陪伴角色**真的在过日子**的 DSH 插件。五个仓库各自独立，装哪个都行，但装在一起才是一整只：

| 仓库 | 管什么 |
|---|---|
| [dsh-whale-world](https://github.com/SenuMirai/dsh-whale-world) | 🧠 **世界**：跨会话记忆、生活作息、世界控制台。其他几个的记忆最后都落到这里 |
| [dsh-whale-qq](https://github.com/SenuMirai/dsh-whale-qq) | 💬 **QQ**：群聊/私聊接话、主动搭话、语音、唱歌、表情包、日记、看图 |
| [dsh-whale-wechat](https://github.com/SenuMirai/dsh-whale-wechat) | 💚 **微信**：私聊/群聊接话，会 @ 人、会引用、会发语音，后台发送不抢焦点 |
| [dsh-whale-bili](https://github.com/SenuMirai/dsh-whale-bili) | 📺 **刷 B 站**：她自己开 B 站看片子、记下看了什么、偶尔想跟你说一句 |
| [dsh-douyin-agent](https://github.com/LockyBee/dsh-douyin-agent) （第三方，非本系） | 🎵 **刷抖音**：在本地 Edge 里刷抖音电脑版，按人格和真实观看行为更新兴趣。我们日常在用，直接装上游 |

> 命名统一用 `dsh-whale-` 前缀，看到这个前缀就是一家的。**npm 包名沿用旧 id**（`dsh-qq-group` 这种），因为几个插件之间靠 id 互相认亲，改名会把联动弄断。

### 推荐一起装的插件（别人做的，很好用）

| 插件 | 干什么 |
|---|---|
| [dsh-see-now](https://github.com/deepseek-ai/dsh-see-now) | 让她**看得见你的屏幕**——直播式连续画面，不是一张截图 |
| [dsh-loadout](https://github.com/YongjianWan/dsh-loadout) | 按需**装卸工具组**，几十个工具不至于一次全糊进上下文 |
| [dsh-persona-distill](https://github.com/Speraymer/dsh-persona-distill) | 从聊天记录里**蒸馏人设**，省得手写几万字提示词 |

---

## 她能做什么

| 能力 | 说明 |
|---|---|
| **私聊 / 群聊** | 一对一和群里都能接话；群里默认先判断「这条该不该我说话」 |
| **@ 人** | `@` + 昵称，用**备注名**命中（不是昵称），回车选中后变成真正的 @ 对象 |
| **引用回复** | 被隔开回复时引用原消息，群聊里一眼看得出在回谁 |
| **连发多条** | 一次说几小句是常态，每条独立发送并逐条核对，不会串成乱码 |
| **发图 / 表情包** | 本地图片、微信表情、自建表情包目录 |
| **发语音** | TTS 合成 → 虚拟声卡灌进去 → 走微信自己的「按住说话」，对方收到的是真语音条 |
| **后台发送** | 用窗口消息打字 + 点按钮，**全程不抢前台**，你可以照常用电脑 |
| **主动搭话** | 太久没说话她会自己找个由头；频率、时段、冷却都可配 |
| **记忆 / 日记 / 人设** | 挂在 [dsh-whale-world](https://github.com/SenuMirai/dsh-whale-world) 的记忆中枢上；日记和人设各自独立 |
| **一键启动** | 插件设置里一个「启动」按钮，起 shim + outbox + 桥接，之后自动连上 |

## 架构

```
微信客户端 (Weixin.exe)
   ↕  UIA / 窗口消息（不抢焦点）
桥接 bridge (whale_girl)      —— 读写数据库、收发消息、TTS
   ↕  HTTP + OneBot WebSocket
shim (onebot 3101 / webhook 8790)
   ↕
dsh-wechat-group 插件
```

三个进程各自独立，谁挂了都不影响另外两个；插件里能看到每一段是不是活着。

## 安装

```bash
# 1. 插件本体
dsh plugin add dsh-wechat-group

# 2. Python 运行时（桥接和 shim 要用）
#    Windows + Python 3.12，装依赖：
pip install -r requirements.txt
```

然后在 DSH 设置里打开 **微信** 面板：

1. 填 OneBot 地址（默认 `ws://127.0.0.1:3101`）
2. 点「启动」——它会拉起整条链路，第一次可能弹一次 UAC
3. 面板显示 `link: online` 就是接上了

> ⚠️ **别把运行时放在受管目录里**。实测在 `C:\DSHwork` 这类目录下的 `python.exe` 会被系统降权，
> `OpenProcess` 对微信直接 `ACCESS_DENIED`，读不到内存就写不了 accessibility gate，
> UIA 树永远物化不出来，发送一律 `tree_not_ready`。用 `C:\Program Files\Python312\python.exe` 这种官方路径跑。

## 配置

配置在 `~/.dsh/dsh-wechat-group/qq.json`（沿用 QQ 版的文件名，别改）：

| 键 | 意思 |
|---|---|
| `enabled` | 总开关 |
| `url` | OneBot WebSocket 地址 |
| `privates` / `groups` | 白名单：只有这里列的会话会回 |
| `wechat.windowHwnd` | 手动钉微信主窗句柄，**0 = 自动**（正常都填 0） |
| `wechat.backgroundSend` | 后台发送（默认开） |
| `wechat.accountGuard` | 认不出是哪个微信号时宁可不发 |
| `tts.*` | 语音合成网关与音色 |

## 已知限制

- **Windows only**。UIA 驱动微信这条线没有跨平台替代。
- 微信大版本升级后 accessibility gate 的 RVA 会漂移，桥接会自动扫候选并逐个校验，但极端情况下要等更新。
- 首次启动需要一次 UAC（微信链路要最高权限）。之后的「启动」按钮走计划任务，不再弹。

## License

MIT

---

## English

A WeChat adapter for DeepSeek Harness: read and reply to DMs and group chats, real `@` mentions, quoted replies, multi-bubble bursts, images, stickers, TTS voice messages — all sent through a **background channel that never steals focus**.

Split out of [dsh-whale-qq](https://github.com/SenuMirai/dsh-whale-qq). Memory / diary / persona are independent from the QQ side.

**Windows only** — it drives the real WeChat client through UIA, and that path has no cross-platform equivalent.
