Docker默认是没有开启HTTP远程访问的，默认只支持通过unix socket通信操作docker daemon,使用HTTP restful接口需要修改配置。

**步骤如下:**

1. 修改配置文件

  文件位置`/usr/lib/systemd/system/docker.service`。将原来的`ExecStart`修改为：

```shell
#ExecStart=/usr/bin/dockerd
ExecStart=/usr/bin/docker daemon --tls=false -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```
2. 执行命令加载以上配置
   ```shell
   systemctl daemon-reload
   systemctl restart docker.service
   ```
3. 重启docker服务

如果docker命令是无法使用则在`etc/profile`中配置：

`export DOCKER_HOST= 'http://0.0.0.0:2375'`

通过以下命令使之生效
`source /etc/profile`

这种测试场景在系统使用`systemctl`服务的时候，以上是在centos7.2亲测，其他的os版本如有此情况也适用.