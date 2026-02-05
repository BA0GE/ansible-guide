# 02-环境准备 (Prerequisites)

## 1. 控制节点 (Control Node)

*   **硬件要求**:
    *   CPU: 2 核以上 (并发连接会消耗 CPU 资源)
    *   RAM: 4GB 以上 (Ansible 自身很轻量，但大量 Fork 进程会消耗内存)
*   **操作系统支持**:
    *   ✅ Linux (RHEL, CentOS, Ubuntu, Debian)
    *   ✅ macOS
    *   ❌ Windows (原生不支持作为控制节点，需使用 **WSL2**)
*   **软件依赖**:
    *   Python 3.8+ (核心运行时)
    *   `pip` (包管理器)

## 2. 受控节点 (Managed Nodes)

*   **基本要求**:
    *   **Python**: Python 2.7+ 或 Python 3.5+。Ansible 的模块需要在目标机上运行 Python 代码。
        *   *常见错误排查 (Troubleshooting)*: 如果目标机是极简容器 (如 Alpine)，默认没有 Python，需使用 `raw` 模块先安装 Python。
    *   **SSH Server**: 确保 `sshd` 服务开启并在监听（默认端口 22）。
    *   **SFTP**: 默认依赖 SFTP 传输文件。如果 `sshd_config` 禁用了 SFTP，需配置 Ansible 使用 `scp` 模式。

## 3. 连通性自检 (Connectivity Check)

```bash
# Ad-hoc 临时命令测试
# -m ping: 调用 ping 模块 (注意：这不是 ICMP ping，而是 Python 级别的连通性测试)
# -u root: 指定以 root 用户连接
ansible all -m ping -u root
```

### 常见错误排查 (Troubleshooting)
*   `"msg": "module configuration failed"` -> 目标机缺少 Python 环境。
*   `"msg": "Permission denied (publickey)"` -> SSH 公钥未正确分发 (`ssh-copy-id`)。
