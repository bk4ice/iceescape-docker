# Secrets 目录

此目录用于存放敏感文件，通过 Docker volume 以只读方式挂载到后端容器的 `/app/secrets/` 路径。

## 和风天气私钥

如果你需要使用天气功能，请将和风天气的私钥文件放入此目录：

```bash
cp /path/to/your-private-key.pem ./secrets/qweather_private_key.pem
```

对应 `.env` 中的配置：

```
QWEATHER_PRIVATE_KEY_PATH="/app/secrets/qweather_private_key.pem"
```

## 安全须知

- **此目录中的文件绝不应提交到 Git**（已在 `.gitignore` 中排除）
- 仅 `.gitkeep` 和 `README.md` 会被提交，用于保留目录结构
- 私钥文件权限建议设置为 `600`：`chmod 600 ./secrets/*.pem`
