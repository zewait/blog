title: 安装httpd并整合tomcat
date: 2015-05-01 09:21:50
tag:
- tomcat
- apache httpd
- linux
categories:
- 服务器
---

# 安装httpd
下载[httpd](http://httpd.apache.org/)并解压
{% codeblock lang:shell %}
  weget http://mirrors.hust.edu.cn/apache//httpd/httpd-2.2.29.tar.gz
  tar -zxvf httpd-2.2.29.tar.gz
{% endcodeblock %}

配置编译安装
{% codeblock lang:shell %}
  cd httpd-2.2.29
  #prefix是指定了安装位置
  ./configure --prefix=/usr/local/apache2 \      # 安装目录
  --enable-modules=most \                        # 大部分模块静态编译到httpd的二进制文件中
  --enable-mods-shared=all \                     # 表示动态加载所有模块,如果去掉shared的话,是静态加载所有模块
  --enable-so \
  make && make install
{% endcodeblock %}

以上没有错误就启动httpd, 注意:因为还没加进服务，所以需要到httpd安装目录下用前端控制工具启动(不能用service httpd start)
{% codeblock lang:shell %}
  /usr/local/apache2/bin/apachectl start
{% endcodeblock %}
打开浏览器浏览[http://localhost](http://localhost)看见It's ok就代表httpd安装并启动成功了

加进服务和开机启动
{% codeblock lang:shell %}
  cp /usr/local/apache2/bin/apachectl /init.d/httpd
  ln -s /etc/init.d/httpd /etc/rc.d/rc5.d/S85httpd
  # 运行chkconfig --list发现列表中并没有httpd,通过chkconfig --add httpd来添加,
  # 可能会提示httpd服务不支持chkconfig,需要编辑/etc/init.d/httpd,加入注析#chkconfig 345 85 15
  # 345代表哪些linux级别需要启动httpd,启动序号85 关闭序号15
  chkconfig --add httpd
  # 发现存在了
  chkconfig --list httpd
  # 接下来就可能用service http命令关闭和启动httpd了

  # 加进开机启动
  chkconfig httpd on
{% endcodeblock %}

# 整合tomcat
在/usr/local/apache2/conf/httpd.conf最后加上Include conf/extra/tomcat-vhosts.conf,
编辑/usr/local/apache2/conf/extra/tomcat-vhosts.conf
{% codeblock lang:xml %}
# enable name virtual
NameVirtualHost *:80

<VirtualHost *:80>
  ServerName test.h4fan.com
  DocumentRoot /path/to/project/root
  ProxyPass / http://localhost:8080/
  ProxyPassReverse / http://localhost:8080/
  ErrorLog "logs/test-error.log"
</VirtualHost>
{% endcodeblock %}

重启httpd,发现出错了
*Invalid command 'ProxyPass', perhaps misspelled or defined by a module not included in the server configuration*
需要安装proxy等模块

介绍下添加模块的命令
{% codeblock lang:shell %}
/usr/local/apache2/bin/apxs -c -i -a mod_proxy.c proxy_util.c
{% endcodeblock %}
说明:
-c 执行编译操作
-i 安装操作,安装一个或多个动态共享对象到modules目录
-a 自动增加一个LoadModule到httpd.conf文件,以激活模块,若此存在则启动之
-A 与-a类似,但是它增加的LoadModule前面加有#
-e 需要执行编辑操作,可与-a和-A选项配合使用,与-i操作类似,修改httpd.conf文件,但并不安装此模块

切换到源代码目录下的/modules/proxy,执行
{% codeblock lang:shell %}
#/usr/local/apache2/bin/apxs -c -i -a mod_proxy.c proxy_util.c
#/usr/local/apache2/bin/apxs -c -i -a mod_proxy_http.c
#/usr/local/apache2/bin/apxs -c -i -a mod_proxy_ajp.c ajp*.c
{% endcodeblock %}

现在再启动httpd就没出错了

