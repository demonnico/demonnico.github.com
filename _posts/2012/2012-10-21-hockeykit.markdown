---
author: demon
comments: true
date: 2012-10-21 22:31:24
layout: post
slug: hockeykit
title: HockeyKit
wordpress_id: 24
categories:
- IOS
- 开发
tags:
- HockeyKit
- IOS
- OTA
---

在不知道[TestFight](https://testflightapp.com/)之前，傻逼地认为ipa交给测试之后，只能通过itunes同步进行安装（包括破解后用的第三方工具安装）。后来发现这个好东西之后其实也没怎么用得上，因为公司的网速实在不怎么样，每次打包好之后的ipa包大概也有50M+,上传到TestFight服务器上测试进行OTA安装又要重新下载，感觉实在有点费时间，google了之后发现其实还有[HockeyKit](https://github.com/TheRealKerni/HockeyKit/)这样的好东西。

简单地搭建一个支持php的服务器就能安装Hockey的服务端，假设在公司或者小团队的内部网络中，开发编译打包后上传就不是问题了，测试的安装也可以一气呵成。 lion中自带了apache，打开httpd.conf之后修改一下配置，让apache支持php就可以了。

下面简单记录一下HockeyKit Server的搭建过程，英文好的可以参考HotkeyKit在Github上的[wiki](https://github.com/TheRealKerni/HockeyKit/wiki/Server)
	
    1. git clone https://github.com/TheRealKerni/HockeyKit.git，打开包之后会发现包内包含server，client，还有demo。这里我们只要server
	2. 然后我们要有一个支持php5的服务器，确保你的服务器的httpd.conf 中 _AllowOverride All_ ，以支持HockeyKit Server中子目录的配置重载，确保服务的rewirte可用（lion默认开启 LoadModule rewrite_module libexec/apache2/mod_rewrite.so)
	3. 在服务器相应的位置创建一个子目录，在这里我们把这个目录作为 HockeyKit的公共服务目录,我们假设这个目录叫做_public_
	4. cd hockeykit/server/php/public ls .
	发现包含隐藏的.htaccess，将包内所有的文件都cp到我们在上一步创建的_public_中
	5. 在服务目录下创建另一个子目录，出于安全考虑，我们把这个目录放在_public_的外部，可以和_public_处于同一层级，我们可以把这个目录命名为_include_，用于存放一些HockeyKit服务用到的一些库文件
	6. 打开我们刚刚cp到_public_包内的config.php文件,修改_include_相对与_public_路径
	7. 修改权限，使php对_public_中的stats目录拥有写权限
	8. 打开stats目录下的.htaccess文件可以针对当前目录的访问权限等进行适当配置
	9. 在我们上文中的_public_目录下创建你的应用子目录，用APP的bundle name进行命名，以便区分
	10. 将hockeykit/server/php/public/test中所有的目录copy到我们在服务器下的创建的public中
	11. 在设备中访问xxx.xxx.xxx.x/xxx/public，应该能看到一个基本的界面了，里面包含了我们在上一步中放入public中的一些ios的应用（如果是android设备的话看到的应该是一些android的应用）
	12. 点击install application，如果出现alertview提示错误，重新回到步骤2检查一下是否设置了_AllowOverride All


