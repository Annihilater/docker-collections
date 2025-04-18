# Ananta - SSH 批量管理工具

Ananta 是一个基于 Docker 的轻量级 SSH 批量管理工具，用于同时管理和操作多台服务器。它支持密钥认证和密码认证，可以按标签分组管理服务器，并提供灵活的输出控制选项。

## 特性

- 🔒 安全：以非 root 用户运行，密钥文件只读挂载
- 🏷️ 标签：支持服务器分组管理
- 🎨 美观：彩色输出，可自定义终端宽度
- 🚀 高效：批量执行命令，并发处理
- 📦 便携：基于 Docker，一键部署
- 🔑 认证：支持密钥和密码两种认证方式

## 快速开始

### 1. 拉取 Docker 镜像

```bash
docker pull icecodexi/ananta:latest
```

### 2. 准备环境

```bash
# 创建 SSH 密钥目录
mkdir -p "${HOME}/.ssh/"

# 设置正确的密钥文件权限
find "${HOME}/.ssh/" -type f -print0 | xargs -0 -r chmod 600

# 创建主机列表文件
touch "$(pwd)/hosts.csv"
```

### 3. 配置主机列表

在 hosts.csv 中配置要管理的服务器信息，格式如下：

```csv
# 格式：主机地址,SSH端口,用户名,认证信息(密钥路径或密码),标签(可选,多个用分号分隔)
192.168.1.100,22,root,~/.ssh/id_rsa,prod;web
192.168.1.101,22,admin,password123,prod;db
192.168.1.102,2222,ubuntu,~/.ssh/custom_key,staging
```

### 4. 运行容器

基本用法：

```bash
docker run --rm -it \
    --volume /etc/localtime:/etc/localtime:ro \
    --volume "${HOME}/.ssh/:/home/nonroot/.ssh/:ro" \
    --volume "$(pwd)/hosts.csv:/home/nonroot/hosts.csv:ro" \
    --cpu-shares 512 --memory 512M --memory-swap 512M \
    icecodexi/ananta:latest [命令选项]
```

## 命令选项

| 选项 | 说明 | 示例 |
|------|------|------|
| `-k, --default-key` | 指定默认 SSH 密钥 | `-k ~/.ssh/id_rsa` |
| `-t, --host-tags` | 按标签筛选主机 | `-t prod,staging` |
| `-n, --no-color` | 禁用彩色输出 | `-n` |
| `-s, --separate-output` | 分开显示每个主机的输出 | `-s` |
| `-e, --allow-empty-line` | 允许空行输出 | `-e` |
| `-c, --allow-cursor-control` | 允许光标控制 | `-c` |
| `-w, --terminal-width` | 设置终端宽度 | `-w 120` |
| `-v, --version` | 显示版本信息 | `-v` |

## 使用示例

### 使用密钥连接所有主机

```bash
docker run --rm -it \
    --volume "${HOME}/.ssh/:/home/nonroot/.ssh/:ro" \
    --volume "$(pwd)/hosts.csv:/home/nonroot/hosts.csv:ro" \
    icecodexi/ananta:latest -k ~/.ssh/id_rsa
```

### 连接特定标签的主机

```bash
docker run --rm -it \
    --volume "${HOME}/.ssh/:/home/nonroot/.ssh/:ro" \
    --volume "$(pwd)/hosts.csv:/home/nonroot/hosts.csv:ro" \
    icecodexi/ananta:latest -t prod
```

### 分开显示每个主机的输出

```bash
docker run --rm -it \
    --volume "${HOME}/.ssh/:/home/nonroot/.ssh/:ro" \
    --volume "$(pwd)/hosts.csv:/home/nonroot/hosts.csv:ro" \
    icecodexi/ananta:latest -s -k ~/.ssh/id_rsa
```

## 资源限制

默认的资源限制配置：

- CPU 份额：512
- 内存限制：512MB
- 交换空间：512MB

可以根据需要调整这些参数：

```bash
docker run --rm -it \
    --cpu-shares 1024 \
    --memory 1G \
    --memory-swap 1G \
    ...
```

## 安全建议

1. 始终使用密钥认证而不是密码认证
2. 定期更新 Docker 镜像以获取安全更新
3. 确保 SSH 密钥文件权限正确（600）
4. 使用只读挂载来保护敏感文件
5. 避免在生产环境中使用 root 用户运行容器

## 故障排除

1. 密钥权限问题

```bash
# 修复密钥权限
chmod 600 ~/.ssh/id_rsa*
```

2. 容器访问权限问题

```bash
# 如果需要以 root 用户运行
if [[ "$UID" -eq '0' ]]; then
    _run_as_root='--user root'
fi
docker run --rm -it ${_run_as_root} ...
```

## 贡献指南

欢迎提交 Issue 和 Pull Request！在提交之前，请确保：

1. 代码符合项目的编码规范
2. 添加了必要的测试用例
3. 更新了相关文档
4. 提交信息清晰明了

## 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件
