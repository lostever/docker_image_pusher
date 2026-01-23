# Aliyun Image Pusher

### 基于 GitHub Actions 的镜像同步工具 (Fork 优化版)

本项目是一个轻量级的镜像中转工具，旨在将 Docker Hub 的镜像自动同步至阿里云私有仓库（ACR），解决国内环境拉取镜像慢的问题。

## ✨ 本次 Fork 引入的优化 (Key Enhancements)

对比原版逻辑，本项目进行了以下核心改进：

1. **组织名隔离机制 (Namespace Isolation)**：
* **痛点**：原版在处理不同组织下的同名镜像（如 `apache/doris` 和 `hello/doris`）时，容易发生覆盖。
* **改进**：引入了组织名前缀化逻辑。所有镜像在同步时会自动映射为 `组织名_镜像名:标签` 的格式，确保不同来源的镜像互不冲突。


2. **官方镜像标准化**：
* 自动识别 Docker 官方镜像（无前缀镜像），统一归类到 `library_` 前缀下，使阿里云镜像列表更加规整。


3. **智能增量同步**：
* 集成 `skopeo` 探测功能。每次运行前会核对阿里云端是否已存在该镜像。**已存在的镜像将直接跳过**，大幅提升执行效率，节省 GitHub Actions 构建分钟数。


4. **超大镜像支持**：
* 通过 `maximize-build-space` 自动释放构建环境磁盘空间。即使是数 GB 的重型镜像（如 Doris, Oracle）也能平稳同步。



## 🚀 快速使用

### 1. 配置 Secrets

在 GitHub 仓库的 `Settings` -> `Secrets and variables` -> `Actions` 中添加：

* `ALIYUN_REGISTRY`: 阿里云仓库地址
* `ALIYUN_NAME_SPACE`: 命名空间
* `ALIYUN_REGISTRY_USER`: 阿里云镜像服务用户名
* `ALIYUN_REGISTRY_PASSWORD`: 阿里云镜像服务访问令牌 (临时密码)

### 2. 编辑镜像列表

修改 `images.txt` 文件，每行填入一个镜像地址：

```text
# 官方镜像会自动转为 library_mysql:8.0
mysql:8.0
# 自动转为 apache_doris:fe-2.1.11
apache/doris:fe-2.1.11

```

### 3. 执行同步

* **自动触发**：修改并提交 `images.txt` 后，工作流会自动运行。
* **手动触发**：在 `Actions` 页面手动点击 `Run workflow`。

## 📝 命名转换规则参考

| 源镜像 (Docker Hub) | 阿里云仓库名 (ACR) |
| --- | --- |
| `mysql:latest` | `library_mysql:latest` |
| `apache/doris:2.1` | `apache_doris:2.1` |
| `bitnami/git:latest` | `bitnami_git:latest` |
