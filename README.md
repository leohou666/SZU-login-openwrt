# ⚡ srun-login ![Go](https://github.com/leohou666/SZU-login-openwrt/workflows/Go/badge.svg) [![Go Report Card](https://goreportcard.com/badge/github.com/leohou666/SZU-login-openwrt)](https://goreportcard.com/report/github.com/leohou666/SZU-login-openwrt)

深圳大学校园网自动登录工具，支持深澜（srun）教学区和宿舍区，**原生支持 OpenWrt**。

## 特性

- `config.yaml` 配置文件，无需命令行传密码
- 可指定服务器 IP，绕过 DNS 解析问题
- 持续监控 + 断线自动重连
- 强制 IPv4 检测，避免 OpenClash 等代理通过 IPv6 造成误判
- OpenWrt 静态二进制，开箱即用

## 快速开始

编辑 `config.yaml`，然后：

```bash
go build -o szu-login ./cmd/
./szu-login
```

### config.yaml 示例

```yaml
credentials:
  username: "学号"
  password: "密码"

network:
  teaching:
    enabled: true
    url: "https://net.szu.edu.cn/"
    ip: "172.31.63.36"  # 可选，防止 DNS 解析问题
    ac_id: "12"
  dormitory:
    enabled: false
    url: "http://172.30.255.42:801/eportal/portal/login/"
    ip: ""

monitor:
  enabled: true
  interval: 60
  test_urls:            # 只用国内地址，避免 IPv6 代理误判
    - "http://connect.rom.miui.com/generate_204"
    - "http://wifi.vivo.com.cn/generate_204"

debug:
  enabled: false
  verbose_network_detection: false
  timeout: 10
```

### 命令行参数

```
--username / --password     凭据（覆盖配置文件）
--host                      登录服务器 URL
--teaching-ip               教学区服务器 IP
--dormitory-ip              宿舍区服务器 IP
-i / --interface            绑定网卡（Linux）
```

---

## OpenWrt 部署

### 下载预编译二进制

从 [GitHub Actions Artifacts](https://github.com/leohou666/SZU-login-openwrt/actions) 下载对应架构：

| 架构 | 文件名 |
|------|--------|
| x86_64（软路由） | `szu-login-openwrt-x86_64` |
| ARM64 | `szu-login-openwrt-aarch64` |
| MIPS（小米/TP-Link 等） | `szu-login-openwrt-mipsle` |

### 部署步骤

```bash
# OpenWrt 默认无 sftp，需加 -O 使用旧协议
scp -O szu-login-openwrt-x86_64 root@192.168.1.1:/root/szu-login
scp -O config.yaml root@192.168.1.1:/root/config.yaml

ssh root@192.168.1.1 "chmod +x /root/szu-login && /root/szu-login"
```

### 开机自启（procd）

创建 `/etc/init.d/szu-login`：

```sh
#!/bin/sh /etc/rc.common
START=99
USE_PROCD=1

start_service() {
    procd_open_instance
    procd_set_param command /root/szu-login
    procd_set_param respawn
    procd_set_param stdout 1
    procd_set_param stderr 1
    procd_close_instance
}
```

```bash
chmod +x /etc/init.d/szu-login
/etc/init.d/szu-login enable
/etc/init.d/szu-login start
```

### OpenWrt + OpenClash 注意事项

`test_urls` 只填国内地址（miui/vivo 的 generate_204）。程序强制 IPv4 拨号，`generate_204` 返回 204 才视为联网，302（门户劫持）视为未登录需重新认证。

---

## License

MIT License
