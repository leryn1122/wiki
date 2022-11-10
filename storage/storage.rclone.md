<a name="aXoIw"></a>
# Rclone
参考文档：

- [https://rclone.org/](https://rclone.org/)
- [https://github.com/rclone/rclone](https://github.com/rclone/rclone)
- [https://winfsp.dev/rel/](https://winfsp.dev/rel/)

Rclone 是一款支持超过 40 种云存储的命令行工具，包括（NFS、S3 对象存储）。命令与 Unix 下相似，支持 Shell 管道。跨平台支持操作系统 Windows、macOS、linux 和 FreeBSD。同时能够实现挂载云存储到本地盘。<br />开发语言：golang 和前端界面使用 React
<a name="iZNZI"></a>
## 安装
<a name="ERr7R"></a>
### Rclone
Windows 下载安装包即可，只包含一个可执行文件。<br />Linux / macOS / BSD 用户请这样安装：
```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```
<a name="ZRYmj"></a>
### WinFsp (Windows)
仅安装 Rclone 本身并不能支持各种复杂的文件系统协议，Windows 下需要安装 WinFsp（Windows File System Proxy），它来实现各个协议的对接。下载[安装包](https://winfsp.dev/rel/)安装即可。
<a name="eHbGd"></a>
### FUSE（Linux）
Linux 下安装文件协议对应的 FUSE 即可。
```vbnet
sudo apt install -y nfs-common s3fs
```
<a name="I649Z"></a>
## GUI
它会先从远程下载一个 React 的前端包（界面不是特别好看），再启动一个 Web UI 界面。如果需要指定密码登录、端口和 TLS 证书等等，请参考 [GUI 文档](https://rclone.org/gui/)。一般只用来临时启动界面编辑配置。
```bash
rclone rcd --rc-web-gui

--rc-web-gui-update           # 更新 UI 包
--rc-web-gui-force-update     # 强制更新 UI 包
--rc-web-gui-no-open-browser  # 关闭默认打开浏览器
```
<a name="ecwZm"></a>
## 命令
```bash
rclone config         # 命令行交互式编辑配置
rclone ls             # 和*nix大部分命令一致：ls, cp, rmdir
rclone mount remote:path /path/to/mount
```
除了`rclone config` 和 GUI，也可以直接在 `C:\Users\用户名\AppData\Roaming\rclone`下创建配置文件：
```toml
[aliyun-oss]
type = s3
access_key_id = admin
endpoint = https://oss.leryn.top/
secret_access_key = xxxxx
```
注意 Windows 不支持 Daemon 模式后台运行，如果命令行窗口退出，那么诸如挂载的行为也会终止。可以借用下面这个方法写一个 Visual Basic 脚本来后台运行。例如将阿里云下的 web 对象桶挂载成为本地的 Y 盘：
```basic
set shell = wscript.createObject("WScript.Shell")
iReturn = shell.run("rclone.exe mount aliyun-oss:/web/ Y: --no-check-certificate --allow-other --allow-non-empty", 0, TRUE)
```
Win + R 运行 `shell:startup` 打开启动目录，将这个 `.vbs` 脚本放到该目录下即可开机自启动挂载盘。以下命令杀 Windows 进程：
```powershell
tasklist | findstr rclone
taskkill /t /f /im 进程号
```
