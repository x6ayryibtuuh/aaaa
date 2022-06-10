## 注意

 1. **请勿滥用，Heroku账号封禁风险自负。**
 2. Heroku的文件系统是临时性的，每24小时强制重启一次后会恢复到部署时状态。不适合长期下载和共享文件用途。
 3. Aria2和qBittorrent配置文件默认限速5MB/s。
 4. 免费Heroku dyno半小时无Web访问会休眠，可以使用uptimerobot、hetrixtools等免费VPS/网站监测服务定时http ping，保持持续运行。

[概述](#概述)

[部署方式](#部署方式)

[变量设置](#变量设置)  

[初次使用](#初次使用)  

[更多用法和注意事项](#更多用法和注意事项)  

## 概述

本项目集成了yt-dlp、gallery-dl、Aria2+Rclone+qBittorrent+WebUI、pyLoad下载管理器、Rclone联动自动上传功能、、Rclone远程存储文件列表和Webdav服务、Filebrowser轻量网盘、OliveTin网页执行shell命令、ttyd Web终端、Xray Vmess协议。

 1. 联动上传功能只需要准备rclone.conf配置文件, 其他一切配置都预备齐全。
 2. Rclone以daemon方式运行，可在WebUI上手动传输文件和实时监测传输情况。
 3. Aria2、qBittorrent和Rclone可以接入其它host上运行的AriaNg/RcloneNg等前端面板和flexget/Radarr/Sonarr等应用。
 4. 自动备份相关配置文件到Cloudflare Workers KV，dyno重启时尝试恢复，实现了配置文件持久化。
 5. 可以从OliveTin网页端执行yt-dlp、gallery-dl和Rclone指令。
 6. ttyd网页终端，可命令行执行yt-dlp下载工具和其它命令。
 7. log目录下有每个服务独立日志。

## 部署方式

  **请勿使用本仓库直接部署**  

  **Heroku修复安全漏洞中，目前无法通过网页从私有库部署**  

 1. [设置Cloudflare Workers KV服务](https://github.com/wy580477/PaaS-Related/blob/main/SET_CLOUDFLARE_KV_chs.md)
 2. 点击本仓库右上角Fork，再点击Create Fork。
 3. 在Fork出来的仓库页面上点击Setting，勾选Template repository。
 4. 然后点击Code返回之前的页面，点Setting下面新出现的按钮Use this template，起个随机名字创建新库。
 5. 比如你的Github用户名是bobby，新库名称是green。浏览器登陆heroku后，访问<https://dashboard.heroku.com/new?template=https://github.com/bobby/green> 即可部署。

 **Heroku部署变量设置**

对部署时可设定的变量做如下说明。
| 变量| 说明 |
| :--- | :--- |
| `GLOBAL_USER` | 用户名，适用于除qBittorrent外所有需要输入用户名的Web服务 |
| `GLOBAL_PASSWORD` | 务必修改为强密码，同样适用于除qBittorrent外所有需要输入密码的Web服务，同时也是Aria2 RPC密钥。 |
| `GLOBAL_LANGUAGE` | 设置导航页、qBittorrent和Filebrowser界面语言，chs为中文 |
| `GLOBAL_PORTAL_PATH` | 导航页路径和所有Web服务的基础URL，务必设置为不常见路径。不能为“/"和空值，结尾不能加“/" |
| `TZ` | 时区，Asia/Shanghai为中国时区 |
| `CLOUDFLARE_WORKERS_HOST` | Cloudflare Workers 服务域名 |
| `CLOUDFLARE_WORKERS_KEY` | Cloudflare Workers 服务密钥 |
| `VMESS_UUID` | Vmess协议UUID，务必修改，建议使用UUID工具生成 |

### 初次使用

1. 比如你的heroku域名是bobby.herokuapp.com，导航页路径是/portal，访问bobby.herokuapp.com/portal 即可到达导航页。
2. 点击AriaNg，这时会弹出认证失败警告，按下图把之前部署时设置的密码填入RPC密钥即可。
       <img src="https://user-images.githubusercontent.com/98247050/163184113-d0f09e78-01f9-4d4a-87b9-f4a9c1218253.png"  width="700"/>
3. 点击qBittorrent或者VueTorrent，输入默认用户名admin和默认密码adminadmin登陆。然后更改用户名和密码，务必设置为强密码。
4. 通过Filebrowse将rclone.conf文件上传到config目录，可以通过编辑script.conf文件更改Rclone自动上传设置。
5. yt-dlp和gallery-dl下载工具可以通过ttyd在网页终端执行。   
    内置快捷指令：  
    dlpr：使用yt-dlp下载视频到videos文件夹下，下载完成后发送任务到rclone。 
    gdlr：使用gallery-dl下载文件到gallery_dl_downloads文件夹下，下载完成后发送任务到rclone。 

## [Cloudflare Workers反代绕过Heroku非信用卡认证账号每月550小时限制](https://github.com/wy580477/PaaS-Related/blob/main/CF_Workers_Reverse_Proxy_chs.md)

### 更多用法和注意事项

 1. 如果网页访问APP出现故障，按下shift+F5强制刷新，如果还不行，从浏览器中清除app对应的heroku域名缓存和cookie。
 2. pyLoad已知Bug：
    - 登陆后重定向到http，解决方法：关闭当前pyLoad页面，重新打开。
    - 解压后不能删除原文件，解决方法：Settings--Plugins--ExtractArchive，将"Move to trash instead delete"项设置为off。
 3. Heroku部署后，将下列内容添加到rclone.conf文件，可以将Heroku本地存储作为Rclone的远程存储，便于在Rclone WebUI上手动上传。

       ```
       [local]
       type = alias
       remote = /mnt/data
       ```
       
 4. 对于不支持qBittorrent自定义路径的应用比如Radarr, 部署前在content目录下的Caddyfile文件中找到下列内容，去除每行开头的注释符号“#”:

       ```
       handle /api* {
              reverse_proxy * localhost:61804
       }
       ```

 5. Aria2 JSON-RPC 路径为： \$\{GLOBAL_PORTAL_PATH\}/jsonrpc   
    Aria2 XML-RPC 路径为： \$\{GLOBAL_PORTAL_PATH\}/rpc
 6. 无法通过Rclone Web前端建立需要网页认证的存储配置。
 7. Vmess协议AlterID为0，可用Vmess WS 80端口或者Vmess WS tls 443端口连接。Xray设置可以通过content/xray.yaml文件修改。Heroku国内直连可能需要使用Cloudflare或其它方式中转。
