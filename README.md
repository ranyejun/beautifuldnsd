DNS 加速代理
====

原理介绍
----

为了上网，为了解析域名，我们需要 DNS 服务器所提供的服务。

通常，联网之后 ISP（互联网服务提供商，如中国电信、中国联通等）会分配一个他们自己的、经常是只服务于某个地区的 DNS 服务器地址。但是它们经常有以下问题：

1. 对不存在的域名进行劫持，返回自家的导航页面地址。对于浏览器来说还好点，毕竟用户一看就知道是怎么回事，虽然很讨厌就对了。而对于程序，以及非网页用途，这种功能就可能会导致问题了。比如火狐在地址栏直接输入不像域名的内容（如「google」），火狐会尝试添加常见前后缀，也会向搜索引擎询问是否有足够的自信判断用户想去哪里，或者直接显示搜索结果。（这个 dnsmasq 也可以解决）
2. 对于不常访问的域名经常返回服务器失败的信息，或者超时。虽然 ISP 的 DNS 服务器通常只服务很少的用户，但是它们经常处理不过来。
3. 有些 ISP 的 DNS 会返回奇怪的结果。比如对某些 IP 的 PTR 记录总是返回「localhost」。

用户也可以选择使用国内外的公共 DNS 服务器，比如 114.114.114.114 或者 Google 的 8.8.8.8。但是它们也有自己的问题：

1. 国外服务器的延迟比较高。
2. 即使支持相关 DNS 扩展，但是它们还是不能根据地域返回离用户最近的 DNS 节点。这对访问大型网站非常有用，比如淘宝、百度。以及一些使用第三方 CDN 服务的网站，比如使用又拍云、七牛的网站。

本程序为了综合不同 DNS 服务提供方的优点、避免它们的缺点，而使用了如下方案：

1. 处理掉已知的虚假应答
2. 向不同服务器并发发起请求，先收到谁的应答就使用谁的，但是 ISP 的 DNS 服务器享有一定的优先权。这样，如果 ISP 的 DNS 服务器能够及时响应，那么就会使用它的结果，否则使用公共 DNS 服务器的结果。有需要使用 CDN 的网站通常用户访问非常多，ISP 的 DNS 服务器能够及时响应这些请求。

配置及使用
----

本程序应当在实际需要访问的地方运行，比如运行于用户的电脑上，或者用户的网关/路由器上。

本程序需要预先安装以下软件：

1. Python 3.4 或以上
2. Python 的 dnslib 库

Arch Linux 用户可以直接从 AUR 安装「beautifuldnsd」软件包（感谢 lilydjwg 的打包）。

本程序的配置请见「beautifuldnsd.yaml」中的注释。

使用「beautifuldnsd --help」来查看可用的命令行选项。

本程序不具有缓存功能。请配合 dnsmasq 使用以取得最佳效果。

测试结果
----

如下是某次的统计结果：

    <UDPDNSServer [UDP:127.0.0.1:53535]>: count/avg/mdev/failed: 59828/337.3/1210.0/986
    <UDPDNSClient [UDP:202.106.0.20:53]>: count/avg/mdev/win/timeout/failed: 61454/3137.1/261927.7/49926/5693/1075
    <UDPDNSClient [UDP:8.8.8.8:53]>: count/avg/mdev/win/timeout/failed: 20760/2314.5/3710.8/4520/3865/110
    <UDPDNSClient [UDP:192.168.1.1:53]>: count/avg/mdev/win/timeout/failed: 14455/2839.4/57274.5/3683/2689/0
    <UDPDNSClient [UDP:4.2.2.2:53]>: count/avg/mdev/win/timeout/failed: 7392/3215.4/4149.1/713/1985/19

总共处理了近六万 DNS 请求，其中有近一千个失败了。平均响应延迟是三百多毫秒，标准差是一千多。

第一个上游是 ISP 的 DNS 服务器，总共发送了六万多请求，超时的近六千，服务器返回失败的有一千多。平均响应延迟高达三秒多，标准差更是25万之甚（说明延迟波动非常大，有的请求能够很快返回结果，有的会等非常久）。本程序从这个 DNS 服务器返回了近五万的应答结果。

余下的也是类似地解读。

可以看出，本程序的平均响应时间比所有上游更稳定、也更迅速。

贡献
----

本程序为公有领域的作品，你可以自由地使用。欢迎提交补丁。

