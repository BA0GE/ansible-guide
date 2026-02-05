# 04-分步实战教程 (Step-by-Step Tutorial)

> **本章目标**: 从零开始，部署一个生产级的高可用 Nginx Web 服务器。我们将从最简单的 Ad-Hoc 命令进化到模块化的 Roles 结构。

## 场景描述 (Scenario)
*   **目标**: 在一组服务器上安装 Nginx，推送自定义 `index.html`，并确保服务开机自启。
*   **环境**:
    *   控制节点: 本机
    *   受控节点: `server-a`, `server-b` (假设 IP 为 192.168.1.10, 192.168.1.11)

## Step 1: 建立主机清单 (Inventory)

创建 `inventory.ini` 文件。我们使用 INI 格式，因为它简单直观。

```ini
[webservers]
server-a ansible_host=192.168.1.10
server-b ansible_host=192.168.1.11

[webservers:vars]
# 定义该组的通用变量
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**验证**:
```bash
ansible webservers -i inventory.ini -m ping
```

## Step 2: 编写第一个 Playbook

创建 `site.yml`。这是 Ansible 的核心编排文件。

```yaml
---
- name: Deploy Nginx Web Server
  hosts: webservers
  become: yes  # 需要 root 权限安装软件

  vars:
    http_port: 80
    server_name: "My Awesome Site"

  tasks:
    - name: Ensure Nginx is installed
      apt:
        name: nginx
        state: present
        update_cache: yes
      # 解释: state=present 保证软件已安装 (幂等性)

    - name: Deploy custom index.html
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
        mode: '0644'
      notify: Restart Nginx
      # 解释: 只有当文件内容发生变化时，才会触发 Handler

    - name: Ensure Nginx is running and enabled
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
      # 解释: Handlers 只有在被 notify 时才会执行，且在 Play 最后统一执行
```

## Step 3: 模板化配置 (Jinja2 Templates)

创建 `templates/index.html.j2`。使用 Jinja2 语法动态注入变量。

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ server_name }}</title>
</head>
<body>
    <h1>Welcome to {{ ansible_hostname }}</h1>
    <p>This server is managed by Ansible.</p>
</body>
</html>
```

*   `{{ server_name }}`: 来自 Playbook 中定义的变量。
*   `{{ ansible_hostname }}`: 来自 Ansible 自动收集的 Facts (系统主机名)。

## Step 4: 执行与验证

运行 Playbook：

```bash
ansible-playbook -i inventory.ini site.yml
```

**输出解读**:
*   `PLAY [Deploy Nginx Web Server]`: 剧本开始。
*   `TASK [Gathering Facts]`: 自动收集目标机信息。
*   `TASK [Ensure Nginx is installed]`:
    *   `changed`: 第一次运行，安装了软件。
    *   `ok`: 第二次运行，检测到已安装，跳过。
*   `RUNNING HANDLER [Restart Nginx]`: 仅在配置文件修改后触发。

## 常见错误排查 (Troubleshooting)

1.  **YAML 缩进错误**:
    *   *现象*: `Syntax Error while loading YAML`.
    *   *解决*: 确保使用 2 空格缩进，严禁使用 Tab 键。
2.  **权限不足**:
    *   *现象*: `apt: Permission denied`.
    *   *解决*: 检查 `become: yes` 是否已添加。
