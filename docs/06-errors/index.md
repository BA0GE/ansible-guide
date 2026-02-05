# 06-错误代码索引 (Error Reference)

> **本章目标**: 面对红色的报错信息不再恐慌。这里汇总了 90% 的常见错误及其根本原因。

## 1. 调试三板斧

在查阅错误码之前，先掌握调试工具：

1.  **`-v` 参数**: 增加详细度。
    *   `-v`: 显示结果。
    *   `-vv`: 显示输入输出。
    *   `-vvv`: 显示 SSH 连接细节 (连接排查神器)。
2.  **`debug` 模块**: 打印变量值。
    ```yaml
    - name: Print variable
      debug:
        var: my_variable
    ```
3.  **Check Mode (`--check`)**: 模拟执行，不进行实际修改。

## 2. 连接类错误 (Connection Issues)

### `UNREACHABLE! => {"msg": "Failed to connect to the host via ssh..."}`
*   **原因 1**: 网络不通。
    *   *排查*: `ping <ip>`。
*   **原因 2**: SSH 服务未启动或端口不对。
    *   *排查*: `telnet <ip> 22`。
*   **原因 3**: 密钥认证失败 (`Permission denied (publickey)`).
    *   *解决*: 运行 `ssh-copy-id user@host` 或检查 `ansible_ssh_private_key_file` 路径。

### `Host key verification failed`
*   **原因**: 目标主机的指纹 (Fingerprint) 不在控制机的 `~/.ssh/known_hosts` 中，或者发生了变化（重装系统）。
*   **解决**:
    *   *临时*: `export ANSIBLE_HOST_KEY_CHECKING=False`
    *   *永久 (`ansible.cfg`)*: `host_key_checking = False`

## 3. 权限与执行类错误

### `Missing sudo password`
*   **原因**: Playbook 中开启了 `become: yes`，但 `sudo` 执行需要密码，且未提供。
*   **解决**: 运行命令时加 `-K` (即 `--ask-become-pass`)。

### ` /bin/sh: /usr/bin/python: No such file or directory`
*   **原因**: 目标机未安装 Python，或者 Python 路径不在标准位置（如 Python 3 安装在 `/usr/bin/python3`，但 Ansible 默认找 `python`）。
*   **解决**: 设置变量 `ansible_python_interpreter=/usr/bin/python3`。

## 4. 语法与逻辑错误

### `Syntax Error while loading YAML`
*   **原因**: 缩进错误，或者使用了 Tab 键。YAML 对缩进极其敏感。
*   **排查**: 使用 IDE (VS Code) 的 YAML 插件进行格式化。

### `The task includes an option with an undefined variable`
*   **原因**: 引用了不存在的变量 `{{ my_var }}`。
*   **解决**: 检查变量名拼写，或使用 `default` 过滤器: `{{ my_var | default('value') }}`。

## 5. 错误处理策略 (Error Handling)

有时候我们希望忽略错误继续执行：

*   **`ignore_errors: yes`**: 即使当前 Task 失败，继续执行下一个 Task。
*   **`failed_when`**: 自定义失败条件。
    ```yaml
    - command: /bin/my_script
      register: result
      failed_when: "'CRITICAL' in result.stdout"
    ```
*   **`changed_when`**: 自定义何时标记为 "Changed" (保持幂等性)。
    ```yaml
    - command: /bin/install_something
      register: result
      changed_when: "'Installed successfully' in result.stdout"
    ```
