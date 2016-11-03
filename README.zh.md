
# 设置Mac开发环境
本文说明了如何在Mac上优雅的设置一套PHP开发环境，如果你有什么建议的话可以 [email](mailto:khsing.cn_AT_gmail.com) 我或者在 [twitter](https://twitter.com/khsing)联系我也可以(微博的话同ID).

接下来是主要内容
- [牛逼的 IDE 和 Apps](#greate-ide-and-apps).
- [Xcode 或者 Command Line Tools](#xcode-or-command-line-tools)
- [System Integrity Protection](#system-integrity-protection)
- [Homebrew](#homebrew)
- [Nginx, PHP, and MySQL](#nginx-php-and-mysql)
- [DNS and 端口转发](#dns-and-port-forwarding)
- [最后](#final)

## Greate IDE and Apps
Mac上有很多牛逼的编辑器和应用

- IDE: [TextMate](http://macromates.com), [Sublime Text 2/3](http://www.sublimetext.com), [Atom](https://atom.io)
- 浏览器: [Chrome](https://google.com/chrome), [Firefox](https://www.firefox.com), [Safari](https://www.apple.com/safari)
- 终端: [iTerm2](https://www.iterm2.com)
- 窗口管理: [Spectacle](https://www.spectacleapp.com)
- 密码管理 : [1Password](https://agilebits.com/onepassword)
- 云盘: [Dropbox](https://www.dropbox.com), [OneDrive](https://onedrive.live.com), [Google Drive](https://google.com/drive)
- Shells: [Zsh](http://www.zsh.org/) with [Oh My Zsh](http://ohmyz.sh)
- 其他: 
	- 防睡眠模式: [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake): 	-
	- GIF录像: [LICEcap](http://www.cockos.com/licecap/)
	- 键盘映射: [Karabiner](https://pqrs.org/osx/karabiner/)
	- MySQL/PostgreSQL图形化工具: [Sequel Pro](http://sequelpro.com), [PSequel](http://www.psequel.com)
	- PNG压缩[ImageOptim](https://imageoptim.com) 

- 扩展
	- [Emmet](http://emmet.io/) 
	- DocBlockr: [Sublime Text](https://packagecontrol.io/packages/DocBlockr), [Atom](https://atom.io/packages/docblockr)
	- For Chrome
		- [Don't Track me Google](https://chrome.google.com/webstore/detail/dont-track-me-google/gdbofhhdmcladcmmfjolgndfkpobecpg)
	- For Safari
		- [JSONview](https://github.com/acrogenesis/jsonview-safari)
		- [ClickToPlugin](https://hoyois.github.io/safariextensions/clicktoplugin/)
		- [Fontface Ninja](http://www.fontface.ninja)

## Xcode or Command Line Tools

如果你是一个iOS/Mac应用开发，你可以从Mac App Store中安装Xcode，如果不是直接安装Command Line Tools就可以了

```
xcode-select --install
```

如果你已经安装了Xcode

```
xcode-select -s /Applications/Xcode.app/Contents/Developer
```
## System Integrity Protection
自从 OS X 10.11, 苹果公司引入了一套安全机制防止系统文件被修改，这个玩意叫SIP，也叫`rootless`。

不幸的是，这个玩意会把`/usr/local`和`/etc`目录都会锁上，这会妨碍安装`Homebrew`和配置DNS的一些小技巧。所以得先关上这个bug(feature)，设置完了之后再打开。

关闭步骤:

- 关机并按住 `CMD+R` 启动.
- 进入恢复模式之后打开 Utility -> Terminal
- 执行 `csrutil disable`
- 重启

启用步骤:

- 进入恢复模式，终端中执行 `csrutil enable`
- 重启

可以用 `csrutil status` 来检查当前SIP的状态

## Homebrew
Homebrew是一套包管理系统，有点类似FreeBSD的ports-tree，可以方便的安装开源的App

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
安装完Homebrew之后强烈建议安装最新的`git`来替代苹果在Command Line Tools中自带的。

```
brew install git
```

## Nginx, PHP, and MySQL

### Nginx
相比于Apache，我更喜欢用Nginx。

```
brew install nginx
```

### PHP
Homebrew 预编译的PHP使用的`libcurl`是macOS系统自带的，但是这个版本本身是有问题的，所以最好是使用Homebrew中的curl

```
brew install curl --with-openssl
```
PHP7.0已经发布了，不过一些项目都还在PHP5.x上，所以就安装PHP5.6就好了

安装 `php56`之前, 先来`tap`一下PHP的仓库

```
brew tap homebrew/php
```
然后安装配备Homebrew版本libxml2和curl的`php56`.

```
brew install php56  --with-homebrew-libxml2  --with-homebrew-curl  --with-pear
```
如果你有需要安装的PHP模块，搜索并安装就好了，不过要注意版本 `brew search php-*`。

### MySQL/PostgreSQL
安装MySQL或者PostgreSQL

```
brew install mysql postgresql
```

### Homebrew Services
[Homebrew Services](https://github.com/Homebrew/homebrew-services)是一套可以通过 `launchctl`来管理安装的服务的套件，非常好用推荐服用。

```
brew tap homebrew/services
```
然后就可以:

- 启动MySQL `brew services start mysql` 
- 启用PHP-FPM `brew services restart php56`
- 启用Nginx `brew services stop Nginx`

### Configure Nginx 

只有root用户可以监听1000以下的端口， Homebrew 安装 Nginx 都是使用当前用户，`brew services`也是以普通用户来启动服务，HTTP服务又是在80端口，我们也不想访问的时候加上一个8080，也不像`brew services`的时候`sudo`一下，所以接下来的配置呢，会让Nginx运行在8080端口，然后通过系统的`pf`把80端口映射到8080端口。

增加一个Nginx配置文件

- Nginx: `/usr/local/etc/nginx/servers/dev.conf`

*** 别忘了替换 USERNAME ***

```
server {
    listen       8080;
    server_name ~^(.+)\.dev$;
    set $file_path $1;
    root    /Users/USERNAME/workspace/www/$file_path/public;
    location / {
	    index index.html index.php;
	    try_files $uri $uri/ /index.php?$query_string;
	}
    charset utf-8;
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
    location ~ \.php$ {
        root    /Users/USERNAME/workspace/www/$file_path/public;
        fastcgi_pass   127.0.0.1:9000; # PHP-FPM default running on this port.
        fastcgi_index  index.php;
        include        fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```
这个配置会将 `foobar.dev` 的 `DocumentRoot` 指向 `/Users/USERNAME/workspace/www/foobar/public` 并且把路由请求交给 `index.php`, 大部分的PHP框架都是这么配置的.

这样配置之后只要重启Nginx就好了

```
brew services restart nginx
```

## DNS 和端口转发

在开发环境中更倾向于使用一个`dev`结尾的域名用于开发方便，比如 `foo.dev -> ~/workspace/www/foo` 或者 `bar.dev  -> ~/workspace/www/bar`. 这样我们就可以直接在浏览器中访问了相对应的项目了。

### dev 域名和 DNS 解析
`dnsmasq` 是一个非常强悍的DNS转发器。

```
brew install dnsmasq
```
一样的可以使用Homebrew Services管理服务 `brew services start dnsmasq`

编辑 `/usr/local/etc/dnsmasq.conf` 加入下面这些配置.

```
address=/.dev/127.0.0.1
listen-address=127.0.0.1
port=35353
```

创建一个 `sudo mkdir /etc/resolver` 并且放一个 `dev` 文件进去，内容如下

```
nameserver 127.0.0.1
port 35353
```
重启`dnsmasq`，`brew services restart dnsmasq`

`ping`一下可以检查一下是不是配置好了，`ping foobar.dev`

### Configure 端口转发
自从 Yosemite(10.10)开始, `ipfw` 被 `pf` 取代. 下面就是如何用`pf`配置80端口转发8080

- 创建一个`pf`规则文件 `/etc/pf.anchors/dev.cutedge`

```
rdr pass on lo0 inet proto tcp from any to self port 80 -> 127.0.0.1 port 8080
rdr pass on lo0 inet proto tcp from any to self port 443 -> 127.0.0.1 port 8443
```
注意: 最后一行必须为空

测试这个规则文件:

```
sudo pfctl -vnf /etc/pf.anchors/dev.cutedge
```

检查`pf`的状态:

```
sudo pfctl -s nat 
```

修改 `/etc/pf.conf` 

- 找到`rdr-anchor "com.apple/*"`，在这一行后插入 `rdr-anchor "devport"`
- 找到`load anchor "com.apple" from "/etc/pf.anchors/com.apple"`，在这一行后插入`load anchor "devport" from "/etc/pf.anchors/dev.cutedge"` 

保持最后一行空白

检查一下配置文件`pf.conf` 是否正确 `sudo pfctl -ef /etc/pf.conf`

- 自动启动 `pf`

```
sudo defaults write /System/Library/LaunchDaemons/com.apple.pfctl ProgramArguments '(pfctl, -f, /etc/pf.conf, -e)'
```

执行这一条需要SIP处于关闭状态

## Final

OK，现在PHP开发环境就配置好了，在新建一个`$HOME/workspace/www/test/public/index.php`文件

```
<?php
phpinfo();
```

然后访问[http://test.dev/](http://test.dev/)，就可以看到了

最后，不要忘了重新打开SIP。


