### 【自建V2Ray养鸡场】通过宝塔面板搭建V2board完整运营教程—前端面板篇

V2board是一个开源且易于管理V2Ray程序的可视化用户管理系统，集成了web网站前端+后端多个v2ray节点+多用户管理+支付+邮件系统，支持TCP、WS+CDN、WS+TLS等协议，前端面板简洁易用。这篇教程主要记录下搭建使用V2board的方法，主要分为：面板搭建、节点对接、支付对接、邮件对接等教程。

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/v2boardqd.png)

V2board的前端面板是一个web网站，我这里用的是宝塔面板（[bt.cn](https://www.bt.cn/?invite_code=MV9mZ3RzcHA=)）部署环境和网站，官方采用的是aaPanel面板（宝塔国际版）部署，英文环境差别不大，机器配置要求最低1核512M内存，建议选择1G内存及以上服务器，选择debian等消耗资源比较少的Linux系统，在本站有各种优惠性价比高的服务器推荐。

搭建注意：该教程仅面向对电脑/服务器知识有一点基础（指会用键盘鼠标、认字的）、善用搜索引擎的用户，不适用于纯小白、理解能力不够、死脑筋、眼睛不知在哪的用户。

**宝塔面板搭建网站**

0、通过SSH工具连接上服务器，更新下软件源和安装开发者工具包（可选）：

```
yum update -y  ## Debian系统把yum改为apt-get
yum -y groupinstall "Development Tools"  ## Debian系统把yum改为apt-get
```

1、然后在命令行输入下列命令进行安装宝塔面板，宝塔搭建WEB环境的详细安装教程在前面有文章，这里简略说下：

```
##CentOS系统安装命令：
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
##Debian安装命令：
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh
```

2、安装好宝塔面板后在浏览器输入提供的地址进入面板，选择LNMP安装以下web环境，大于等于以下环境版本即可：

- Nginx 1.17
- MySQL 5.6
- PHP 7.3

3、安装web环境后打开PHP版本的设置，安装redis扩展（可选安装ionCube、fileinfo、opache、sg11）；

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/%E5%9B%BE%E7%89%87-8.png)

4、解除被禁用的函数：putenv ， proc_open ， pcntl_alarm ， pcntl_signal

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/%E5%9B%BE%E7%89%87-9.png)

5、点击宝塔左侧的网站，选择添加站点，输入网站域名或IP地址，域名需解析指向到该服务器IP，站点根目录的文件名不要有点号之类的特殊符号，否则后面可能无法执行队列服务；

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/%E5%9B%BE%E7%89%87-10.png)

**安装V2board**

1、通过SSH工具连接上服务器，cd到网站根目录，执行下列命令，注意替换文件名：

```
cd /www/wwwroot/站点文件名
chattr -i .user.ini
rm -rf .htaccess 404.html index.html
```

2、执行命令从 Github 克隆到当前目录（站点根目录），并把v2board文件夹的文件全部剪切到站点根目录文件夹（自行在宝塔站点文件目录查看剪切粘贴）：

```
git clone https://github.com/v2board/v2board.git
```

3、执行命令下载 composer.phar 到当前目录（站点根目录）：

```
wget https://getcomposer.org/download/1.9.0/composer.phar
```

4、执行命令进行包安装：

```
php composer.phar install
```

安装过程中报错或者无法继续安装的请分配 swap，如何分配 swap 请参考本站swap虚拟内存分配教程，建议使用内存1G以上服务器安装。

5、安装v2board，根据提示输入数据库地址和用户名及默认的管理员账户密码：

```
php artisan v2board:install
```

6、避免后面打开网站出现500错误提示，给目录重新赋予权限，站点根目录执行下列命令，如执行后还显示500错误，可进一步尝试重启web环境和检查redis是否运行：

```
chown -R www:www *
```

7、后期v2board面板升级更新，命令行在站点根目录执行下列命令：

```
sh update.sh
```

**配置网站目录和伪静态规则**

1、回到宝塔面板，左侧网站-设置-网站目录，取消防跨站攻击，目录设置为/public 并保存。

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/image-14.png)

2、继续选择旁边的伪静态，输入以下规则并保存：

```
location /downloads {
}
 
location / {
    try_files $uri $uri/ /index.php$is_args$query_string;
}
 
location ~ .*\.(js|css)?$
{
    expires 1h;
    error_log off;
    access_log /dev/null;
}
```

**配置定时任务和添加守护队列**

1、在宝塔面板左侧选择计划任务，任务类型shell、任务名称v2board，周期每一分钟1次，脚本内容输入：

```
php /www/wwwroot/站点文件名/artisan schedule:run
```

2、v2board的邮件系统和支付自动开通都依赖队列服务，在宝塔面板左侧软件商店搜索PM2管理器进行守护队列，找到PM2 Manager进行安装，然后添加项目，项目根目录选择站点根目录，启动文件名：pm2.yaml，项目名称：v2board，然后确定添加：

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/image-15.png)

这时候在浏览器输入域名即可访问v2board面板前端网站了，域名后面加/admin则进入管理员面板，同时也可以在宝塔面板的网站设置开启SSL证书访问。

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/v2gfw.png)

