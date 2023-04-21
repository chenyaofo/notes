## 服务器上如何使用 Clash？

 - 第一步：订阅转换

 在[订阅转换网站](https://acl4ssr-sub.github.io/)中，选择`ACL4SSR Online Mini MultiMode.ini 精简版 自动测速、故障转移、负载均衡(与Github同步)`配置，`猫熊`后端，生成订阅短链接。

  - 第二步：下载订阅文件

```
wget -O config.yaml https://suo.yt/xxxxxxx
```

 - 第三步：启动 Clash (基于Docker)

```
docker run -d --name=clash -v "$PWD/config.yaml:/root/.config/clash/config.yaml" -p "7890:7890" -p "9090:9090" --restart=unless-stopped dreamacro/clash
```

 - 第四步：在 Web 界面中配置 Clash

打开 http://yacd.haishan.me ，配置 Clash， 然后就可以愉快使用了。如果游览器遇到 CORS 错误，请参照 https://kebingzao.com/2021/10/11/chrome-private-cors-error/