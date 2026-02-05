# 05-高级调优 (Advanced Optimization)

> **本章目标**: 当你的管理节点从 10 台增加到 1000 台时，默认配置会慢如蜗牛。本章将介绍让 Ansible 飞起来的核心调优技巧。

## 1. SSH Pipelining (管道加速)

这是提升 Ansible 性能**最有效**的单一设置。

*   **原理**:
    *   *默认模式*: SSH 连接 -> SFTP 上传脚本 -> SSH 连接 -> 执行脚本 -> 删除脚本 (多次网络 IO)。
    *   *Pipelining*: 建立一个 SSH 会话，直接通过管道 (`|`) 将 Python 脚本输入到远程解释器执行。减少了 SFTP 连接和文件写入开销。
*   **配置 (`ansible.cfg`)**:
    ```ini
    [ssh_connection]
    pipelining = True
    ```
*   **前提条件**: 目标机的 `/etc/sudoers` 配置中不能有 `requiretty` (现代 Linux 发行版默认通常没有)。

## 2. 并发控制 (Forks)

Ansible 默认一次只操作 5 台主机 (`forks = 5`)。对于大集群，这太保守了。

*   **优化**: 根据控制节点的 CPU 核心数，适当调大。
    ```ini
    [defaults]
    forks = 50
    ```
*   **注意**: 内存消耗会随之增加。

## 3. 关闭 Facts 收集 (Gathering Facts)

每次 Playbook 运行前，Ansible 都会像查户口一样收集目标机的 OS、IP、磁盘等信息。这非常耗时。

*   **策略 1: 显式关闭**: 如果你的 Playbook 不依赖 `ansible_os_family` 等变量，直接关掉。
    ```yaml
    - hosts: webservers
      gather_facts: no
    ```
*   **策略 2: Facts 缓存 (Smart Gathering)**: 收集一次，缓存 24 小时 (Redis/JSON)。
    ```ini
    [defaults]
    gathering = smart
    fact_caching = jsonfile
    fact_caching_connection = /tmp/facts_cache
    fact_caching_timeout = 86400
    ```

## 4. SSH ControlPersist (长连接)

复用 SSH 连接，避免每次执行 Task 都重新进行 TCP 握手和密钥交换。

*   **配置**:
    ```ini
    [ssh_connection]
    ssh_args = -o ControlMaster=auto -o ControlPersist=60s
    ```
*   **效果**: 第一次连接慢，后续命令秒级响应。

## 5. 针对大规模集群的 Mitogen

**Mitogen** 是一个第三方 Python 库，完全重写了 Ansible 的通信层。
*   **效果**: CPU 占用降低 50%，执行速度提升 2-7 倍。
*   **适用场景**: 拥有数千台服务器的超大规模环境。

## 调优效果对比表

| 优化手段 | 适用场景 | 预期提升 | 风险 |
| :--- | :--- | :--- | :--- |
| **Pipelining** | 通用 | ⭐⭐⭐⭐⭐ | 需检查 sudoers |
| **Forks** | 主机数 > 10 | ⭐⭐⭐⭐ | 内存消耗增加 |
| **Fact Caching** | 频繁运行 | ⭐⭐⭐ | 数据可能过期 |
| **ControlPersist** | 连续 Tasks 多 | ⭐⭐⭐⭐ | 无 |
