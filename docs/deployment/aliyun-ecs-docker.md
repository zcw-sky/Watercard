# 阿里云 ECS Docker 部署方案

## 部署目标

首期在一台阿里云 ECS 上运行 Docker Compose。图片存 ECS 本地持久目录以节省对象存储成本。小程序由微信平台托管，构建产物通过微信开发者工具上传，不放在 Docker 内运行。

## Compose 服务建议

- `nginx`：接收 HTTPS 请求并转发到 Laravel/PHP-FPM。
- `app`：Laravel API 和 Filament 管理后台。
- `mysql`：MVP 可先部署在同一台 ECS；数据库文件使用独立持久卷。
- 不默认部署 Redis。

## 持久化目录

建议主机目录结构：

```text
/srv/watercard/
├── uploads/       # 商品、公告等上传图片
├── mysql-data/    # MySQL 数据目录（若使用容器数据库）
├── backups/       # 数据库和图片备份输出
└── app.env        # 不提交到 Git 的生产环境变量
```

将 `uploads/` 挂载到 Laravel 可访问的存储目录，将 `mysql-data/` 挂载到 MySQL 数据目录。容器应可重建，持久数据不得只存在容器可写层。

## 访问路径

- 小程序 API 和后台共用已备案/配置的 HTTPS 域名，例如 `https://api.example.com`。
- Nginx 终止 TLS，后端只接受来自 Nginx/内部 Docker 网络的请求。
- MySQL 端口不对公网开放。
- 图片可由 Nginx 通过受控静态路径提供，也可由 Laravel 生成访问 URL；上传目录不得允许执行 PHP。

## 备份和恢复

- 定期备份 MySQL 数据和 `uploads/` 图片目录。
- 备份文件应保存在独立目录/介质，避免与生产数据同盘故障时同时丢失。
- 上线前至少演练一次从备份恢复数据库和图片。
- 同机容器持久卷可降低误删容器造成的数据丢失，但不能替代异地备份。

## 发布流程

1. 构建并标记 Laravel 应用镜像。
2. 更新 ECS 上的 Compose 镜像并滚动重启服务。
3. 执行数据库迁移。
4. 检查后台登录、商品图片访问、小程序 API 和订单创建。
5. 保留上一个镜像标签，便于应用回滚。

## 后续扩容路径

- ECS 本地图片改为阿里云 OSS：通过 Laravel 文件存储配置切换。
- 同机 MySQL 改为阿里云 RDS：调整连接配置并迁移数据。
- 多台应用服务器前，先迁出本地图片，因为单机本地磁盘不适合作为多实例共享文件系统。
