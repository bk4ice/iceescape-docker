# SSL 证书目录

此目录用于存放 HTTPS 证书和私钥，仅在 `docker-compose.prod.yml`（生产环境）中使用。

## 使用方法

将你的 SSL 证书和私钥放入此目录：

```bash
cp /path/to/your/cert.pem ./ssl/cert.pem
cp /path/to/your/cert.key ./ssl/cert.key
```

对应 `.env` 中的配置：

```
SSL_CERT_PATH=./ssl/cert.pem
SSL_KEY_PATH=./ssl/cert.key
```

## 证书获取

- 免费证书：[Let's Encrypt](https://letsencrypt.org/)
- 阿里云/腾讯云等也可申请免费 DV 证书

## 安全须知

- **此目录中的文件绝不应提交到 Git**（已在 `.gitignore` 中排除）
- 仅 `.gitkeep` 和 `README.md` 会被提交，用于保留目录结构
- 私钥文件权限建议设置为 `600`：`chmod 600 ./ssl/*.key`
