# 02-Prerequisites

## 1. Control Node

*   **Hardware Requirements**:
    *   CPU: 2 Cores+ (Concurrent connections consume CPU)
    *   RAM: 4GB+ (Ansible itself is lightweight, but forking many processes consumes memory)
*   **OS Support**:
    *   ✅ Linux (RHEL, CentOS, Ubuntu, Debian)
    *   ✅ macOS
    *   ❌ Windows (Not supported natively as a Control Node; use **WSL2**)
*   **Software Dependencies**:
    *   Python 3.8+ (Core runtime)
    *   `pip` (Package manager)

## 2. Managed Nodes

*   **Basic Requirements**:
    *   **Python**: Python 2.7+ or Python 3.5+. Ansible modules need to run Python code on the target machine.
        *   *Troubleshooting*: If the target is a minimal container (like Alpine), Python is missing by default. Use the `raw` module to install Python first.
    *   **SSH Server**: Ensure `sshd` service is on and listening (default port 22).
    *   **SFTP**: Defaults to SFTP for file transfer. If `sshd_config` disables SFTP, configure Ansible to use `scp` mode.

## 3. Connectivity Check

```bash
# Ad-hoc command test
# -m ping: Calls the ping module (Not ICMP ping, but Python-level connectivity test)
# -u root: Connect as root user
ansible all -m ping -u root
```

### Common Errors (Troubleshooting)
*   `"msg": "module configuration failed"` -> Target machine missing Python environment.
*   `"msg": "Permission denied (publickey)"` -> SSH public key not correctly distributed (`ssh-copy-id`).
