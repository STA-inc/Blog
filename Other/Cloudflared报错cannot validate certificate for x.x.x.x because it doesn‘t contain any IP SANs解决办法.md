前言：Cloudflared是美国CDN服务商Cloudflare提供的一个零信任服务，常用于企业搭建零信任服务框架。当然，也可以用于隐藏C2服务器，C2服务器对外提供的服务和监听端口经Cloudflared映射后能够比较好的隐藏C2的真实IP，想要登陆C2服务器也只能持有Cloudflared凭证（这里表述不太准确），通过Cloudflared代理程序登陆，避免被网络空间资产测绘引擎扫描到。
我的config.yml是这样写的：

```bash
tunnel: my_tunnel
credentials-file: /etc/cloudflared/<ID>.json
originRequest:
     noTLSVerify: true
ingress:
# host1
 - hostname: host1.domain.com
   service: https://192.168.100.254
 # Catch-all rule doesn’t actually use any of the config
 - service: http_status:404
```
在这里有一个神奇的问题，originRequest需要写在映射的服务后面，不能写在开头作为全局设置。所以应该这样写：

```bash
tunnel: my_tunnel
credentials-file: /etc/cloudflared/<ID>.json
ingress:
# host1
 - hostname: host1.domain.com
   service: https://192.168.100.254
   originRequest:
     noTLSVerify: true
 # Catch-all rule doesn’t actually use any of the config
 - service: http_status:404
```
对https服务关闭TLS验证就可以了。
出现这个报错的可以按照上面的思路自查。
