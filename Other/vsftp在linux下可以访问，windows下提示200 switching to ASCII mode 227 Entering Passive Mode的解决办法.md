vsftp这个东西我真的是太恨它了，一天到晚一大堆问题，烦不胜烦，今天遇到这个问题情景是云服务器，开启了21/22端口，Ubuntu下访问正常，windows10下通过资源管理器访问FTP服务器提示：  

200 switching to ASCII mode  

227 Entering Passive Mode  

百度了半天说是因为windows使用了被动模式连接云服务器上的vsftp，  

因此将其改为主动模式。  

1.设置IE浏览器，然后点击Internet选项，  

点击高级  

点击将“使用被动FTP（用于防火墙和DSL调制解调器的兼容）”选项去掉  

点击确定  

2.此方法对FileZilla无效，原因不明。  

2019.09.02更新  

今天再登录，同样的服务器，同样的客户端，又出幺蛾子，提示200 port command successful. consider using pasv 425 failed to establish connection，将客户端防火墙关闭、服务器端口全部开放均无效。故卸载vsftp，改用putty自带的pscp向服务器传输文件  

scp命令格式如下：  

pscp -i 私钥路径 文件路径 服务器用户名@IP地址:文件存放路径  

-------end--------
