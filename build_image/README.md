# FastDFS 镜像构建

## 项目简介

本目录包含构建 FastDFS 镜像的相关文件，支持 ARM 架构。

## 构建步骤

### 1. 准备依赖

构建目录中已包含所需的依赖包：

- `soft/libfastcommon-master.zip` - FastDFS 依赖库
- `soft/fastdfs-6.08.zip` - FastDFS 核心代码
- `soft/fastdfs-nginx-module-1.22.zip` - FastDFS Nginx 模块

### 2. 构建镜像

```bash
# 进入构建目录
cd build_image

# 构建镜像
docker build -t <your-registry>/fastdfs:ubuntu26arm .

# 推送镜像（可选）
docker push <your-registry>/fastdfs:ubuntu26arm
```

## 配置说明

### 1. 主要配置文件

- `conf/tracker.conf` - Tracker 服务配置
- `conf/storage.conf` - Storage 服务配置
- `conf/client.conf` - 客户端配置
- `conf/mod_fastdfs.conf` - Nginx 模块配置
- `conf/http.conf` - HTTP 访问配置

### 2. 存储路径

- **基础路径**：`/data/fastdfs_data`
- **存储路径**：`/data/fastdfs_data/upload`
- **日志路径**：`/data/fastdfs_data/logs`

### 3. 服务端口

- **Tracker**：22122
- **Storage**：23000
- **Nginx**：80

## 启动脚本

`start.sh` 脚本用于启动 FastDFS 服务：

- `./start.sh tracker` - 启动 Tracker 服务
- `./start.sh storage` - 启动 Storage 服务

## Nginx 配置

- `nginx_conf/nginx.conf` - Nginx 主配置
- `nginx_conf.d/default.conf` - 虚拟主机配置

## 注意事项

1. **存储路径**：确保 `/data/fastdfs_data` 目录已挂载到持久化存储

2. **权限**：确保存储目录有正确的权限，FastDFS 服务能够正常读写

3. **网络**：确保 Tracker 和 Storage 之间能够正常通信

4. **ARM 架构**：镜像已针对 ARM 架构优化，可在 ARM 设备上正常运行

## 故障排查

### 1. 查看日志

- Tracker 日志：`/data/fastdfs_data/logs/trackerd.log`
- Storage 日志：`/data/fastdfs_data/logs/storaged.log`
- Nginx 日志：`/data/fastdfs_data/logs/`

### 2. 常见问题

- **存储路径不存在**：启动脚本会自动创建所需的目录结构
- **端口占用**：确保相关端口未被其他服务占用
- **权限错误**：检查存储目录权限设置

## 许可证

本项目基于 MIT 许可证开源。