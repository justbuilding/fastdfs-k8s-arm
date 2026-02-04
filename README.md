# FastDFS on Kubernetes (ARM)

## 项目简介

本项目提供了在 Kubernetes 环境中部署 FastDFS 的完整配置，支持 ARM 架构，包括国产化麒麟v10等系统。

## 主要功能

- FastDFS 分布式文件系统部署
- Nginx 集成，支持 HTTP 访问
- 持久化存储配置
- Kubernetes 资源配置文件

## 部署步骤

### 1. 创建持久化存储

#### 方法一：使用静态 PV（默认）

```bash
# 创建 PV
kubectl apply -f pv.yaml

# 创建 PVC
kubectl apply -f pvc.yaml
```

#### 方法二：使用 StorageClass

```bash
# 创建使用 StorageClass 的 PVC
kubectl apply -f pvc_use_storageclass.yaml

# 修改部署文件，使用新的 PVC
# 修改 tracker-deployment.yaml 中的 claimName 为 fastdfs-tracker-pvc-sc
# 修改 storage-deployment.yaml 中的 claimName 为 fastdfs-storage-pvc-sc
```

### 2. 部署 FastDFS

```bash
# 部署 Tracker
kubectl apply -f tracker-deployment.yaml
kubectl apply -f tracker-service.yaml

# 部署 Storage
kubectl apply -f storage-deployment.yaml
kubectl apply -f storage-service.yaml
```

### 3. 测试文件上传

```bash
# 进入 Storage 容器
kubectl exec -it $(kubectl get pods -l app=fastdfs-storage -o jsonpath='{.items[0].metadata.name}') -- /bin/bash

# 创建测试文件并上传
echo "Hello FastDFS" > test.txt && /usr/bin/fdfs_upload_file /etc/fdfs/client.conf test.txt
# 输出示例：group1/M00/00/00/ZGEoXmmC6yWADHFaAAAADjf5ba8278.txt

# 验证文件存储
find /data/fastdfs_data/ -name ZGEoXmmC6yWADHFaAAAADjf5ba8278.txt
```

### 4. 访问上传的文件

```bash
# 通过 Nginx 访问
curl fastdfs-storage:80/group1/M00/00/00/ZGEoXmmC6yWADHFaAAAADjf5ba8278.txt
```

## 配置说明

- **存储路径**：FastDFS 数据存储在 `/data/fastdfs_data` 目录，已挂载到持久化存储
- **服务端口**：
  - Tracker: 22122
  - Storage: 23000
  - Nginx: 80

## 构建镜像

```bash
cd build_image
docker build -t <your-registry>/fastdfs:ubuntu26arm .
```

## 注意事项

- 本配置适用于 ARM 架构的 Kubernetes 集群
- **存储路径**：持久化存储使用 hostPath，需要固定在某个机器上：
  - Tracker 存储：`/data/k8s/fastdfs/tracker`
  - Storage 存储：`/data/k8s/fastdfs/storage`
  - 确保这些路径在指定节点上存在且有正确的权限
- 生产环境建议使用更可靠的存储方案，如 NFS、Ceph 或云存储
- 可根据实际需求调整存储容量和副本数
