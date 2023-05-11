## 服务器上如何使用 Clash？

 - 第一步：订阅转换

 在订阅转换网站 https://id9.cc 中，选择`Basic(仅GEOIP CN+Final)`配置，`猫熊`后端，生成订阅短链接。

  - 第二步：下载订阅文件

```
wget -O config.yaml https://sub.cm/xxxxxxx
```

 - 第三步：启动 Clash (基于Docker)

```
docker run -d --name=clash -v "$PWD/config.yaml:/root/.config/clash/config.yaml" -p "7890:7890" -p "9090:9090" --restart=unless-stopped dreamacro/clash
```

 - 第四步：在 Web 界面中配置 Clash

打开 http://yacd.haishan.me ，配置 Clash（在Proxies配置组中选择合适的节点就好了，GLOBAL配置组忽略）， 然后就可以愉快使用了。如果游览器遇到 CORS 错误，请参照 https://kebingzao.com/2021/10/11/chrome-private-cors-error 进行设置

 - （可选）一键刷新脚本

```
wget -O ~/clash/config.yaml https://sub.cm/xxxxxxx && docker restart clash
```

## 服务器上proxy的便捷使用

```
proxyip=$(cat /etc/resolv.conf | grep nameserver | awk '{ print $2 }')
port=7890
alias p="http_proxy=http://${proxyip}:${port} https_proxy=http://${proxyip}:${port} all_proxy=http://${proxyip}:${port}"
alias p5="http_proxy=socks5://${proxyip}:${port} https_proxy=socks5://${proxyip}:${port} all_proxy=socks5://${proxyip}:${port}"
alias pq="proxychains4 -q"
```

