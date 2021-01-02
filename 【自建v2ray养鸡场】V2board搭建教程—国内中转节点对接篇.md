### 【自建v2ray养鸡场】V2board搭建教程—国内中转节点对接篇

由于当前的网络环境，很多场主的节点都是通过国内服务器中转到节点服务器上去的方式，这样既能提升被墙的概率又能改善落地鸡的网络线路，通过前面的教程我们已经搭建好了一套完整v2board面板和简单的TCP节点对接，这里再来个通过v2board面板设置中转节点的教程。

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/v2boardzz.png)

1、v2board的中转节点比较简单，首先我们需要中转服务器上设置好转发，本站提供了好几种转发教程（任选1种），参考如下：

- [GOST隧道一键脚本使用教程，利用GOST搭建中转安全隧道](https://zhujiget.com/3818.html)
- [Ehco中转隧道简单使用教程，通过ehco搭建wss安全加密隧道](https://zhujiget.com/4977.html)
- [Brook中转脚本使用教程，支持DDNS域名转发/可转发TCP/UDP流量](https://zhujiget.com/3547.html)
- [CentOS 7开启firewall流量转发功能，简单配置服务器TCP/UDP中转加速教程](https://zhujiget.com/3451.html)
- [WebSocket（WS）协议隧道 - VNet-Tunnel搭建详细教程，利用隧道中转加速SSR科学上网](https://zhujiget.com/3374.html)
- [利用iptables端口转发脚本，实现VPS中转网络加速的教程](https://zhujiget.com/3044.html)

2、转发好后打开我们的v2board面板，添加中转节点，落地节点请提前根据前面的教程配置好：

![img](https://zhujiget.com/wp-content/uploads/2020/06/image.png)

3、至此中转设置完成，打开显隐开关，再连接查看节点是否可用，如有多台中转鸡也可类似设置，可多台中转至同一个落地节点（父节点）。