# openwrt-ci-roc 改动记录

> 仓库: lq-259/openwrt-ci-roc → icerobott fork 编译  
> 设备: 京东云亚瑟 (jdcloud_re-ss-01)  

---

## 新增文件

### `.github/workflows/JDCloud-LiBwrt.yml`
基于 LiBwrt 主源（openwrt-6.x）的编译 workflow，复用 Build-OpenWrt.yml。
- 源码: `LiBwrt/openwrt-6.x.git` 分支 `main-nss`
- 固件 tag: `JDCloud-LiBwrt-日期-#编号`
- 产物: NSS 全栈 + WiFi 1200 + Argon 主题

---

## 修改文件

### `.github/workflows/Build-OpenWrt.yml`

| 项 | 改动 | 目的 |
|---|---|---|
| apt 依赖 | 去掉 `ack antlr3 fastjar genisoimage lrzsz msmtp nano p7zip texinfo uglifyjs vim` 等 | 砍掉 ~2GB 无用包，初始化从 2.5h → 8min |
| apt install | 加 `--no-install-recommends`（主行 + asciidoc 行） | 阻止 `dblatex→texlive→openjdk` 依赖链 |
| 编译 | 加 4GB swap + `make -j2` | 防 OOM kill |
| manifest | `mv` 加 `2>/dev/null \|\| true` | LiBwrt 无 ImmortalWrt 命名格式的 manifest 时不炸 |
| release | tag 改为 `{TAG}-{日期}-#{编号}`，去掉 `allowUpdates` | 每次编译新 release，不覆盖 |


### `scripts/Roc-script.sh`

| 项 | 改动 |
|---|---|
| 主机名 | 按 `$SOURCE_REPO` 区分：ImmortalWrt → `ImmortalWrt`，LiBwrt → `LiBwrt` |
| QModem | 源码克隆已移除（与 NSS ECM 冲突 `nss_rmnet_rx_get_ifnum`），改为刷机后 `apk add` |
| turboacc | 已移除（`add_turboacc.sh` 修改 `nf_conntrack_ecache.c`，与 LiBwrt NSS 内核补丁冲突） |


### `configs/JDCloud.config`

| 项 | 改动 |
|---|---|
| 去掉死包 | `kmod-usb-net-qmi-wwan-fibocom`、`mbim-cli`（不存在的包名） |
| QModem | 已注释（NSS ECM 冲突），刷机后 `apk` 后装 |
| Argon 主题 | `luci-theme-argon=y`、`luci-app-argon-config=y` |
| 内核模块 | `kmod-nf-flow=y`、`kmod-ipt-offload=y`、`kmod-tcp-bbr=y` |


### `configs/General.config`

| 项 | 改动 |
|---|---|
| Argon | `=y`（原 `=n`） |
| Aurora | 删除（原 `luci-theme-aurora=y`、`luci-app-aurora-config=y`） |
| ACME | 删除（原 `acme-acmesh-dnsapi=y`、`luci-app-acme=y`） |

---

## 已知限制

| 问题 | 说明 |
|---|---|
| QModem 无法源码编译 | 与 LiBwrt/ImmortalWrt 的 NSS ECM 均冲突，改后装（LiBwrt 用 apk，ImmortalWrt 也是 apk，ABI 对齐） |
| turboacc UI 不可用 | `add_turboacc.sh` 打补丁动 `nf_conntrack_ecache.c`，与 LiBwrt NSS 内核补丁冲突 |
| 关 NSS 只能用 CLI | `qca-nss-ecm stop && qca-nss-ecm disable`，无图形界面 |

---

## 刷机后 QModem 安装（LiBwrt/ImmortalWrt 通用）

```bash
cd /tmp
wget https://gh.255913.xyz/https://github.com/FUjr/QModem/releases/download/v3.1.0/QModem-arm64_apk.tar.gz
tar -xzf QModem-arm64_apk.tar.gz
apk add --allow-untrusted *.apk
```

LiBwrt 快照与 QModem apk 原生对齐，无需 ABI hack。ImmortalWrt 同样适用。
