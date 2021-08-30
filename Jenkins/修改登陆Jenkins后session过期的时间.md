Jenkins的web默认会话30分钟过期,因此可以修改该值. 因为Jenkins使用Jetty,所以启动war包时，追加参数覆盖原配置即可`-DsessionTimeout=<minutes>`. 或者也可以配置文件方式,在Jenkins的配置目录下找到`web.xml`,路径是`jenkins/var/WEB-INF/web.xml`.

例如我的路径是：

```
vim /opt/apache-tomcat-9.0.41/webapps/ROOT/WEB-INF/web.xml
```

修改该文件,在<session-config>中添加<session-timeout>,值为session的过期分钟数,比如需要6个小时过期就填**360**. 

```shell
  <session-config>
    <session-timeout>360</session-timeout>
  </session-config>
```

然后再重启jenkins即可