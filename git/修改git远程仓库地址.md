### 步骤指导：更新 Git 远程仓库地址并提交代码

假设你已经从旧私有仓库克隆了项目（本地代码完整），仓库迁移到新地址后，只需更新远程 origin 的 URL，然后正常提交即可。分支保持相同，不会影响。以下是详细步骤（在项目根目录下执行命令）：

1. **检查当前远程仓库地址**（可选，确认旧地址）：
   ```
   git remote -v
   ```
   输出类似：
   ```
   origin  git@old-repo.com:username/repo.git (fetch)
   origin  git@old-repo.com:username/repo.git (push)
   ```

2. **更新远程仓库 URL 到新地址**：
   - 如果是 SSH 方式（常见私有仓库）：
     ```
     git remote set-url origin git@new-repo.com:username/repo.git
     ```
   - 如果是 HTTPS 方式：
     ```
     git remote set-url origin https://new-repo.com/username/repo.git
     ```
   - **注意**：替换 `new-repo.com:username/repo.git` 为实际新地址。确保你有新仓库的访问权限（SSH 密钥或 Token）。

3. **验证更新**：
   ```
   git remote -v
   ```
   现在应该显示新地址。

4. **拉取最新代码**（可选，如果新仓库有其他变更）：
   ```
   git fetch origin
   git pull origin <branch-name>  # 如 main 或 master
   ```

5. **提交你的代码**：
   - 添加变更：
     ```
     git add .  # 或 git add <specific-file>
     ```
   - 提交：
     ```
     git commit -m "你的提交消息"
     ```
   - 推送到新仓库（指定分支，确保相同）：
     ```
     git push origin <branch-name>  # 如 main
     ```

### 常见问题排查
- **权限错误**（如 "Permission denied"）：检查 SSH 密钥或 HTTPS Token 是否配置正确。新仓库可能需要重新添加凭证。
- **分支不存在**：如果新仓库分支名稍有不同，用 `git branch -a` 检查，然后用 `git push origin <local-branch>:<remote-branch>` 推送并创建。
- **如果本地是浅克隆**（`git clone --depth=1`）：建议先 `git fetch --unshallow` 转为完整历史。
- **私有仓库**：推送前确保新地址已添加你的账号为协作者。
