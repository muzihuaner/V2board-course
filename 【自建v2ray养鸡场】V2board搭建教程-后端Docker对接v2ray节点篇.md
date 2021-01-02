### 【自建v2ray养鸡场】V2board搭建教程-后端Docker对接v2ray节点篇

前面介绍了v2board的前端网站搭建教程，本篇接上文！关于v2board如何对接后端v2ray节点，通过docker快速对接v2ray后端节点，v2ray后端对接脚本程序：Aurora、v2ray-poseidon、sogo三种对接脚本。

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/v2boardjd.png)

其中Aurora由官方人员维护，属于v2board的亲儿子，需付费258 USDT才能使用（永久版）；v2ray-poseidon又叫波塞冬，社区版可以免费提供50个有效用户，商业版年付65 USDT；sogo社区版提供免费88个有效用户，商业版年付65 USDT。

三者最大差别是Aurora仅能在v2board面板使用，v2ray-poseidon和sogo都可以对接v2board、vnetpanel、sspanel三种面板.



#### 前端网站添加节点

1、打开v2board面板网站管理中心，找到权限组，创建一个组，然后再找到节点管理-添加节点；

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/jd.png)

2、我这里添加下最简单的TCP协议节点做示范，其它协议类似：名称标签随意，倍率是用户使用流量按多少倍算，权限组就是刚才添加的组，节点地址和端口填需要对接的后端服务器IP或域名、自定义端口，连接端口和服务端口一般情况请保持一致，传输协议TCP，然后提交确认；

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/%E5%9B%BE%E7%89%87-13.png)

3、选择系统配置找到服务端，设置通讯密钥（自定义16位数以上），社区版无授权文件，如有授权文件则填入进去；

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/%E5%9B%BE%E7%89%87-15.png)



#### 后端服务器对接节点

**（一）、v2ray-poseidon后端**

1、通过SSH连接上你的Linux后端节点服务器（需要性价比高的服务器在本站都有推荐），推荐使用CentOS7；安装内核加速，推荐使用bbr plus。先安装内核，选择2，重启后，开启加速，选择7，如需其它BBR加速脚本看本站提供的教程。

```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh"
chmod +x tcp.sh
./tcp.sh
```

2、同步时间为北京时间：一般不需要，保险起见，建议还是同步一下。

```
yum -y install ntpdate
timedatectl set-timezone Asia/Shanghai
ntpdate ntp1.aliyun.com
```

3、关闭防火墙：必须要做，否则大部分对接上节点但是连接都会无网络连接。

```
systemctl start supervisord
systemctl disable firewalld
systemctl stop firewalld
```

4、安装并启动 Docker/docker-compose。

```
curl -fsSL https://get.docker.com | bash
curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod a+x /usr/local/bin/docker-compose
rm -f `which dc` 
ln -s /usr/local/bin/docker-compose /usr/bin/dc 
 
systemctl start docker
service docker start
systemctl enable docker.service
systemctl status docker.service
```

5、从GitHub获取后端源码安装v2ray-poseidon。

```
yum install -y git
git clone https://github.com/ColetteContreras/v2ray-poseidon.git
```

6、修改配置文件，我这里对接的tcp协议，如果需要对接其它协议就进入到v2board目录下的其它协议文件夹。config.json文件需要修改三项。docker-compose.yml文件需要修改端口。

```
cd /root/v2ray-poseidon/docker/v2board/tcp
 
vi config.json
##"nodeId":6 // 面板里添加完节点后生成的自增ID
##"webapi": "https://zhujiget.com",// v2board 的域名信息或单独设置个专门对接后端不需要用户访问的域名
##"token": "zhujiget.com666666", // v2board 和 v2ray-poseidon 的通信密钥
 
vi docker-compose.yml
##'服务端'修改为前端面板设置的端口数字
```

7、赋予Docker权限，并且启动Docker：

```
chmod +x  /bin/dc
cd /root/v2ray-poseidon/docker/v2board/tcp
dc up -d
```

8、到此为止你已经全部设置好了。接下来我们可以查看一下日志看看有没有报错。

```
dc logs
```

9、回到前端v2board面板网站，可以看到刚才添加的节点状态显示蓝色，表示已对接成功，红色则对接失败检查故障，最后打开显隐开关连接节点测试。

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/%E5%9B%BE%E7%89%87-14.png)

**（二）、soga后端对接**

1、按照前面的添加好节点，把防火墙或时间同步下，这里就不加上了，输入下列命令安装soga；

```
bash <(curl -Ls https://blog.sprov.xyz/soga.sh)
或
bash <(curl -Ls https://raw.githubusercontent.com/sprov065/soga/master/install.sh)
```

2、安装好后命令行输入：vi /etc/soga/soga.conf 编辑以下几个地方（面板类型、面板域名、通信密钥、节点ID），其它根据自身需求配置；

```
type=v2board       ## 对接的面板类型，可选v2board/sspanel/vnetpanel
server_type=v2ray  ## 对接的节点类型，可选v2ray/trojan/SS
api=webapi         ## 对接的方式，可选webapi 或 db，表示 webapi 对接或数据库对接
 
##webapi 对接
webapi_url=https://zhujiget.com/  ## 面板域名地址，或自定义个专用后端对接不提供访问的域名
webapi_mukey=zhujigetcom666666    ## 面板设置的通讯密钥
 
##数据库对接
db_host=db.xxx.com  ## 数据库地址
db_port=3306  ## 数据库端口
db_name=name  ## 数据库名
db_user=root  ## 数据库用户名
db_password=asdasdasd  ## 数据库密码
 
node_id=1   ## 前端节点id
soga_key=  ## 授权密钥，社区版无需填写，最多支持88用户，商业版无限制
user_conn_limit=0  ## 限制用户连接数，0代表无限制，v2board 必填！！！
user_speed_limit=0   ## 用户限速，0代表无限制，单位 Mbps，v2board 必填！！！
check_interval=100   ## 同步前端用户、上报服务器信息等间隔时间（秒），近似值
force_close_ssl=false ## 设为true可强制关闭tls，即使前端开启tls，soga也不会开启tls，方便用户自行使用nginx、caddy等反代
forbidden_bit_torrent=true  ## 设为true可禁用bt下载
default_dns=8.8.8.8,1.1.1.1  ## 配置默认dns，可在此配置流媒体解锁的dns，以逗号分隔
```

3、编辑好自己需要的设置后保存退出，命令行输入soga，输入数字4启动soga，可输入7或8查看状态和日记，没意外的话面板已经亮灯了，自行测试是否能上网；