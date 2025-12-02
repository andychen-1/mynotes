### Ubuntu 22.04 设置 SSH 证书认证（公钥认证）指南

SSH 证书认证（也称公钥认证）比密码认证更安全，能避免暴力破解。以下是完整步骤，假设您在 Ubuntu 22.04 客户端生成密钥，并连接到远程服务器（也为 Ubuntu 或类似 Linux）。整个过程无需 root 权限（除服务器配置时）。

#### 前提条件
- 确保 SSH 已安装：`sudo apt update && sudo apt install openssh-client`（客户端）；服务器端：`sudo apt install openssh-server` 并启动 `sudo systemctl enable --now ssh`。
- 远程服务器需允许公钥认证（默认开启）。

#### 步骤 1: 生成 SSH 密钥对（客户端）
在您的 Ubuntu 22.04 机器上执行：
```
ssh-keygen -t ed25519 -C "your_email@example.com"
```
- `-t ed25519`：推荐算法（更安全、更快；若不支持，用 `-t rsa -b 4096`）。
- 提示时：
  - 文件路径：默认 `~/.ssh/id_ed25519`（私钥）和 `~/.ssh/id_ed25519.pub`（公钥）。
  - Passphrase：设置密码短语（可选，但推荐保护私钥）。
- 生成后，验证：`ls ~/.ssh/` 应看到密钥文件。

#### 步骤 2: 将公钥复制到远程服务器
使用 `ssh-copy-id` 工具（最简单）：
```
ssh-copy-id user@remote-host
```
- `user@remote-host`：替换为实际用户名和 IP/域名（如 `ubuntu@192.168.1.100`）。
- 首次连接需输入服务器密码（这是最后一次用密码）。
- 工具会自动将 `~/.ssh/id_ed25519.pub` 内容追加到服务器的 `~/.ssh/authorized_keys`。

**手动方式**（如果 `ssh-copy-id` 不可用）：
1. 查看公钥：`cat ~/.ssh/id_ed25519.pub`（复制输出）。
2. 登录服务器：`ssh user@remote-host`。
3. 创建目录：`mkdir -p ~/.ssh && chmod 700 ~/.ssh`。
4. 编辑授权文件：`echo "your-public-key-content" >> ~/.ssh/authorized_keys`。
5. 设置权限：`chmod 600 ~/.ssh/authorized_keys`。

#### 步骤 3: 配置 SSH 客户端（可选，增强安全性）
编辑 `~/.ssh/config`（若无，创建）：
```
Host remote-host  # 替换为您的主机别名或 IP
    HostName remote-host
    User user
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes  # 只用指定密钥
    PubkeyAuthentication yes
    PasswordAuthentication no  # 禁用密码认证（可选）
```
- 保存后，连接：`ssh remote-host`（无需密码，直接用密钥）。

#### 步骤 4: 配置 SSH 服务器（可选，禁用密码认证）
在远程服务器上编辑 `/etc/ssh/sshd_config`（需 sudo）：
```
sudo nano /etc/ssh/sshd_config
```
- 找到并修改：
  ```
  PubkeyAuthentication yes
  AuthorizedKeysFile .ssh/authorized_keys
  PasswordAuthentication no  # 禁用密码（推荐，但先测试）
  ```
- 保存，重载：`sudo systemctl reload ssh`。
- 测试：从客户端 `ssh user@remote-host` 应成功，无需密码。

#### 常见问题排查
- **Permission denied (publickey)**：检查服务器 `~/.ssh/authorized_keys` 权限（600），目录 700；SELinux/AppArmor 若启用，检查日志 `sudo journalctl -u ssh`。
- **密钥不匹配**：确保公钥完整复制（无多余空格）。
- **多密钥**：用 `ssh-add ~/.ssh/id_ed25519` 添加到 agent：`eval "$(ssh-agent -s)"`。
- **防火墙**：服务器确保 22 端口开放：`sudo ufw allow ssh`。
- **日志检查**：客户端 `-v` 选项：`ssh -v user@remote-host`；服务器 `/var/log/auth.log`。

