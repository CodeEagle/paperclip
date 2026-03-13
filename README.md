# Paperclip - 懒猫微服迁移项目

本项目将上游 [paperclipai/paperclip](https://github.com/paperclipai/paperclip) 打包为可在懒猫微服中安装的应用。

## 应用说明

Paperclip 是一个面向多智能体团队协作的开源运营平台，提供任务编排、组织结构、预算治理、审计日志和 Web 控制台。

本迁移仓库采用上游官方 Dockerfile 源码构建镜像，运行时使用 `paperclip + postgres` 两服务模式：

- Web 入口：`/`
- 容器内端口：`3100`
- 应用数据目录：`/paperclip`
- 数据库目录：`/var/lib/postgresql/data`
- 持久化挂载：
  - `/lzcapp/var/data -> /paperclip`
  - `/lzcapp/var/db/paperclip/postgres -> /var/lib/postgresql/data`

## 首次启动

首次安装后，至少需要确认以下配置：

- `BETTER_AUTH_SECRET`
  用于认证签名，未配置时应用无法完成认证模式启动。
- `PAPERCLIP_PUBLIC_URL`
  建议设置为懒猫分配给应用的实际访问地址，避免登录回调或邀请链接指向错误地址。

数据库连接由 manifest 内置：

- `DATABASE_URL=postgres://paperclip:paperclip@postgres:5432/paperclip`

默认环境变量：

- `PAPERCLIP_DEPLOYMENT_MODE=authenticated`
- `PAPERCLIP_DEPLOYMENT_EXPOSURE=private`
- `HOST=0.0.0.0`
- `PORT=3100`
- `SERVE_UI=true`
- `PAPERCLIP_HOME=/paperclip`

可选环境变量：

- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`

## 数据说明

根据上游 `doc/DOCKER.md` 和当前迁移拓扑，以下数据会被持久化：

- Paperclip 本地文件、附件、workspace 与实例目录：`/lzcapp/var/data`
- PostgreSQL 数据：`/lzcapp/var/db/paperclip/postgres`
- 上传附件
- 本地 secrets key
- 本地 agent 工作目录

为避免懒猫挂载目录默认属主导致的写权限问题，manifest 会在启动前自动创建 `/paperclip/instances/default/logs` 并把 `/paperclip` 递归授权给容器内 `node` 用户。

## 构建与发布

仓库内的 `.github/workflows/update-image.yml` 会执行以下流程：

1. 获取上游最新 semver 版本
2. 从上游源码 tag 构建 `ghcr.io/CodeEagle/paperclip:<source-commit-sha>`
3. 复制镜像到 `registry.lazycat.cloud/...`
4. 精确更新 `lzc-manifest.yml` 与 `.lazycat-build.json`
5. 生成 `.lpk` 并发布 GitHub Release

## 上游链接

- Upstream Repo: https://github.com/paperclipai/paperclip
- Homepage: https://paperclip.ing
- Docker 文档: https://github.com/paperclipai/paperclip/blob/master/doc/DOCKER.md
- License: MIT
