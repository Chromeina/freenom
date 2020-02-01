# freenom：freenom域名自动续期

[![Build Status](https://img.shields.io/badge/build-passed-brightgreen?style=for-the-badge)](https://scrutinizer-ci.com/g/luolongfei/freenom/build-status/master)
[![Php Version](https://img.shields.io/badge/php-%3E=7.1-brightgreen.svg?style=for-the-badge)](https://secure.php.net/)
[![Scrutinizer Code Quality](https://img.shields.io/badge/scrutinizer-9.07-brightgreen?style=for-the-badge)](https://scrutinizer-ci.com/g/luolongfei/freenom/?branch=master)
[![MIT License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=for-the-badge)](https://github.com/luolongfei/freenom/blob/master/LICENSE)

### 缘起
众所周知，Freenom是地球上唯一一个提供免费顶级域名的商家，不过需要每年续期，每次续期最多一年。由于我申请了一堆域名，而且不是同一时段申请的，
所以每次续期都觉得折腾，于是就写了这个自动续期的脚本。

### 效果
![邮件示例](https://s2.ax1x.com/2020/01/31/139Rrd.png "邮件内容")

无论是续期成败或者脚本执行出错，都会收到的程序发出的邮件。如果是续期成败相关的邮件，邮件会包括未续期域名的到期天数等内容。
邮件参考了微信发送的注销公众号的邮件样式。

### 事前准备
- 发信邮箱：为了方便理解又称机器人邮箱，用于发送通知邮件。目前支持`Gmail`、`QQ邮箱`以及`163邮箱`，程序会自动判断发信邮箱类型并使用合适的配置。推荐使用`Gmail`。
- 收信邮箱：用于接收机器人发出的通知邮件。推荐使用`QQ邮箱`，`QQ邮箱`唯一的好处只是收到邮件会在`QQ`弹出消息。
- VPS：随便一台服务器都行，系统推荐`Centos7`，另外PHP版本需在`php7.1`及以上。
- 没有了

### 配置发信邮箱
下面分别介绍`Gmail`、`QQ邮箱`以及`163邮箱`的配置，你只用看自己需要的部分。注意，`QQ邮箱`与`163邮箱`均使用账户加授权码的方式登录，
`谷歌邮箱`使用账户加密码的方式登录，请知悉。另外还想吐槽一下，国产邮箱你得花一毛钱给邮箱提供方发一条短信才能拿到授权码。
#### 设置Gmail
1、在`设置>转发和POP/IMAP`中，勾选
- 对所有邮件启用 POP 
- 启用 IMAP

![gmail配置01](https://s2.ax1x.com/2020/01/31/13tKsg.png "gmail配置01")

然后保存更改。

2、允许不够安全的应用

登录谷歌邮箱后，访问[谷歌权限设置界面](https://myaccount.google.com/u/2/lesssecureapps?pli=1&pageId=none)，启用允许不够安全的应用。

![gmail配置02](https://s2.ax1x.com/2020/01/31/1392KH.png "gmail配置02")

另外，若遇到提示
> 不允许访问账户

登录谷歌邮箱后，去[gmail的这个界面](https://accounts.google.com/b/0/DisplayUnlockCaptcha)点击允许。这种情况较为少见。

#### 设置QQ邮箱
在`设置>账户>POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务`下，开启`POP3/SMTP服务`

![qq邮箱配置01](https://s2.ax1x.com/2020/01/31/13cIKA.png "qq邮箱配置01")

此时坑爹的QQ邮箱会要求你用手机发送一条短信给腾讯，发送完了点一下`我已发送`

![qq邮箱配置02](https://s2.ax1x.com/2020/01/31/13c4vd.png "qq邮箱配置02")

然后你就能看到你的邮箱授权码了，使用邮箱账户加授权码即可登录，记下授权码

![qq邮箱配置03](https://s2.ax1x.com/2020/01/31/13cTbt.png "qq邮箱配置03")

![qq邮箱配置04](https://s2.ax1x.com/2020/01/31/13coDI.png "qq邮箱配置04")

#### 设置163邮箱
在`设置>POP3/SMTP/IMAP`下，开启`POP3/SMTP服务`和`IMAP/SMTP服务`并保存

![163邮箱配置01](https://s2.ax1x.com/2020/01/31/13WKZn.png "163邮箱配置01")

![163邮箱配置02](https://s2.ax1x.com/2020/01/31/13WQI0.png "163邮箱配置02")

现在点击侧边栏的`客户端授权密码`，并获取授权码，你看到画面可能和我不一样，因为我已经获取了授权码，所以只有`重置授权码`按钮，这里自己根据网站提示申请获取授权码，网易和腾讯一样恶心，需要你用手机给它发一条短信才能拿到授权码

![163邮箱配置03](https://s2.ax1x.com/2020/01/31/13WMaq.png "163邮箱配置03")

*邮箱设置到此就完成了，下面可以愉快的配置本程序了*:)

### 配置脚本
所有操作均在Centos7系统下进行，其它Linux发行版大同小异
#### 获取源码
```bash
$ mkdir -p /data/wwwroot/freenom
$ cd /data/wwwroot/freenom

# clone本仓库源码
$ git clone https://github.com/luolongfei/freenom.git ./
```

#### 配置过程
```bash
# 复制配置文件模板
$ cp .env.example .env

# 编辑配置文件
$ vim .env

# .env文件里每个项目都有详细的说明，这里不再赘述，简言之，你需要把里面所有项都改成你自己的。需要注意的是多账户配置的格式：
# e.g. MULTIPLE_ACCOUNTS='<账户1>@<密码1>|<账户2>@<密码2>|<账户3>@<密码3>'
# 当然，若你只有单个账户，只配置FREENOM_USERNAME和FREENOM_PASSWORD就够了，单账户和多账户的配置会被合并在一起读取并去重。

# 编辑完成后，按“Esc”回到命令模式，输入“:wq”回车即保存并退出，不会用vim编辑器的问下谷歌大爷:)
```

#### 添加计划任务
#### 安装crontabs以及cronie
```bash
$ yum -y install cronie crontabs

# 验证crond是否安装及启动
$ yum list cronie && systemctl status crond

# 验证crontab是否安装
$ yum list crontabs $$ which crontab && crontab -l
```

##### 打开任务表单，并编辑
```bash
$ crontab -e

# 任务内容如下
# 此任务的含义是在每天早上9点执行/data/wwwroot/freenom/路径下的run文件
# 注意将/data/wwwroot/freenom/替换为你自己run文件所在路径
00 09 * * * cd /data/wwwroot/freenom/ && php run >/dev/null 2>&1
```

##### 重启crond守护进程
```bash
$ systemctl restart crond
```
若要检查`计划任务`是否正常，你也可以添加一个几分钟后执行的临时的`计划任务`，看任务是否执行。

*至此，所有的配置都已经完成，下面我们验证一下整个流程是否走通*:)

#### 验证
你可以先将`config.php`中的`noticeFreq`的值改为1（即每次执行都发邮件通知），然后执行
```bash
$ cd /data/wwwroot/freenom/ && php run
```
不出意外的话，你将收到一封关于域名情况的邮件。

遇到任何问题或bug欢迎提[issues](https://github.com/luolongfei/freenom/issues)，如果freenom改变算法导致此项目失效，
请提[issues](https://github.com/luolongfei/freenom/issues)告知，我会及时修复，本项目长期维护。欢迎star~

### 捐赠 Donate

#### PayPal: [https://www.paypal.me/mybsdc](https://www.paypal.me/mybsdc)
> Every time you spend money, you're casting a vote for the kind of world you want. -- Anna Lappe

![pay](https://s2.ax1x.com/2020/01/31/1394at.png "Donate")

![每一次你花的钱都是在为你想要的世界投票。](https://s2.ax1x.com/2020/01/31/13P8cF.jpg)

开源不求盈利，多少随缘...star也是一种支持。

### 作者
- 主程序以及框架：[@luolongfei](https://github.com/luolongfei)
- 英文版文档：[@肖阿姨](#)

### 鸣谢
- [PHPMailer](https://github.com/PHPMailer/PHPMailer/)（邮件发送功能依赖此库）
- [guzzle](https://github.com/guzzle/guzzle)（Curl库）

### 开源协议
[MIT](https://opensource.org/licenses/mit-license.php)