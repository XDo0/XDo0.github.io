---
title: ubuntu20.04服务器安装μtorrent和qbitorrent
tag: [linux]
key: 2023-03-24-ubuntu_torrent
---

# 安装qbittorrent

安装命令如下[^5]：

```bash
sudo add-apt-repository ppa:qbittorrent-team/qbittorrent-stable
sudo apt install qbittorrent-nox
vim /etc/systemd/system/qbittorrent-nox.service
```

可以加入以下配置，来更换端口为8085

```plaintext
[Unit]
Description=qBittorrent client
After=network.target

[Service]
ExecStart=/usr/bin/qbittorrent-nox --webui-port=8085
Restart=always

[Install]
WantedBy=multi-user.target
```

然后输入以下命令启动服务，并设置开机启动：

```bash
sudo service qbittorrent-nox start
sudo systemctl enable qbittorrent-nox
```

最后防火墙开放qbittorrent的web UI的端口8085

浏览器访问`你的ip:8085`即可看到web UI

语言可以在`Advance`修改

默认账号`admin`，密码是`adminadmin`，可以在`设置`-`Web UI`-`验证`修改

下载的文件保存在`//Downloads/` ，可以在设置中修改

qbittorrent下载需要开放个端口，在`选项`-`连接`-`监听端口`中可以看到，记得防火墙开放这个端口[^4]。
上传依然需要开放端口，默认是9000，在`设置`-`高级`-`qbittorrent相关`-`内置tracker端口`中看到
{:.warning}

# μtorrent

μtorrent最新版仅支持Ubuntu13.04，但是ubuntu20.04依然可以用[^1]。

注意libssl是必须安装的，而且其他博客写的下载链接已经404了，需要按最新的[^2]。

如果μtorrent web UI默认的8080端口运行了其他服务，可以更换端口，见7-8行[^3]。

记得防火墙需要放行这个端口。

```bash
wget http://download-hr.utorrent.com/track/beta/endpoint/utserver/os/linux-x64-ubuntu-13-04 -O utserver.tar.gz
sudo tar xvf utserver.tar.gz -C /opt/
sudo apt install libssl-dev
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl1.0/libssl1.0.0_1.0.2n-1ubuntu5_amd64.deb
sudo apt-get install ./libssl1.0.0_1.0.2n-1ubuntu5_amd64.deb
sudo ln -s /opt/utorrent-server-alpha-v3_3/utserver /usr/bin/utserver
echo "ut_webui_port: 8085" >  /opt/utorrent-server-alpha-v3_3/utserver.conf
utserver -settingspath /opt/utorrent-server-alpha-v3_3/ -configfile /opt/utorrent-server-alpha-v3_3/utserver.conf -daemon
```

[^1]: [How to Install uTorrent in Ubuntu 20.04](https://www.linuxbabe.com/ubuntu/install-utorrent-ubuntu-20-04)
[^2]: [utserver: error while loading shared libraries: libssl.so.1.0.0: cannot open shared object file: No such file or directory](https://askubuntu.com/a/1368551)
[^3]: [How to change port number utorrent in ubuntu?](https://askubuntu.com/questions/202429/how-to-change-port-number-utorrent-in-ubuntu#:~:text=How%20to%20Change%20uTorrent%20Port%20A%20super%20easy,to%20Settings%20and%20select%20webUI%20option.%20See%20More.)
[^4]: [Linux 服务器安装 qBittorrent - Mr. Ma's Blog (misterma.com)](https://www.misterma.com/archives/902/)
[^5]: [Install qBittorrent-nox on Ubuntu 20.04 | Lindevs](https://lindevs.com/install-qbittorrent-nox-on-ubuntu)

然后可以在浏览器输入`你的ip:8085/gui`打开μtorrent的web版，第一次登录的账户名是`admin`，密码是空密码，在设置中`Web UI`中更改。
