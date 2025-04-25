---
{"dg-publish":true,"permalink":"/开发/Alist/Alist + 夸克 + Vidhub 实现多端影音自由/"}
---


# [Alist](https://alist.nn.ci/zh/)
一个支持多种存储的文件列表程序，使用 Gin 和 Solidjs。

alist支持多种安装方式，如果有 nas 或者国内服务器，推荐使用 docker 安装（快捷好管理）。

# 思路
1. 使用 docker 方式安装 alist
2. 启动 alist（作为服务器），挂载夸克网盘
3. 启动 vidhub，使用 webdav 方式连接 alist 服务器，即可读取夸克网盘文件内容。
4. 视频资源从 tg 频道或者其他途径保存到夸克。
5. 以上思路适用于任何网盘（阿里云盘限速，不推荐）。
---

# 步骤
## 1. 安装 alist

有服务器条件的情况下，推荐使用 docker-compose.yml 来安装，方便保管和修改命令。命令如下，可以保存为 .yml 文件来运行。

```docker
services:
  alist:
    image: 'xhofe/alist:latest'
    container_name: alist
    command: /opt/alist/alist server # 威联通 nas 需要该命令，其他服务器可以删掉
    volumes:
      - '/share/CACHEDEV1_DATA/Container/alist:/opt/alist/data'
    ports:
      - '5244:5244'
    environment:
      - PUID=0
      - PGID=0
      - UMASK=022
      - TZ=Asia/Shanghai
    restart: unless-stopped
```

### 踩坑点
#### docker 镜像被墙
由于 dockerhub 被墙，国内无法访问，因此我们需要迂回一下，有几种方式：
1. 让 nas 走软路由出国
2. 使用某台电脑充当代理服务器出国（亲测无效，但建议优先尝试）。
3. 修改 nas 镜像地址（亲测无效，但建议优先尝试）。
4. 手动 pull 镜像后，再上传到 nas 进行安装。
我们最终使用了第4个方案，比较土，但简单有效。


#### docker 安装环境的架构与本机系统的架构不同
我们在进行 docker pull 操作时，可能使用的是 macOS，架构是 arm64。而实际安装 docker 的 nas 环境是 amd64，所以在 pull 时需要指定架构：
```docker
docker pull --platform=linux/amd64 xhofe/alist:latest
```


#### docker 命令调整
由于我们使用威联通 Container Station 来安装，所以基于官方文档模板，我们要额外做一些命令调整：
1. Container Station 不需要 version 字段，因此 .yml 中不要写 version。
2. Container Station 需要 command 或者 entrypoint 字段，其含义是：
	- 容器启动后，将会执行 `/opt/alist/alist` 这个可执行文件。
	- `server` 是传递给 `/opt/alist/alist` 这个程序的参数。根据 Alist 的文档，这个参数通常用于启动 Alist 的 Web 服务。



## 2. 配置 Alist，挂载夸克

参考(教程)[https://zh.okaapps.com/blog/6544cbf9948009206c562b9f]。

官网教程有所缺失，我们还需要进行 Alist 用户配置：
1. Alist 用户需要拥有 webdav 读取权限，否则 vidhub 播放器无法正常连接 Alist。
操作：打开 Alist 后台 -> 管理 -> 用户，编辑用户，勾选“Webdav 读取”。
建议单独新建1个只读用户，勾选“Webdav 读取”，与 admin 区分开。

2. vidhub 添加 alist 时，如果遇到无法正常添加的错误提示，检查以下几点：
- 协议（一般是 http）
- alist 服务器地址
- 端口（按照 docker 命令里写的是5244）
- alist 用户名和密码
- 路径：/dav尝试写一下，提示选填，但实测要填

## 3. 公网访问 Alist 后台

如果 nas 和路由器均具备 UPNP 功能，那只需要在 nas 侧添加1个服务即可。否则可能需要在路由器额外配置路由转发。然后就可以通过域名:端口在公网愉快管理 Alist 后台了。

---

## THE END

如果是苹果用户，记得打开 vidhub 的 iCloud 同步功能，即可多端同步相同文件源配置（速度巨快）。
现在在 vidhub 的文件源 tab 下就可以看到夸克网盘的文件了。
开始享受自由的观影体验吧！

![](https://picgo-1256490109.cos.ap-guangzhou.myqcloud.com/20250425171037.png)

---


# FAQ

问题：将镜像打包成 tar 报错
```
 docker save xhofe/alist:latest > alist-amd64.tar

Error response from daemon: unable to create manifests file: NotFound: content digest sha256:25aacca3037e968c2d8e9361c1743e0045c8c6db10c48d707a3695fb5351d649: not found
```

答：尝试先运行容器，再导出整个容器
```
docker run -d --name alist-container xhofe/alist:latest
docker export alist-container > alist-container.tar
```
---

问题：我有多个网盘，怎么添加？
答：在 Alist 后台中继续添加“存储”即可。