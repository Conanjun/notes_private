### Slow HTTP DOS攻击

描述：
WVS扫描发现http慢速拒绝服务，使用测试脚本对该站点测试，发现页面超时无法打开，达到拒绝服务效果。
停止测试脚本，页面可立即打开。


修复参考：

方法一、升级apache新版本。

方法二、使用mod_reqtimeout和mod_qos两个模块相互配合来防护。
1、mod_reqtimeout用于控制每个连接上请求发送的速率。配置例如：
#请求头部分，设置超时时间初始为10秒，并在收到客户端发送的数据后，每接收到500字节数据就将超时时间延长1秒，但最长不超过40秒。可以防护slowloris型的慢速攻击。
RequestReadTimeout header=10-40,minrate=500

#请求正文部分，设置超时时间初始为10秒，并在收到客户端发送的数据后，每接收到500字节数据就将超时时间延长1秒，但最长不超过40秒。可以防护slow message body型的慢速攻击。
RequestReadTimeout body=10-40,minrate=500
需注意，对于HTTPS站点，需要把初始超时时间上调，比如调整到20秒。

2、mod_qos用于控制并发连接数。配置例如：

（1）当服务器并发连接数超过600时，关闭keepalive
QS_SrvMaxConnClose 600

（2）限制每个源IP最大并发连接数为50

QS_SrvMaxConnPerIP 50
这两个数值可以根据服务器的性能调整。




### 泄露Web Server 各种组件详细版本号相关信息

漏洞描述：
测试发现服务Response信息中泄露了Web Server 各种组件详细版本号相关信息。为攻击者做信息收集提供方便，便于收集各组件存在漏洞情况。

修复建议：
在服务端进行配置，返回客户端的响应包中禁止显示此类信息。



### SSL漏洞修复

Web网站的SSL漏洞主要包括如下几种：

1、SSL RC4 Cipher Suites Supported

2、SSL Weak Cipher Suites Supported

3、The FREAK attack(export cipher suites supported)

4、The POODLE ataack（SSLV3 supported）

SSLv3协议POODLE漏洞
漏洞影响：攻击者可以利用此漏洞获取受害者的https请求中的敏感信息，如cookies等。
漏洞介绍和修复参考：https://bbs.aliyun.com/read/179684.html


5、SSL 2.0 deprecated protocol

6、OpenSSL 'ChangeCipherSpec' MiTM Vulnerability


(awvs官方推荐)
SSL修复建议总结如下：

1、禁用SSL 2.0 and SSL 3.0

2、禁用 TLS 1.0 压缩以及弱密码

3、针对不同的web Server修改如下配置


【OPENSSL】使用如下配置
ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5


【Apache】配置指南 mod_ssl
适用于Apache HTTP Server 2.2+/2.4+ with mod_ssl，配置文件apache/conf/extra/httpd-ssl.conf

SSLProtocol ALL -SSLv2 -SSLv3
SSLHonorCipherOrder On
SSLCipherSuite ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5
SSLCompression Off

需要注意的是Redhat系列的Linux版本，可能要做如下配置：
对于部分apache版本，不支持SSLCompression Off的配置，可能需要在/etc/sysconfig/httpd文件中插入OPENSSL_NO_DEFAULT_ZLIB=1配置来禁用ssl压缩



【Nginx】配置指南

ssl_prefer_server_ciphers On;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5;

需要注意的是nginx下，关闭TLS压缩，可能和你运行的OpenSSL及nginx版本相关，如果你使用是OpenSSL 1.0上版本，nginx版本为1.1.6以上或者是1.0.9+ TLS压缩默认就是关闭的。如果你使用的OpenSSL1.0以下版本，由必须使用nginx 1.2.2+/1.3.2




### Tomcat 弱口令 && 后台getshell漏洞

访问Tomcat后台，需要对应用户有相应权限。

Tomcat7+权限分为：

 - manager（后台管理）
   - manager-gui 拥有html页面权限
   - manager-status 拥有查看status的权限
   - manager-script 拥有text接口的权限，和status权限
   - manager-jmx 拥有jmx权限，和status权限
 - host-manager（虚拟主机管理）
   - admin-gui 拥有html页面权限
   - admin-script 拥有text接口权限

这些权限的具体作用 http://tomcat.apache.org/tomcat-8.5-doc/manager-howto.html


在`conf/tomcat-users.xml`文件中配置用户的权限：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">

    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <role rolename="manager-jmx"/>
    <role rolename="manager-status"/>
    <role rolename="admin-gui"/>
    <role rolename="admin-script"/>
    <user username="tomcat" password="tomcat" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-gui,admin-script" />

</tomcat-users>
```


可知用户名tomcat拥有上述【所有权限】，密码是`tomcat`

tomcat8:正常安装默认没有任何用户，且manager页面只允许本地IP访问。只有管理员手工修改了这些属性的情况下，才可以进行攻击。
tomcat7:

打开tomcat管理页面`http://your-ip:8080/manager/html`
输入弱密码`tomcat:tomcat`
即可访问后台
在后台部署war包，直接部署webshell到web目录。
