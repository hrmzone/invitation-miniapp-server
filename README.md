# 邀请函小程序后端服务
本项目基于[marriage-miniapp-server](https://github.com/Jiezhi/marriage-miniapp-server)和前端项目为[OnceLove](https://github.com/Jiezhi/OnceLove)。

修改后的后端项目为[invitation-miniapp-server](https://github.com/hrmzone/invitation-miniapp-server/)和前端项目为[invitation-miniapp](https://github.com/hrmzone/invitation-miniapp)。

> 感谢作者[Jiezhi](https://github.com/Jiezhi)的指导。

> 开源项目中有非常多很好的工具，但是由于开源项目无法开箱即用，而且缺少相应的配置说明，导致开源项目无法推广。可能是我能力问题，走了一点弯路，总算配置成功，并且和前端小程序配套可以使用，下面将这个小玩意的配置方法贴出来，供需要的用户借鉴。

> 我是一个职业培训机构[荆州青年教育](https://jzyouth.com)的老师，有学历提升(成教、网教、自考等)、职业资格方面的需求，可联系我    ---做个小广告

## 配置
1. 复制 `/src/main/resources/application.yml.template` 或者修改其扩展名生成 `application.yml` 文件；
2. 配置文件中，所有属性冒号后面的文字之前需要加空格（包括连接数据库的帐号和密码配置）;
3. 配置（与微信小程序开发配置中保持一致）

```
appid:wx77856a389b827dee
secret:f6cc28cbdfd7e46e2a24f6742
Token:whws2019
EncodingAESKey:APJVp5WTrtWOGMBFXkifMa2ZNUixPoV6Qx4T
```

**如果提示Token无法验证(消息服务，非消息服务无需关心)**

1. 确保url是正确的，如地址必须是服务器对应的地址，如上面的长地址，如不是仅仅域名，即包含appid的地址（http://wx.whwsxx.cn/wx/portal/wx77856a389b827dee）；
2. 验证token时服务器必须开启，否则无法验证token，且要保证开的是80端口，如果ssl是443端口；
3. update:微信小程序的域名服务器配置里面，域名地址禁止出现符号，因此不能出现`http://wx.whwsxx.cn/wx/portal/wx77856a389b827dee`这样的地址，在apache的转发配置里需要转发至相应的appid地址。

## Mysql数据库配置
### 数据库字符编码配置
1. 设置mysql字符集为utf8
2. 先停止mysql服务。修改/etc/mysql/my.cnf

```
[client]
default-character-set = utf8
[mysqld]
character-set-server = utf8
```

3. 编码未配置为unicode，插入中文会提示错误

```
ERROR 1366: 1366: Incorrect string value: '\xE6\xAD\xA6\xE6\xB1\x89...' for column 'address' at row 1
```

### 允许Mysql外网访问便于调式
1. `/etc/mysql/mysql.conf.d/mysqld.cnf`

```
bind-address          = 127.0.0.1
```

2. 为mysql数据库张号给与外网权限

```
use mysql select user,host from user;

grant all privileges on *.* to 'root'@'[允许的ip]' identified by '[密码]' with grant option;

flush privileges;
```

### 为数据库添加数据
为数据库表main_info添加初始数据(main_info为最重要的表，是小程序初始化数据)

```
INSERT INTO `invite`.`main_info` (`id`, `address`, `app_name`, `appid`, `code`, `date`, `he`, `he_tel`, `hotel`, `lat`, `lng`, `she`, `she_tel`, `uid`) VALUES ('1', '武汉阳逻经济开发区汽渡路413号', '娲石学校', 'wx77856a389b827dee', '434000', '', '座机', '13419586088', '武汉市娲石技术学校', '30.628050', '114.574790', '胡老师', '027-59204559', '1');
```

## maven打包
1. 清理maven缓存
`mvn clean`
2. maven打包，根据pom配置文件，生成的jar包
`mvn package`
3. 在项目的target目录下，项目被打包成jar包文件


## 运行
1. Spring boot自带了容器，可以通过命令直接跑起来
`java -jar weixin-java-miniapp-love-1.0.0-local.jar`
2. 服务器默认情况下是以8080端口，在调试的时候，要保证vps的防火墙开启了8080端口，否则无法访问，排查起来麻烦的很。
3. 指定spring boot为80端口 `--server.port=80`
`java -jar weixin-java-miniapp-love-1.0.0-local.jar --server.port=80`

## Apache2转发
直接使用spring boot的情况下，搞了好久都无法绑定ssl证书，即使指定443端口也无效，微信小程序绑定的域名均为ssl，使用apache2作为前面的代理服务器，方便后面配置SSL证书

1. 启用3个proxy模块 `proxy proxy_http proxy.load`
	`a2enmod proxy proxy_http`
2. 绑定域名并配置转发

```
ServerName wx.whwsxx.cn
ProxyPass / http://wx.whwsxx.cn:8080/
ProxyPassReverse / http://wx.whwsxx.cn:8080/
```
3.转发配置成功后,配置微信小程序平台地址为:`wx.whwsxx.cn`
## 安装ssl证书
1. 安装SSL模块`ssl.conf ssl.load`
2. 使用`a2enmod ssl`
3. 安装certbot使用带apache2的插件

```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository universe
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot python-certbot-apache 
```
4. 生成证书
`certbot -i apache -d wx.whwsxx.cn`
这里面最后一步选择`no redirec`t选项，sites-available目录下会生成ssl的配置文件
*微信小程序的消息服务，可以使用80端口，其他的域名服务配置的域名必须使用ssl*

