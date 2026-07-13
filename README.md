# 京东云亚瑟 OpenWrt 云编译

> 设备: jdcloud_re-ss-01 / jdcloud_re-cs-02 (IPQ60xx)  
> 上游: [laipeng668/openwrt-ci-roc](https://github.com/laipeng668/openwrt-ci-roc)

---

## 固件信息

| 项目 | 值 |
|---|---|
| 管理地址 | `192.168.15.1` |
| 用户名 | `root` |
| 密码 | `password` |
| 主题 | Argon |
| 主机名 | ImmortalWrt（ImmortalWrt 版）/ LiBwrt（LiBwrt 版） |

## 两种编译版本

| | JDCloud-ImmortalWrt | JDCloud-LiBwrt |
|---|---|---|
| 源码 | [laipeng668/immortalwrt](https://github.com/laipeng668/immortalwrt) | [LiBwrt/openwrt-6.x](https://github.com/LiBwrt/openwrt-6.x) `main-nss` |
| NSS | 半闭（WiFi 不走 NSS） | 全开（WiFi 1200） |
| 内核 | 6.12 | 6.12 |
| 包管理器 | apk | apk |
| 适用场景 | 低负载（0.x），USB WAN | 满血 WiFi，硬 WAN/LAN NSS 加速 |

## 内置软件包

- **OpenClash** — 代理
- **SFTP** — `openssh-sftp-server`
- **USB 供网** — ECM/NCM/QMI/MBIM 全驱动 + quectel-cm/uqmi/umbim
- **Argon 主题** + argon-config
- **网络加速** — `kmod-nf-flow` / `kmod-ipt-offload` / `kmod-tcp-bbr`
- **常用工具** — htop/btop/fdisk/nano/tmux/ddns/upnp/wol/vlmcsd/frps/frpc/diskman/cpufreq 等

## QModem（刷机后安装）

QModem 源码编译与 NSS ECM 冲突，改为刷机后 apk 后装：

```bash
cd /tmp
wget https://gh.255913.xyz/https://github.com/FUjr/QModem/releases/download/v3.1.0/QModem-arm64_apk.tar.gz
tar -xzf QModem-arm64_apk.tar.gz
apk add --allow-untrusted *.apk
```

## 编译

1. Fork → Actions → 选 **JDCloud-ImmortalWrt** 或 **JDCloud-LiBwrt** → Run workflow
2. 编译约 1-2h，产物发布到 Releases（每次独立 tag，不覆盖历史版本）
3. `SYSUPGRADE.BIN` 从现有 OpenWrt 升级，`FACTORY.BIN` 从 uboot 刷入

## 定制

- 软件包：修改 `configs/JDCloud.config` 或 `configs/General.config`
- 脚本：修改 `scripts/Roc-script.sh`（IP/密码/插件克隆等）
- 不需要的包把 `y` 改成 `n`，仅加 `#` 无效
