# 把个人 QQ 号变成 AI 聊天机器人 · 全程跑在 Android 手机里

> 一份实机落地记录：**宿主是一台安卓手机，不是电脑、不是云服务器。**
> 目标是把一个 QQ 号交给 AI 接管，日常操作趋近于零 —— 不用扫码、不用手动启动、不用管它。

## 为什么是手机

| 方案 | 问题 |
| --- | --- |
| 电脑常开 | 要一直开机、耗电、异地带不走；关一次就掉线 |
| 云服务器 | 个人 QQ 号异地登录极易触发风控验证，且无图形界面跑不了 Linux QQ |
| **闲置安卓手机** | 内置电池 = 自带 UPS、常插充电器、7×24 在线、可以随手挪走 |

结论：**找一台闲置手机当唯一宿主，电脑只承担「装 APK / 看日志调试 / 供电」三件事。**

## 整体架构

```
┌─────────────────────── Android 手机（唯一宿主）───────────────────────┐
│                                                                       │
│  Termux（非 root）                                                     │
│    │                                                                  │
│    ├─ proot-distro: Ubuntu 24.04 容器                                 │
│    │    ├─ Linux QQ  /opt/QQ/qq  ── xvfb 虚拟屏 ── 免扫码登录           │
│    │    ├─ NapCat  ── 注入 QQ，暴露 OneBot 协议                        │
│    │    │      HTTP :3000   /   WebSocket :3001                       │
│    │    └─ Python bot  ── 连 WS :3001 收发消息                         │
│    │           └─ DeepSeek API（聊天）/ edge-tts（语音）               │
│    │                                                                  │
│    └─ watchdog.sh  每 60s 检查两个 proot 会话，挂了自动拉起             │
│                                                                       │
│  Termux:Boot → ~/.termux/boot/start-qq.sh  →  开机全自动               │
└───────────────────────────────────────────────────────────────────────┘
```

## 技术栈

| 层 | 选型 | 备注 |
| --- | --- | --- |
| 宿主 | vivo X80 Pro（PD2185）/ Android 16 / 12G+512G | 闲置机，长期插电 |
| 终端环境 | Termux（F-Droid 版）+ termux-boot | **不用 root** |
| Linux 环境 | proot-distro · Ubuntu 24.04（aarch64） | 用户态容器，够跑 QQ |
| QQ 客户端 | Linux QQ | 配合 xvfb 无头运行 |
| 协议层 | NapCat | 注入 NTQQ，输出 OneBot 11 |
| Bot | 自写 Python（单文件 bot.py） | 没用 LangBot，见 [docs/02](docs/02-环境与选型.md) |
| LLM | DeepSeek `deepseek-chat` | OpenAI 兼容格式，换模型只改 3 个变量 |
| TTS | edge-tts | 本地大模型在 ARM 上不可实时，见 [docs/07](docs/07-语音与TTS.md) |

## 目录

| 文档 | 内容 |
| --- | --- |
| [docs/01-整体架构.md](docs/01-整体架构.md) | 进程树、端口、数据流向、路径约定 |
| [docs/02-环境与选型.md](docs/02-环境与选型.md) | 为什么 Termux+proot、为什么 NapCat、为什么放弃 LangBot |
| [docs/03-部署全流程.md](docs/03-部署全流程.md) | 从一台空手机到机器人上线 |
| [docs/04-开机自启与进程守护.md](docs/04-开机自启与进程守护.md) | Termux:Boot、wake-lock、双层守护循环 |
| [docs/05-配置管理.md](docs/05-配置管理.md) | 「代码里的 DEFAULTS 是唯一事实来源」及它的代价 |
| [docs/06-群聊状态机.md](docs/06-群聊状态机.md) | 潜水 / 活跃 / 静默 / 休眠四态 |
| [docs/07-语音与TTS.md](docs/07-语音与TTS.md) | ARM 上各 TTS 方案实测结论 |
| [docs/08-踩坑记录.md](docs/08-踩坑记录.md) | 12 个真实故障与解法 |
| [docs/09-运维与排障.md](docs/09-运维与排障.md) | 日志位置、adb 命令、三条恢复路径 |
| [docs/10-安全边界与风险.md](docs/10-安全边界与风险.md) | 权限模型、提示词注入防护、封号风险 |
| [scripts/](scripts/) | 实机脚本（删掉凭据后可直接复用） |

## 五条最有价值的结论

1. **`setsid nohup ... < /dev/null` 是手机后台进程能活下来的关键**，缺 `< /dev/null` 时 proot 会话会在父 shell 退出时被带走。
2. **`termux-wake-lock` 必须调**，否则息屏后 Android 会冻结 Termux，机器人静默掉线。
3. **QQ / NapCat / bot 三层各要一个 `while true` 重启循环**，只靠外层看门狗会有几十秒的空窗。
4. **配置千万别整份持久化**，否则代码里改新默认值会被旧文件永久锁死（我踩过，见 docs/05）。
5. **本地 TTS 在 ARM 手机上不现实**：ChatTTS 实测 **86 秒/句**，只能走 edge-tts 这类在线接口。

## 风险提示

- 个人 QQ 号跑机器人**违反 QQ 用户协议，有封号风险**，本方案是在知情前提下自用。
- 仓库内所有凭据均为占位符，真实密钥保存在手机本地，不随代码分发。
- 请勿把本方案用于群发、加好友引流等用途。

## License

MIT
