---
title: Nginx配置HTTPS证书
date: 2022-05-14 00:00:00.0
updated: 2022-08-10 15:43:10.239
categories: 
- Dev
tags: 
- Nginx
---

## 确保打开443端口

1. 以阿里云为例，来到控制台打开实例界面，然后点击配置安全组规则：

	![控制台里的配置安全组规则](/img/2022/08/nginx-https1.png)
    
2. 随后点击配置规则：

    ![配置规则](/img/2022/08/nginx-https2.png)

3. 开放443端口：
    ![开放443端口](/img/2022/08/nginx-https3.png)

我就是这里没打开，导致后面死活不成功。

## 用Certbot生成证书

去到[官网](https://certbot.eff.org/)。我这里把在`Ubuntu18`的`Nginx`安装的步骤抄了一遍。

1. 安装`snapd`：

   ```bash
   sudo apt install snapd
   ```

2. 确保你版本最新（我认为这一步可以跳过，不过他都说了我就抄下来吧）：

   ```bash
   sudo snap install core; sudo snap refresh core
   ```

3. 如果你用`apt`之类的包安装过了，请务必卸载掉：

   ```bash
   sudo apt remove certbot
   ```

4. 准备运行`certbot`命令（就是创建了个软连接）：

   ```bash
   sudo ln -s /snap/bin/certbot /usr/bin/certbot
   ```

5. 之后让他自动运行就好了，我这里没折腾手动的话怎么操作：

   ```bash
   sudo certbot --nginx
   ```

大功告成! 现在访问你的网站，如果不行就`Ctrl+F5`强制刷新一下，就能看到网址旁边有一个小锁了。