
### 关于本地端口转发

关于本地端口转发，无论我们是在config里配置好，还是在本地机上ssh连接的时候就-L声明，都会受到一个很麻烦的限制，**那就是必须是本地一个端口对应远端一个端口**

```bash
# p.s. 为了助记端口 如果我们用的是本地端口转发 我们可以选择将本地端口取名和远端一致 比如：
# 我们服务器上启动的服务是
root@ceph57 ~ # python -m SimpleHTTPServer 8888
Serving HTTP on 0.0.0.0 port 8888 ...

# 所以我们本地机的config是
Host root57
  User root
  HostName 10.10.8.57
  IdentityFile ~/.ssh/id_rsa
  ProxyJump msun@202.38.95.226:2222
# 我们选择了和服务器开放的端口一致的端口号
  LocalForward 8888:localhost:8888
# p.s. 前一个8888是本地的端口 后一个8888是服务器端口

# 然后我们在本地机上ssh登录远程机
PS C:\Users\hfut_> ssh root57
Last login: Fri Dec 24 15:28:41 2021 from 10.12.3.1
root@ceph57 ~ #
```

**然后就可以在本地浏览器上访问端口8888，在浏览器输入```http://10.10.8.57:8888```，结果如下**

<img src="https://raw.githubusercontent.com/sunmiao0301/Public-Pic-Bed/main/1224local.png" alt="1224local.png" style="zoom: 50%;" />

**但是这样有一个弊端就是服务器上的每个服务都需要在本地另开以一个新端口**

### 关于动态端口转发

*why not to try Dynamic? — zhp*

```bash
# p.s. 如果是使用ssh命令里面 就是：ssh -D port_number user@host
# p.s. 如果是在ssh config里面 就是：

Host root57
  User root
  HostName 10.10.8.57
  IdentityFile ~/.ssh/id_rsa
  ProxyJump msun@202.38.95.226:2222
  DynamicForward 80
# 如此，就可以在本地的80端口访问多个服务
# 接下来
# 和本地端口转发一样 我们在服务器上启动服务
root@ceph57 ~ # python -m SimpleHTTPServer 8888
Serving HTTP on 0.0.0.0 port 8888 ...

# 然后在本地启动ssh
PS C:\Users\hfut_> ssh root57
Last login: Fri Dec 24 17:22:11 2021 from 10.12.3.1
root@ceph57 ~ #
```

**在这之前，我们可以配置一下[SwitchyOmega](chrome-extension://padekgcemlokbadohgkifijomclgjgif/options.html#!/about)插件**

<img src="https://raw.githubusercontent.com/sunmiao0301/Public-Pic-Bed/main/1224SwitchOmega.png" alt="1224SwitchOmega" style="zoom:80%;" />



**然后就可以在本地访问了，在浏览器输入```http://10.10.8.57:8888```，结果如下(注意和本地端口转发的访问地址不同之处)**



<img src="https://raw.githubusercontent.com/sunmiao0301/Public-Pic-Bed/main/1224Dynamic.png" alt="1224Dynamic" style="zoom:50%;" />