#### 安全提示
- 备份私钥：`~/.ssh/id_ed25519` 勿泄露。
- 使用 passphrase 保护密钥。
- 定期轮换密钥：删除旧公钥从 `authorized_keys`。
- 若为 Git 等工具，配置 `~/.ssh/config` 中的 `Host` 匹配仓库 URL。

完成测试：`ssh user@remote-host "echo 'Success!'"`。


### Ubuntu 22.04 检查 SSH 服务端是否开放访问指南

检查 SSH 服务端（sshd）是否已开放访问，主要涉及服务状态、端口监听、防火墙配置和连接测试。以下是逐步方法（假设您在服务器上执行命令，或从客户端测试）。这些步骤适用于 Ubuntu 22.04，默认 SSH 端口为 22。

#### 步骤 1: 检查 SSH 服务是否运行（服务器端）
在服务器终端执行：
```
sudo systemctl status ssh
```
- **预期输出**：显示 `Active: active (running)` 表示服务正常运行。
- 如果未运行，启动：`sudo systemctl start ssh` 并启用开机自启：`sudo systemctl enable ssh`。
- 更多详情：`sudo journalctl -u ssh -f`（实时日志）。

#### 步骤 2: 检查端口是否监听（服务器端）
确认 SSH 监听在 22 端口（或自定义端口）：
```
sudo ss -tuln | grep :22
```
- **预期输出**：类似 `tcp LISTEN 0 128 0.0.0.0:22 0.0.0.0:*` 表示监听所有 IP（开放访问）。
- 替代命令（若 ss 不可用）：`sudo netstat -tuln | grep :22`（需安装 net-tools：`sudo apt install net-tools`）。
- 如果无输出，检查 `/etc/ssh/sshd_config` 中的 `Port 22` 并重载：`sudo systemctl reload ssh`。

#### 步骤 3: 检查防火墙是否允许 SSH（服务器端）
Ubuntu 默认用 UFW（Uncomplicated Firewall）：
```
sudo ufw status verbose
```
- **预期输出**：包含 `22/tcp ALLOW Anywhere` 或 `22 ALLOW Anywhere` 表示允许。
- 如果未允许，添加规则：`sudo ufw allow ssh`（或 `sudo ufw allow 22/tcp`），然后 `sudo ufw reload`。
- 如果用 iptables：`sudo iptables -L | grep ssh` 或 `sudo iptables -L INPUT -n | grep 22`。
- 检查 SELinux/AppArmor（若启用）：`sudo aa-status`（AppArmor），确保 ssh 未被阻塞。

#### 步骤 4: 从客户端测试连接（推荐，模拟外部访问）
在另一台机器（客户端）上测试：
```
ssh -v user@server-ip-or-hostname
```
- `-v`：详细模式，显示连接过程。
- **预期**：提示输入密码或直接登录（若公钥已设），表示开放。
- 常见错误：
  - `Connection refused`：服务未运行或端口未监听。
  - `Permission denied`：认证问题（密码/密钥）。
  - `No route to host`：网络/防火墙阻塞。
- 测试端口：`telnet server-ip 22` 或 `nc -zv server-ip 22`（预期：`Connected to server-ip`）。

#### 步骤 5: 检查访问日志（服务器端，排查问题）
```
sudo tail -f /var/log/auth.log | grep sshd
```
- 实时监控登录尝试。成功登录显示 `Accepted`，失败显示 `Failed` 或 `Invalid user`。
- 过滤特定 IP：`sudo grep "your-client-ip" /var/log/auth.log`。

#### 常见问题排查
- **局域网/云服务器**：确保安全组（AWS/EC2）或云防火墙允许 22 端口入站。
- **自定义端口**：替换所有 `22` 为实际端口（如 2222），并在 `sshd_config` 中指定。
- **IPv6**：检查 `ss -tuln | grep :22` 是否有 `[::]:22`。
- **如果完全无法连接**：运行 `sudo sshd -t` 测试配置语法错误。
