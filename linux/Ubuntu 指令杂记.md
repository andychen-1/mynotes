
## 文件系统

```bash
# 查看文件夹总大小
du -hs .
# 列出目录下每个文件夹的大小（包含所有子目录）
du -h .
# 列出当前目录下文件夹的大小（不包含下一级目录）
du -h --max-depth=1 .
# 列出当前目录文件夹与文件的大小
du -ah --max-depth=1 .
```


### Docker

```bash
# 允许在 Docker 容器中运行的图形应用程序能够访问宿主机的 X 服务器
xhost +local:docker
```