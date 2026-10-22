# DeepSeek Bot for LazyCat

将 [DeepSeek Bot](https://github.com/Fei2411/DeepSeek-Bot) 打包为 LazyCat LPK v2。DeepSeek Bot 是运行在 DeepSeek Harness（`dsh web`）上的一支长期在线 AI 机器人小队，支持机器人之间互发消息与群聊。

## 镜像来源

上游在分支 CI 成功后自动发布镜像到 GHCR，因此直接使用官方镜像，不重新构建：

- 上游镜像：`ghcr.io/fei2411/deepseek-bot:latest`（linux/amd64；`latest` 跟随 `main`，另有 `dev` 与 `sha-*` tag）
- 运行镜像：`ghcr.1ms.run/fei2411/deepseek-bot:latest`（GHCR 默认加速），manifest 内固定 digest
- 单容器，监听容器内 `3080`

上游只发布 `latest` / `dev` / `sha-*`，没有版本 tag，所以本仓库跟踪 `latest` 的 digest：digest 变化时递增 patch 版本并重新发布。

## 部署参数

| 参数 | 说明 |
| --- | --- |
| `api_key` | 模型服务 API Key（DeepSeek 官方，或其他兼容 Anthropic Messages 协议的服务），可留空 |
| `base_url` | 自定义模型接口地址，留空则使用官方服务 |

## 登录方式（Harness 安全设计）

dsh 每次启动会生成一个随机 token 并打印在容器日志里，形如：

```text
dsh web: http://127.0.0.1:3080/?token=xxxx
```

首次使用需要：

1. 打开应用的「日志」，复制该行的 `token` 参数值；
2. 访问 `https://<本应用地址>/?token=<token>` 打开一次，浏览器写入有效期 30 天的会话 Cookie。

终端用户也可以执行 `lzc-cli app log community.lazycat.app.deepseek-bot` 查看日志。

## 宿主校验

dsh 对每个 `/api` 请求校验 `Host` 头：loopback 始终可信，其他主机名必须列在 `DSH_TRUSTED_HOST` 中，否则返回 403。manifest 里注入 `DSH_TRUSTED_HOST=${LAZYCAT_SUBDOMAIN}.${LAZYCAT_BOX_DOMAIN}` 以适配懒猫的访问域名。

## 数据与运行方式

| 容器路径 | 懒猫路径 | 内容 |
| --- | --- | --- |
| `/data` | `/lzcapp/var/data` | `DSH_HOME`：团队、群聊、会话、配置与浏览器 Cookie 密钥 |
| `/workspace` | `/lzcapp/var/workspace` | 团队工作区，机器人在此读写文件 |

- 服务名：`deepseek-bot`
- 权限：容器即沙箱边界，机器人在挂载的 `/workspace` 内拥有完整读写权限（`danger-full-access`，上游唯一预设）

## 本地构建

```bash
lzc-cli project build -o dist/application.lpk
lzc-cli lpk info dist/application.lpk
```

## 自动发布

`.github/workflows/lazycat.yml` 使用 `ca-x/lazycat-github-action` 跟踪上游 `latest` digest，创建版本化 GitHub Release Asset 并只发布到喵喵私有商店。官方商店已关闭。

## 许可

Apache-2.0（上游）