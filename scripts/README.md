# scripts · 实机脚本

以下脚本**直接取自运行中的手机**（`/data/data/com.termux/files/home` 与容器内 `/root/`），仅做重命名，逻辑未改动。

| 文件 | 运行位置 | 作用 |
| --- | --- | --- |
| `boot-qq.sh` | Termux，需放到 `~/.termux/boot/` | 开机自启入口 |
| `start_all.sh` | Termux | 手动一键起 QQ + bot + watchdog |
| `watchdog.sh` | Termux | 每 60s 巡检，重建消失的 proot 会话 |
| `restore.sh` | Termux | 清理临时文件后恢复全套并打印状态 |
| `run_bot.sh` | 容器内 `/root/bot/` | bot 重启循环 |
| `supervisor.sh` | 容器内 `/root/` | QQ 重启循环（xvfb + QQ） |

## 使用前必须改的地方

1. **路径中的 `<BOT_QQ>`**：替换成你的机器人 QQ 号
2. **容器名 `ubuntu`**：如果你 `proot-distro install` 时用了别的名字，同步改
3. `termux-wake-lock`：确认已安装 termux-api

## 为什么脚本要写死一堆 export

```bash
export PREFIX=/data/data/com.termux/files/usr
export HOME=/data/data/com.termux/files/home
export PATH=/data/data/com.termux/files/usr/bin:/system/bin
export LD_LIBRARY_PATH=/data/data/com.termux/files/usr/lib
```

Termux:Boot 执行脚本时的环境**极其干净**，不继承交互式 shell 的 PATH。不显式 export 的话，`proot-distro` 会直接「command not found」——这个坑排查起来非常费时间，因为在终端里手动跑完全正常。
