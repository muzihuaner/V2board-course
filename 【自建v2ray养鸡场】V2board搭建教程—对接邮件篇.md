### 【自建v2ray养鸡场】V2board搭建教程—对接邮件篇

V2board的邮件可用于注册验证、重置密码、订阅或群发邮件通知用户，设置不恰当会导致邮件无法发出，这篇文章主要介绍V2board如何正确设置邮件信息，对接阿里云的域名邮件服务。

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/v2boardyj.png)

1、如果自己有域名，可以到[阿里云](https://www.aliyun.com/?source=5176.11533457&userCode=w9ksrgbo&type=copy)开通自己的域名邮件推送服务，也可以使用其它邮件服务，有的邮箱的发信容易进垃圾箱，自己选择即可，之后在面板后台系统设置-邮件就能可视化操作，且优先级大于.env配置文件：

邮件参数注意点：SMTP账号如果直接填设置的帐号可能会导致发送邮件收不到，可以尝试设置跟发件地址一样，加密方式大写SSL也可能会导致发送邮件收不到，可以自己尝试改为小写ssl，其次部分邮箱设置465端口可能会导致超时的情况，可以切换为587端口尝试！

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/image.png)

2、设置好后重启宝塔面板的pm2，同时刷新配置缓存，可以在命令行站点根目录执行：php artisan config:cache

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/image-1.png)

3、之后注册下账户或重置下密码测试发信是否正常，发信模板可以在v2board管理中心的系统配置-邮件配置选择；

![img](https://gitee.com/muzihuaner/huangeimages/raw/master/%E5%9B%BE%E7%89%87-21.png)

如上能收到邮件表示配置成功，如还无法收到邮件请检查配置和PM2队列服务启动情况（可尝试重启队列服务），或进入数据库查看邮件日记报错信息。