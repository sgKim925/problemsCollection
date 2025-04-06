首先：卸载vscode和cursor，原因是vscode的ssh插件可能与cursor有冲突。同时注意在服务器中删掉vscode和cursor的残留目录。同时建议在VSCode中删除`Perference: User Setting JSON`的残留配置。

```shell
rm -rf ~/.vscode-server
rm -rf ~/.cursor-server
```

重装cursor，并配置ssh连接服务器。

> 以下内容是针对连接ssh服务器时卡在`正在使用scp将vscode服务器上传到主机`

由于cursor的About->commit id 并非如同VSCode一样的commit ID，因此无法像配置VSCode服务器时那样通过写commit ID从远端手动下载vscode 服务器。

点开View->Output查看输出日志。在任务中选择`Remote ssh`查看ssh运行日志。大概会报出

```shell
816be4e591ad:trigger_server_download
> artifact==cli-alpine-x64==
> destFolder==/home/dell/.cursor-server==
> destFolder2==/vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz==
> 816be4e591ad:trigger_server_download_end
> Waiting for client to transfer server archive...
> Waiting for /home/dell/.cursor-server/vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz.done and vscode-server.tar.gz to exist
>  
[11:32:07.877] Got request to download on client for {"artifact":"cli-alpine-x64","destPath":"/home/dell/.cursor-server/vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz"}
[11:32:07.878] server download URL: https://cursor.blob.core.windows.net/remote-releases/1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870/cli-alpine-x64.tar.gz
[11:32:07.878] Downloading VS Code server locally...
[11:32:07.922] Downloaded VS Code server to /var/folders/l8/blg071_1679bt8n8j30klg9w0000gn/T/450b8a0a-fb7f-4a69-a09f-7f264dd26f86
[11:32:07.923] Renamed VS Code server to /var/folders/l8/blg071_1679bt8n8j30klg9w0000gn/T/vscode_server_1743910327922/vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz
[11:32:07.924] PATH: /Users/seonggunkim/WorkSpace/SoftWare/miniconda3/bin:/Users/seonggunkim/WorkSpace/SoftWare/miniconda3/condabin:/opt/homebrew/opt/php@8.2/sbin:/opt/homebrew/opt/php@8.2/bin:/opt/homebrew/sbin:/opt/homebrew/bin:/Library/Frameworks/Python.framework/Versions/3.12/bin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin
[11:32:07.924] Checking ssh with "ssh -V"
[11:32:07.927] > OpenSSH_9.8p1, LibreSSL 3.3.6

[11:32:07.928] Testing scp with "scp"
[11:32:07.931] scp exited with code: 1
[11:32:07.931] Got stderr from scp: usage: scp [-346ABCOpqRrsTv] [-c cipher] [-D sftp_server_path] [-F ssh_config]
           [-i identity_file] [-J destination] [-l limit] [-o ssh_option]
           [-P port] [-S program] [-X sftp_option] source ... target
[11:32:07.931] Copying file to remote with scp -o ConnectTimeout=15 'vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz' 'vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz.done' ':dbzdhnjh42':'/home/dell/.cursor-server'
[11:32:07.932] Using cwd: file:///var/folders/l8/blg071_1679bt8n8j30klg9w0000gn/T/vscode_server_1743910327922
[11:32:08.063] > :dbzdhnjh42:/home/dell/.cursor-server: No such file or directory
[11:32:08.306] "Copy server to host" terminal command done
```

找到下载server的URL，访问后下载。`https://cursor.blob.core.windows.net/remote-releases/1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870/cli-alpine-x64.tar.gz`。注：每个主机可能不一样。

观察发现它将下载内容放在了`/var/folders/l8/blg071_1679bt8n8j30klg9w0000gn/T/450b8a0a-fb7f-4a69-a09f-7f264dd26f86`并重命名为`/var/folders/l8/blg071_1679bt8n8j30klg9w0000gn/T/vscode_server_1743910327922/vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz`。 

查看`/var/folders/l8/blg071_1679bt8n8j30klg9w0000gn/T/vscode_server_1743910327922`发现该文件下存在两个文件`vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz`和`vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz.done`。一个是下载的压缩包，另一个是标记下载好的done文件。现在将这个文件夹上传到服务器。

```
scp -P port /var/folders/l8/blg071_1679bt8n8j30klg9w0000gn/T/vscode_server_1743910327922 用户名@Host:~/.cursor-server
```

在服务器中走到`~/.cursor-server`中解压

```shell
tar -zxvf vscode-cli-1d623c4cc1d3bb6e0fe4f1d5434b47b958b05870.tar.gz
```

发现解压出了一个`cursor`可执行文件。然后退出服务器，在cursor中重新ssh远程连接即可成功连接。