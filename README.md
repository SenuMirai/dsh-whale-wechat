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
| [dsh-whale-douyin](https://github.com/SenuMirai/dsh-whale-douyin) | 🎵 **刷抖音**：在本地 Edge 里刷抖音电脑版，按兴趣和真实观看行为更新口味 |

> 命名统一用 `dsh-whale-` 前缀，看到这个前缀就是一家的。**npm 包名沿用旧 id**（`dsh-qq-group` 这种），因为几个插件之间靠 id 互相认亲，改名会把联动弄断。

### 推荐一起装的插件（别人做的，很好用）

| 插件 | 干什么 |
|---|---|
| [dsh-see-now](https://github.com/deepseek-ai/dsh-see-now) | 让她**看得见你的屏幕**——直播式连续画面，不是一张截图 |
| [dsh-loadout](https://github.com/YongjianWan/dsh-loadout) | 按需**装卸工具组**，几十个工具不至于一次全糊进上下文 |
| [dsh-persona-distill](https://github.com/Speraymer/dsh-persona-distill) | **人格蒸馏**：从聊天记录里蒸馏出人设，省得手写几万字提示词 |

### 好用的语料

| 语料 | 干什么 |
|---|---|
| [b-corpus](https://huggingface.co/datasets/Limour/b-corpus) | 开源的 **galgame 文本库**。想让她说话带那味儿，就拿去喂人格蒸馏、做语气样本 |
| [galgame-script-corpus](https://github.com/m-rui001/galgame-script-corpus) | 视觉小说剧本语料：**英/日/中三语 450+ 条目**，命名规范统一（`[中文]` `[日本語]` `[中日对照]`）。比 b-corpus 更适合挑**中日对照**做语气校准 |

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

## 踩过的坑（都是真踩过的，按这个顺序排）

这一节是这套东西最值钱的部分。**下面每一条都是我们把功能写完之后又花了两三天才查出来的**，
照着躲能省很多时间。

### 1. 运行时千万别放在「受管目录」里

最有欺骗性的一条。症状是：一切看起来都正常，端口都通、日志也没报错，
但发消息一律失败，回一个 `tree_not_ready`。

真因：`C:\DSHwork` 这类目录下的 `python.exe` 会拿到**受限令牌**。实测：

```
C:\DSHwork\xxx\.venv\Scripts\python.exe
    explorer  OpenProcess(0x0410) -> DENY err=5
    微信      OpenProcess(0x0410) -> DENY err=5

C:\Program Files\Python312\python.exe
    explorer  OpenProcess(0x0410) -> OK
    微信      OpenProcess(0x0410) -> OK
```

把官方 `python.exe` **原样复制**到 `C:\DSHwork\pycopy.exe`，立刻又变 DENY ——
所以不是文件问题，是**路径**问题。

后果链：受限令牌 → 读不了微信内存 → 拿不到 `Weixin.dll` 基址 →
写不了 accessibility gate 字节 → 微信的 Qt 控件树永远物化不出来 → 发送全废。

**做法**：运行时用 `C:\Program Files\Python312\python.exe`，
venv 只当依赖目录（把 `site-packages` + `win32` + `pythonwin` 塞进 `PYTHONPATH`）。

### 2. 多线程共用 UIA/COM 对象树会崩进程

症状：跑几个小时突然整个进程消失，`python312.dll` 里 `0xC0000005`（访问违例），
Python 层的 `faulthandler` 连栈都来不及写。

真因：心跳、发送看门狗、收消息、贴纸测试、语音测试、提醒调度**六条线程**
同时在碰同一棵 uiautomation 对象树。加锁只覆盖了一部分调用点，
异常清理分支、兜底路径、GC 在任意线程回收对象，都还能绕过去。

**做法**：把整棵对象树收敛到**一条专用 owner 线程**（`bridge/uia_worker.py`）：

- 请求/响应封套，请求只带纯数据
- **重入直调**：已经在 owner 线程上就直接执行，嵌套调用不会自己等自己
- 异常在 worker 里捕获、带回调用方再抛，`try/except` 语义不变
- 超时只让调用方拿到 `TimeoutError`，**worker 不重建**（它跟 COM 生命周期绑定）
- COM 控件引用**绝不跨线程传递** —— 连只读的探针也只回传纯数据
- 关闭顺序：停入队 → 排空 → 在 owner 线程里释放对象 → 反初始化 COM → join

### 3. 「钉住的窗口」一旦指错，会永远好不了

症状：日志刷 `热激活 UIA 失败：PID 7392 未找到 Weixin.dll`，
`UIA 树不可用，本次会话使用 OCR 驱动`，每轮 22 秒，死活不自愈。

真因：`config.json` 的 `wechat.windowHwnd` 留着旧句柄，而那个句柄
**是别的程序**（实测是 ChatGPT 的通知宿主窗，因为窗口特别大被自动认窗挑中）。
老代码只问了一句 `IsWindow()` 就认定「这是微信主窗」，
而 UIA 那边只认钉住的 hwnd —— 于是拿着别人的窗口去做热激活，永远失败。

**做法**：钉窗要过两道验证 —— 类名像微信主窗（`QWindowIcon` / `mmui::`）
**且**所属进程真的加载了 `Weixin.dll`。不过就清零改走自动认窗。
另外心跳里连续两次拿不到 UIA 就主动解钉重认，别傻等错窗口自己消失。

### 4. 用 `Start-Process` 启动的进程读不到 UIA

同一个提权环境、同一个微信窗口：

```
Start-Process 启动的进程 → 读到 0 个 UIA 节点（发不出消息）
subprocess 启动的进程   → 读到 165 个节点（正常）
```

`Start-Process` 会**重建进程令牌**，UIA 的跨进程通道就断了。

**做法**：启动 shim / outbox / 桥一律用 `subprocess.Popen`
（`launch-outbox.py` / `launch-bridge.py`），不要用 PowerShell 的 `Start-Process`。

### 5. 一次点击弹出 6~7 个卡死窗口

真因是两个 bug 叠在一起：

1. `start-all.cmd` 里对 shim / 唤醒窗口 / outbox / 桥**各来一次**
   `Start-Process -Verb RunAs` = 3 个 UAC
2. 插件的 `/start-stack` 跑完，客户端紧接着又调 `/start`，
   而 `/start` 内部**也会调** `ensureNapcat()` —— 于是又跑一遍

窗口「卡住不动」是因为它们全都挂在等 UAC 上。

**做法**：收敛成**一次点击 = 一次提权**。`launch-stack.ps1` 里
先查端口（全在就直接返回）、再看锁文件（冷却期内不复提权），
都没有才提权一次。插件侧的 `ensureNapcat` 还要加单飞锁，防并发重复触发。

### 6. 消息发出去了，但对面收不到

症状：她的回复停在会话里，一个字都没到对方那边。

真因（有两个，都很隐蔽）：

**(a) `outlets` 只配了一个平台。** 修好一个平台时把另一个的配置顶掉了。
翻 `bridge-debug.jsonl` 的历史能一眼看出来：
某天之前是 `outlets=qq`，之后变成 `outlets=wechat`。

**(b) 就算两个都配上，`reply` 还是会被广播到所有 outlet。**
QQ 和微信都声明 `reply` 时，微信说的话会同时发到 QQ，反之亦然 —— 互相串台。
所以配置里只能二选一，这也是 (a) 发生的动机。

**做法**：每个平台调 `/dsh-world/bridge/say` 时带 `ticket = 自己的 outlet 名`，
世界那边按 ticket 只回给**送话进来的那一条线**；ticket 为空仍然广播（桌宠等老调用方不受影响）。

### 7. `bridge-hang.log` 会吃掉几个 G

`faulthandler` 每 40 秒把全部线程栈 dump 一次，跑一天就是几百 MB 到几 GB。
它不是崩溃日志，是「卡死时能看出停在哪一步」的探针。
**看的时候只 `tail`，别整个 `cat`** —— 我在调试时因为一次读了几万行把编辑器顶爆过。

### 8. 改了 `cordis.patch.yml` 不重启 = 没改

配置是启动时读的。改完不重启 DSH，跑的还是旧配置。
我们在这里浪费过一整轮排查：以为自己的修改没生效，其实是进程比改动还老。

### 9. 空文本别往发送队列里塞

`text: ""` 的任务有两种可能：真的是空的，或者**它是语音/图片任务**
（内容在 `voice` / `images` 字段里）。日志只打印 `text`，看着就像丢了消息。

**做法**：入队前判「正文 + 图片 + 语音」三样是否全空，全空才拒；
纯空白字符串（`"   "`）按空处理。另外日志里把媒体字段也打出来，别让人误判。

### 10. 概率配置别让「模型已经决定的事」再掷一次骰子

她的表情包有三种来源，原来共用同一个 `imageChance`：

```
对方明确要图   → requestedImageChance（0.9）
自己随手配一张 → 1.0
模型点名要发   → imageChance（0.08）   ← 问题在这
```

模型明明已经挑好了图、话都说出口了，又被 8% 的概率拦掉 ——
表现就是「她说『给你挑了张图』，图却没来」。**92% 的情况下图会被静默丢弃。**

**做法**：模型点名就等于决定，直接必发；随机配图才走概率。
如果不想改代码，也可以关掉「按概率强制发送」、把自动概率拉满，
代价是她不会再主动随手配图。

---

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
