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


## 使用Clash Verge给机场再套个Cloudflare Warp

```
mixed-port: 7890
allow-lan: true
bind-address: '*'
mode: rule
log-level: info
external-controller: '127.0.0.1:9090'
dns:
    enable: true
    ipv6: false
    default-nameserver: [223.5.5.5, 119.29.29.29]
    nameserver: [223.5.5.5, 223.6.6.6]

proxy-providers:
  JiChang:   #名称，默认即可
    type: http
    url: "xxx"  ###此处必填，url里面填入你的机场订阅链接，一般订阅链接加密了，需要转换为直链(https://acl4ssr-sub.github.io/，进阶模式，选择输出为Node List)
    interval: 3600
    path: ./JiChang.yaml   #机场订阅文件下载后的文件名及文件地址，默认即可
    health-check:
      enable: true
      interval: 600
      url: https://www.gstatic.com/generate_204

proxies:
  - name: WARP
    type: wireguard
    server: engage.cloudflareclient.com
    port: 2408
    ip: 172.16.0.2
    ipv6: 2606:4700:110:85a2:2aae:7478:ef9f:4c94
    private-key: #请到Telegram中的Warp Plus Bot获取
    public-key: #请到Telegram中的Warp Plus Bot获取
    udp: true
    reserved: [0,0,0]
    remote-dns-resolve: true  
    dns: [ 1.1.1.1, 8.8.8.8 ]
    dialer-proxy: "前置节点"

proxy-groups:
  # 这个名字为前置节点的策略组可以选择是用自建节点还是机场节点作为前置节点

  - name: 节点选择
    type: select
    proxies:
      - WARP        #给机场套warp之后的节点     
      - 机场自动选择    
      - 机场手动选择

  - name: 前置节点
    type: select
    proxies:  
      - 机场自动选择    
      - 机场手动选择

  - name: 机场自动选择
    type: url-test #选出延迟最低的机场节点
    use:
      - JiChang    #proxy-providers中的名字，默认即可
    url: "http://www.gstatic.com/generate_204"
    interval: 300
    tolerance: 50

  - name: 机场手动选择
    type: select #手动选择
    use:
      - JiChang

rules:
    - GEOIP,LAN,DIRECT
    - GEOIP,lan,DIRECT,no-resolve
    # 默认规则
    - MATCH,节点选择
```

## 换源工具

工具地址：https://github.com/RubyMetric/chsrc

下载安装：curl -L https://gitee.com/RubyMetric/chsrc/releases/download/pre/chsrc-x64-linux -o chsrc; chmod +x ./chsrc

支持的镜像站、软件可通过`./chsrc l`查看，目前对我来说支持的源包括：pip, python, pypi, pdm, npm, node, go, golang；ubuntu, debian；docker, dockerhub, conda。

下面命令可以直接设置`pip`源：`./chsrc set pip`，下面命令仅测速但不设置`pip`源：`./chsrc cesu pip`。

如果不像测速，下面命令可以直接设置预定义中最快的`pip`源：`./chsrc set pip first`。

如果想手动换源，先运行`./chsrc list pip`选择找到源的代号（code），然后`./chsrc set pip <code>`即可。