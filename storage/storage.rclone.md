<a name="aXoIw"></a>
# Rclone
参考文档：

- [https://rclone.org/](https://rclone.org/)
- [https://github.com/rclone/rclone](https://github.com/rclone/rclone)

Rclone 是一款支持超过 40 种云存储的命令行工具，包括（NFS、S3 对象存储）。命令与 Unix 下等效相似，支持 Shell 管道。跨平台支持操作系统 Windows、macOS、linux 和 FreeBSD。同时能够实现挂载云存储到本地盘。<br />开发语言：golang 和前端界面使用 React
<a name="iZNZI"></a>
## 安装
Windows 下载安装包即可，只包含一个可执行文件。<br />Linux / macOS / BSD 用户请这样安装：
```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```
<a name="ecwZm"></a>
## 命令
```bash
rclone config         # 命令行交互式编辑配置
rclone ls             # 和*nix大部分命令一致：ls, cp, rmdir
rclone mount remote:path /path/to/mount
```
注意 Windows 不支持 Daemon 模式后台运行，如果命令行窗口退出，那么诸如挂载的行为也会终止。可以借用下面这个方法写一个 Visual Basic 脚本来后台运行。
```vbnet
set shell = wscript.createObject("wscript.shell")
iReturn = shell.run("rlone.exe mount remote:path /path/to/mount", 0, TRUE)
```
<a name="DdDnX"></a>
## GUI
它会先从远程下载一个 React 的前端包（界面不是特别好看），再启动一个 Web UI 界面。如果需要指定密码登录、端口和 TLS 证书等等，请参考 [GUI 文档](https://rclone.org/gui/)。一般只用来临时启动界面编辑配置。
```bash
rclone rcd --rc-web-gui

--rc-web-gui-update           # 更新 UI 包
--rc-web-gui-force-update     # 强制更新 UI 包
--rc-web-gui-no-open-browser  # 关闭默认打开浏览器
```
