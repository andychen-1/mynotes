
#### 推荐解决方案：导入自定义证书（安全优先）
假设您能从服务器管理员获取证书文件（.crt 或 .pem 格式，通常是服务器的根 CA 证书）。如果没有，浏览器访问 https://cgitlab.shandycloud.com，点击锁图标 > 证书 > 导出（保存为 .crt）。

**针对 Ubuntu/Debian 系统（您的 kevin-OMEN 主机）的步骤**：
1. **复制证书到 CA 目录**：
   ```
   sudo cp your-cert.crt /usr/local/share/ca-certificates/your-cert.crt
   ```
   （替换 `your-cert.crt` 为实际文件名；如果是 PEM 格式，确保以 `-----BEGIN CERTIFICATE-----` 开头。）

2. **更新系统 CA 信任列表**：
   ```
   sudo update-ca-certificates
   ```
   这会将证书添加到 `/etc/ssl/certs/` 并更新 OpenSSL 的信任链。

3. **重启 Git 或终端**（可选，但推荐），然后测试：
   ```
   cd ~/qt-workspace/lidarmapserverarm
   git config --global http.sslBackend "openssl"  # 确保使用 OpenSSL 后端
   git pull
   ```

- **验证**：运行 `openssl s_client -connect cgitlab.shandycloud.com:443 -CAfile /etc/ssl/certs/ca-certificates.crt` 检查是否返回 `Verify return code: 0 (ok)`。
- **如果证书链复杂**（有中间证书）：合并所有证书到一个 .crt 文件（cat root.crt intermediate.crt > combined.crt），然后导入。

#### 其他安全替代方案（无需导入证书）
1. **切换到 SSH**（强烈推荐，局域网内最可靠）：
   - 如之前所述，生成/添加 SSH 密钥到 GitLab 用户设置，避免 HTTPS 证书问题。
   - 更新 URL：`git remote set-url origin git@cgitlab.shandycloud.com:shandy/lidarmapserverarm.git`。
   - 测试：`ssh -T git@cgitlab.shandycloud.com`。

2. **使用 Git 的证书自定义**（仓库级）：
   ```
   git config http.sslCAInfo /path/to/your-cert.crt
   ```
   这指定 Git 只信任此证书，而不影响系统全局。
