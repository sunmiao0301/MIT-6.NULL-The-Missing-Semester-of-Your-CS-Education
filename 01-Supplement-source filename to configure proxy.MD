***以下部分来自 07Debugging and Profiling 的 Homework 的 最后一题，但是与第一节部分知识有一定交互，故作为01的supplement***

(进阶题) `curl ipinfo.io` 命令或执行 HTTP 请求并获取关于您 IP 的信息。打开 [Wireshark](https://www.wireshark.org/) 并抓取 `curl` 发起的请求和收到的回复报文。（提示：可以使用`http`进行过滤，只显示 HTTP 报文）

```bash
####################### 这部分我也搬运到了第二部分 —— 中（因为那边也有很多相关知识）
# 考虑到暂时没有接触到环境内编程，所以我们先进行一些其他的测试
# 看看设置代理和不设置代理 得到的ip信息是不是不一样

# 在Windows上的gitbash
/d/repo (master)
$ vi proxy

# 设置一下代理 这里我们模仿zhp的代理脚本文件 写一下我们自己的
export https_proxy=http://127.0.0.1:33210
export http_proxy=http://127.0.0.1:33210 

# 这样执行，zhp的解释是：脚本内的变量在脚本执行完之后就失效了（涉及变量作用域的知识）
# 其他解释：在当前bash环境下读取并执行FileName中的命令。该filename文件可以无”执行权限”
/d/repo (master)
$ ./proxy

# 也得不到代理信息
/d/repo (master)
$ env|grep -i proxy

# 所以得到的ip地址依旧是安徽合肥
/d/repo (master)
$ curl ipinfo.io
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   255  100   255    0     0     70      0  0:00:03  0:00:03 --:--:--   268{
  "ip": "210.45.123.248",
  "city": "Hefei",
  "region": "Anhui",
  "country": "CN",
  "loc": "31.8639,117.2808",
  "org": "AS4538 China Education and Research Network Center",
  "timezone": "Asia/Shanghai",
  "readme": "https://ipinfo.io/missingauth"
}

# 这样执行
# zhp解释：表示在命令行执行脚本内部语句，相当于把脚本内的东西暴露在当前环境下
# 其他解释：打开一个子shell来读取并执行FileName中命令。该filename文件需要”执行权限”
/d/repo (master)
$ source ./proxy

# 能够得到代理信息
/d/repo (master)
$ env|grep -i proxy
https_proxy=http://127.0.0.1:33210
http_proxy=http://127.0.0.1:33210

# 所以香港了
/d/repo (master)
$ curl ipinfo.io
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   288    0   288    0     0    271      0 --:--:--  0:00:01 --:--:--   272{
  "ip": "218.250.146.87",
  "hostname": "n218250146087.netvigator.com",
  "city": "Hong Kong",
  "region": "Central and Western",
  "country": "HK",
  "loc": "22.2783,114.1747",
  "org": "AS4760 HKT Limited",
  "timezone": "Asia/Hong_Kong",
  "readme": "https://ipinfo.io/missingauth"
}

# 也正是因为如此，
# 举例： 比如您在一个脚本里export KKK=111,假如您用./a.sh执行该脚本，执行完毕后，您运行echoKKK ,发现没有值，假如您用source来执行 ，然后再echo ,就会发现KKK=111。因为调用./a.sh来执行shell是在一个子shell里运行的，所以执行后，结构并没有反应到父shell里，但是 source不同他就是在本shell中执行的，所以能够看到结果.

# 又或者是如下角度
# source scriptname使得脚本中的命令被读取并运行 ，就好像是键入它们一样。
# 而直接鍵入脚本的名字，然后回车呢，其实就像调用一个内置命令一样。
```

以上部分参考[链接](https://blog.csdn.net/banana_baba/article/details/50573497)

p.s.

```bash
# socks5与http/s有何区别
export all_proxy=socks5://127.0.0.1:33211

# 测试
/d/repo (master)
$ env|grep -i proxy

/d/repo (master)
$ curl ipinfo.io
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   255  100   255    0     0    608      0 --:--:-- --:--:-- --:--:--   611{
  "ip": "210.45.123.248",
  "city": "Hefei",
  "region": "Anhui",
  "country": "CN",
  "loc": "31.8639,117.2808",
  "org": "AS4538 China Education and Research Network Center",
  "timezone": "Asia/Shanghai",
  "readme": "https://ipinfo.io/missingauth"
}

/d/repo (master)
$ export all_proxy=socks5://127.0.0.1:33211

/d/repo (master)
$ env|grep -i proxy
all_proxy=socks5://127.0.0.1:33211

/d/repo (master)
$ curl ipinfo.io
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   283  100   283    0     0    719      0 --:--:-- --:--:-- --:--:--   718{
  "ip": "178.173.225.85",
  "hostname": "host-by.nerocloud.io",
  "city": "Hong Kong",
  "region": "Central and Western",
  "country": "HK",
  "loc": "22.2783,114.1747",
  "org": "AS48024 NEROCLOUD LTD",
  "timezone": "Asia/Hong_Kong",
  "readme": "https://ipinfo.io/missingauth"
}

# 注意这里直接ping github.com是不行的
# 需要在 https://ipaddress.com/website/github.com 得到GitHub的ip地址为140.82.114.3
# 可以ping通
/d/repo (master)
$ ping 140.82.114.3

Pinging 140.82.114.3 with 32 bytes of data:
Reply from 140.82.114.3: bytes=32 time=325ms TTL=38
Reply from 140.82.114.3: bytes=32 time=327ms TTL=38

# 说明以下也是可以的
export all_proxy=socks5://127.0.0.1:33211
```





