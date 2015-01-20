---
layout: post
title:  "Ubuntu下Apache反向代理设置"
date:   2014-11-06 20:15:40
categories: teach
---

第一步，首先是要打开apache的相关功能。

{% highlight bash %}
sudo a2enmod rewrite #打开 url 重写
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_balancer
{% endhighlight %}

第二步，设置网站的配置文件，在 `/etc/apache2/sites-available/` 目录下 新建文件 a.com， 写入如下内容

{% highlight bash %}
<VirtualHost *:80>
ServerName a.com
ProxyPreserveHost On
ProxyRequests Off
<Proxy *>
Order deny,allow
Allow from all
</Proxy>
ProxyPass / http://192.168.0.10/
ProxyPassReverse / http://192.168.0.10/
</VirtualHost>
{% endhighlight %}

第三步，执行、启用这个站点

{% highlight bash %}
sudo a2ensite a.com
sudo service apache2 reload
{% endhighlight %}

这样所有来自 `a.com` 的请求都会转发到 `http://192.168.0.10`

虚拟主机设置请戳[这里](http://www.zrq495.com/ubuntu-apache2-how-to-bind-multiple-domain-names/)