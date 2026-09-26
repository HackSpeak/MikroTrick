# MikroTik RouterOS PoC Collection

This repository contains proof-of-concept (PoC) scripts for several MikroTik RouterOS security issues researched and documented in 2026. The materials are intended for authorized security research, laboratory validation, and controlled testing only.

This project is not a production application, not a general-purpose tool, and not intended for use against systems without explicit authorization.

## Scope

The collection currently includes:

- CVE-2026-86060
  - Unauthenticated SSH session policy-mask swap / full-admin takeover path
  - Provided as a standalone PoC script

- CVE-2026-67276
  - SSH public-key authentication bypass PoC for affected RouterOS builds
  - Includes its helper component for signature forgery

## Repository Layout

```text
CVE-2026-mikrotik-poc/
├── README.md
├── .gitignore
├── CVE-2026-86060.py
├── CVE-2026-67276/
│   ├── CVE-2026-67276.py
│   ├── forge_67276.py
│   └── ...
└── .venv/                  # local environment, typically excluded from git
```

## Important Notice

This repository is for:

- authorized red-team exercises
- internal security validation
- research in isolated lab environments
- understanding disclosed vulnerabilities in a controlled setting

This repository must not be used against public or third-party infrastructure without proper authorization and legal review.

## Requirements

The scripts are Python-based and require:

- Python 3
- pip
- Paramiko

Install the dependency with:

```bash
python3 -m pip install paramiko
```

## Getting Started

Clone the repository and enter the project directory:

```bash
git clone <repository-url>
cd CVE-2026-mikrotik-poc
```

Create a virtual environment if needed:

```bash
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install --upgrade pip
python3 -m pip install paramiko
```

## Usage Examples

### CVE-2026-86060

Display the script's built-in help and options:

```bash
python3 CVE-2026-86060.py --help
```

Run the script against a lab target:

```bash
python3 CVE-2026-86060.py <router-ip>
```

### CVE-2026-67276

Display the script's built-in help:

```bash
python3 CVE-2026-67276/CVE-2026-67276.py --help
```

Run the script in an authorized lab environment:

```bash
python3 CVE-2026-67276/CVE-2026-67276.py --host <router-ip> --user <username>
```

## Notes

- These scripts are proof-of-concept implementations and may not be production-safe.
- Behavior can vary depending on RouterOS version, configuration, and environment.
- Some scripts rely on helper files kept in the same directory or on PYTHONPATH.
- The repository is intentionally minimal and focused on reproducibility in lab environments.

## Security and Ethical Use

Use this repository responsibly and only in accordance with applicable laws, policies, and contractual obligations. It is the user's responsibility to ensure that all testing is authorized and isolated from production systems.

## Disclaimer

This project is provided for educational and research purposes only. The authors do not condone unauthorized access or malicious use. All use must be limited to environments you own, operate, or are explicitly authorized to test.

## Maintainer

This project was created for research and defensive learning, with code associated to public disclosure research around MikroTik RouterOS vulnerabilities.

---

# 分发镜像说明(中文)

本仓库为 **MikroTrick(MikroTik RouterOS SSH 未认证接管链)** 漏洞 PoC 的中转分发镜像(英文说明见上方上游原版 README)。内容由上游公开 PoC 仓库镜像而来,仅作存档与分发用途。PoC 仅供安全研究、漏洞验证与授权测试,请勿用于未授权目标。

## 漏洞简述 / Vulnerability Summary

**MikroTrick** 是 CERT Polska 于 2026-09-05 披露、被在野利用的 RouterOS 未认证接管链,由两个缺陷串联:

- **CVE-2026-67279**(CWE-841):SSH 在用户认证前接受客户端 rekey,会话被错误推进到连接协议阶段——未认证客户端即可打开 session channel 并发送 exec 请求;
- **CVE-2026-86060**(CWE-88):SSH 登录路径参数注入。以禁用字符开头的用户名(观测到 `-2`)被登录 helper 当作"从文件描述符读取受信任身份"的语法,从而把会话策略掩码替换为完整管理员权限。

- **受影响版本**:`[7.24, 7.24.2)`、`[7.0.0, 7.23.4)`、`[6.0.0, 6.49.21)`
- **修复版本**:7.24.2(Stable)/ 7.23.4(Long-term)/ 6.49.21(Long-term)/ 7.25beta3
- **利用前提**:SSH 端口可达(默认配置不开放;暴露的多为管理员手动放开)
- **在野利用**:已确认(自 2026-09-02 起);已入 CISA KEV
- **公开 PoC**:本仓库(上游公开 PoC 镜像)

> 注:本仓库同时包含 **CVE-2026-67276**(SSH 公钥认证绕过,e=1 伪造)的 PoC 及伪造辅助脚本,该漏洞是 MikroTrick 的另一条可用入口。

## 目录结构 / Layout

```
CVE-2026-86060.py            —— 完整链路 PoC(未认证策略掩码替换 → 完整管理员)
CVE-2026-67276/
  ├── CVE-2026-67276.py      —— SSH 公钥认证绕过 PoC
  └── forge_67276.py         —— 公钥/签名伪造辅助
```

## 环境与用法 / Requirements & Usage

- 依赖:Python 3 + `paramiko`(`python3 -m pip install paramiko`)
- 目标:授权/实验室环境中的 RouterOS(脚本为 lab client,拒绝非私有地址目标)

```bash
# CVE-2026-86060 完整链路(查看帮助 / 打靶)
python3 CVE-2026-86060.py --help
python3 CVE-2026-86060.py <router-ip>

# CVE-2026-67276 公钥绕过
python3 CVE-2026-67276/CVE-2026-67276.py --help
python3 CVE-2026-67276/CVE-2026-67276.py --host <router-ip> --user <username>
```

> ⚠️ 脚本会在目标设备上创建/操纵账户(86060 路径),请仅在**授权且可销毁**的实验室环境运行。

## 检测与排查(引用公开披露)

- 升级后新版 RouterOS 会在启动时扫描篡改痕迹并置 **Flagged**(`/system/device-mode/print`);
- 日志关键痕迹:`login failure for user -2 from <ip> via ssh`、`user <name> added by ssh:-2@<ip>`;
- 检查不明高权限用户(如 `ops`)、脚本、计划任务、代理与隧道;注意 Flagged 只覆盖部分痕迹,未标记不等于干净。

## 免责声明 / Disclaimer

本 PoC 仅供教学、安全研究与授权测试使用,仅可对自有或获得明确授权的设备运行。利用会以设备最高权限执行任意命令并可能创建持久化账户,请在可销毁的实验室环境中测试。

## 归属与许可 / Attribution & License

- 上游 PoC 作者:**Gagaltotal666 / GhostGTR666**(公开 PoC 仓库 CVE-2026-mikrotik-poc)。
- 漏洞披露:**CERT Polska**;复现分析:**Bishop Fox**;厂商公告:**MikroTik**。
- 分发仓库采用 **MIT License**(见 `LICENSE`)。

## 参考链接 / References

- CERT Polska 公告:https://cert.pl/en/posts/2026/09/mikrotik-routeros-cve/
- CERT Polska 预警:https://cert.pl/en/posts/2026/09/vulnerabilities-in-mikrotik-routeros-actively-exploited/
- MikroTik 官方安全公告:https://mikrotik.com/supportsec/september-2026-vulnerability/
- Bishop Fox 技术分析:https://bishopfox.com/blog/mikrotrick-inside-the-routeros-takeover-chain
- NVD:https://nvd.nist.gov/vuln/detail/CVE-2026-67279
