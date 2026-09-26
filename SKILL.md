---
name: letsencrypt-windows
description: >-
  在 Windows 上申请、安装与续期 Let's Encrypt TLS 证书；优先 Posh-ACME + 阿里云 DNS-01，
  并可上传到阿里云 CDN。在用户提到 Let's Encrypt、SSL/TLS、HTTPS、Posh-ACME、
  Windows 证书续期、CDN 证书时使用。
disable-model-invocation: true
---

# letsencrypt-windows

## 何时使用

在 Windows 上为域名申请 / 续期 Let’s Encrypt 证书，尤其是：

- DNS 在阿里云云解析
- 证书要挂到**阿里云 CDN**（而非仅本机站点）
- 阿里云免费 SSL 额度不可用，需改走 LE

## 权威文档

先读再动手：

1. [docs/README.md](docs/README.md) — 索引与结论  
2. [docs/windows-posh-acme-cdn.md](docs/windows-posh-acme-cdn.md) — 逐步操作  
3. [docs/lessons-learned.md](docs/lessons-learned.md) — 选型与踩坑  

## 推荐步骤（摘要）

1. `Install-Module Posh-ACME`，确认插件含 `Aliyun`  
2. `Set-PAServer LE_PROD`  
3. `New-PACertificate <域名> -DnsPlugin Aliyun -PluginArgs @{ AliKeyId=...; AliSecret=(SecureString) }`  
4. `aliyun cdn SetCdnDomainSSLCertificate`：`CertType=upload`，`SSLProtocol=on`  
5. 业务侧将 CDN 基址改为 `https://`  
6. 约 90 天续期后**重新上传** CDN（本机续期不会自动同步云上）  

细节、参数名与失败案例见 `docs/`。
