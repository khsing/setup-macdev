# Setup Mac Development Envrionment
This document describe how to perfectly setup a Mac as web developing  environment.  Mostly I'm focused PHP environment. If you have any suggestion, feel free to contact me by [email](mailto:khsing.cn_AT_gmail.com) or [twitter](https://twitter.com/khsing).

The following is Contents.

- [Greate IDE and Apps](#greate-ide-and-apps).
- [Xcode or Command Line Tools](#xcode-or-command-line-tools)
- [System Integrity Protection](#system-integrity-protection)
- [Homebrew](#homebrew)
- [Nginx, PHP, and MySQL](#nginx-php-and-mysql)
- [DNS and Port Forwarding](#dns-and-port-forwarding)
- [Final](#final)

## Greate IDE and Apps
There are many greate IDE and Apps  avalibe on Mac OS X.  

- IDE: [TextMate](http://macromates.com), [Sublime Text 2/3](http://www.sublimetext.com), [Atom](https://atom.io)
- Browser: [Chrome](https://google.com/chrome), [Firefox](https://www.firefox.com), [Safari](https://www.apple.com/safari)
- Terminal: [iTerm2](https://www.iterm2.com)
- Window Management: [Spectacle](https://www.spectacleapp.com)
- Password Mangement : [1Password](https://agilebits.com/onepassword)
- Cloud Storage: [Dropbox](https://www.dropbox.com), [OneDrive](https://onedrive.live.com), [Google Drive](https://google.com/drive)
- Shells: [Zsh](http://www.zsh.org/) with [Oh My Zsh](http://ohmyz.sh)
- Others: 
	- Keep your mac awake: [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake): 	-
	- GIF Record: [LICEcap](http://www.cockos.com/licecap/)
	- Keyboard Remap: [Karabiner](https://pqrs.org/osx/karabiner/)
	- MySQL/PostgreSQL GUI Client: [Sequel Pro](http://sequelpro.com), [PSequel](http://www.psequel.com)
	- [ImageOptim](https://imageoptim.com) 

- Extensions
	- [Emmet](http://emmet.io/) 
	- DocBlockr: [Sublime Text](https://packagecontrol.io/packages/DocBlockr), [Atom](https://atom.io/packages/docblockr)
	- For Chrome
		- [Don't Track me Google](https://chrome.google.com/webstore/detail/dont-track-me-google/gdbofhhdmcladcmmfjolgndfkpobecpg)
	- For Safari
		- [JSONview](https://github.com/acrogenesis/jsonview-safari)
		- [ClickToPlugin](https://hoyois.github.io/safariextensions/clicktoplugin/)
		- [Fontface Ninja](http://www.fontface.ninja)

## Xcode or Command Line Tools
If you are a Mac/iOS developer, you have to install Xcode. If not you just install Command Line Tools.

```
xcode-select --install
```

If you have installed Xcode from App Store

```
xcode-select -s /Applications/Xcode.app/Contents/Developer
```
## System Integrity Protection
Since OS X E10.11, Apple has enabled a new default security oriented featured called System Integrity Protection, often called rootless.

Unfortunately, this feature will disturb us install Homebrew and modify system-wild configurations, such as firewall. So we have to disable it during setup environment. After this setup, it's better to re-enable SIP.

Disable step:

-  Restart Mac and hold `CMD+R` keys.
- Enter recovery mode, open Utility -> Terminal
- Execute `csrutil disable`
- Reboot

Re-enable step:

- Enter Recovery mode and execute `csrutil enable`
- Reboot

You can check SIP status by `csrutil status`

## Homebrew
Homebrew is a greate build system for OS X. It just like Ports Tree on FreeBSD.

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
After install Homebrew, better to install homebrew `git` instead of Apple's.

```
brew install git
```

## Nginx, PHP, and MySQL

### Nginx
I prefer to using Nginx as web server, because it's simple enough.

```
brew install nginx
```

### PHP
Homebrew pre-built PHP-curl is using system-wild curl library, but it cause some problem, so that use brew version `curl` and `openssl` is a good option.


```
brew install curl --with-openssl
```
Although PHP7 is came out, but PHP 5.x is more stable and many project hasn't capitiable with PHP7. 

Before install `php56`, tap a PHP repo is necessary.  

```
brew tap homebrew/php
```
Then install `php56` with homebrew libxml2 and  curl.

```
brew install php56  --with-homebrew-libxml2  --with-homebrew-curl  --with-pear
```
If you have other php module need to install, just `brew search php-*` and install it.

### MySQL/PostgreSQL
MySQL/PostgreSQL is very good partner for PHP/Django/Rails 

```
brew install mysql postgresql
```
### Homebrew Services
[Homebrew Services](https://github.com/Homebrew/homebrew-services) can manage every service you install and control by `launchctl`

```
brew tap homebrew/services
```
So that you can:

- start mysql by `brew services start mysql` 
- restart php-fpm by `brew services restart php56`
- stop Nginx by `brew services stop Nginx`

### Configure Nginx 

Only root user able to LISTEN 1000-under port, Nginx is install by user and started by user, so that can't listen 80 port, but we don't want open link with port like http://foo.dev:8080, so we have to setup port forwarding. Here is steps.

- Nginx: `/usr/local/etc/nginx/servers/dev.conf`

*** DO NOT forget to replace USERNAME with yours. ***

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
This configuration will point `foobar.dev` to `/Users/USERNAME/workspace/www/foobar/public` and put route to `index.php`, most PHP framework routing like this.

And then restart Nginx with following command.

```
brew services restart nginx
```

## DNS and Port Forwarding
In developing environment, you'd better config a .dev domain which point to localhost and simply address to different project by name. e.g. `foo.dev -> ~/workspace/foo` and `bar.dev  -> ~/workspace/bar`. So that we can access it directly in browser.


### dev domain and DNS resolver
`dnsmasq` is powerful dns cache and easy to config.

```
brew install dnsmasq
```
Start dnsmasq by Homebrew Services `brew services start dnsmasq`

Edit `/usr/local/etc/dnsmasq.conf` with following lines.

```
address=/.dev/127.0.0.1
listen-address=127.0.0.1
port=35353
```
Make a directory `sudo mkdir /etc/resolver` and put a file named `dev` which content is:

```
nameserver 127.0.0.1
port 35353
```
Check it works fine or not by `ping foobar.dev`

### Configure Port Forwarding 
Since Yosemite(10.10), `ipfw` is replaced by `pf`. Here is how to do port forwarding with `pf`

- Create a pf rules file `/etc/pf.anchors/dev.cutedge`

```
rdr pass on lo0 inet proto tcp from any to self port 80 -> 127.0.0.1 port 8080
rdr pass on lo0 inet proto tcp from any to self port 443 -> 127.0.0.1 port 8443
```
NOTICE: Last line of this file MUST be blank line.

Test this anchor file:

```
sudo pfctl -vnf /etc/pf.anchors/com.cutedge
```
Check pf status:

```
sudo pfctl -s nat 
```
Modify `/etc/pf.conf` 

- Append line `rdr-anchor "devport"` after line `anchor "com.apple/*"`
- Append line `load anchor "devport" from "/etc/pf.anchors/dev.cutedge"` after `load anchor "com.apple" from "/etc/pf.anchors/com.apple"`

And keep last line is blank.

Check `pf.conf` by `sudo pfctl -ef /etc/pf.conf`

- Automate start pf

```
sudo defaults write /System/Library/LaunchDaemons/com.apple.pfctl ProgramArguments '(pfctl, -f, /etc/pf.conf, -e)'
```
This need SIP is disabled.

## Final

OK, Right now, we have setup local PHP developing environment. Let enjoy it.

Don't forget to turn on SIP.


