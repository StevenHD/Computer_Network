# 第18讲 | DNS协议：网络世界的地址簿

现在网站的数目非常多，常用的网站就有二三十个，如果全部用 IP 地址进行访问，恐怕很难记住。于是，就需要一个地址簿，根据名称，就可以查看具体的地址。

> 例如，我要去西湖边的“外婆家”，这就是名称，然后通过地址簿，查看到底是哪条路多少号。

---

### DNS 服务器

在网络世界，也是这样的。你肯定记得住网站的名称，**但是很难记住网站的 IP 地址**，因而也**需要一个地址簿**，就是 **DNS 服务器**。

由此可见，DNS 在日常生活中多么重要。**每个人上网，都需要访问它**。

一旦它出了故障，整个互联网都将瘫痪。

如果大家都**去同一个地方访问某一台服务器**，**时延**将会非常大。因而，**DNS 服务器**，一定要设置成**高可用、高并发和分布式**的。

有了这样**树状的层次结构**——

![img](https://static001.geekbang.org/resource/image/59/a6/59f79cba26904ff721aabfcdc0c27da6.jpg)

![image-20200621091700595](C:\Users\HLHDS\AppData\Roaming\Typora\typora-user-images\image-20200621091700595.png)

---

### DNS 解析流程

为了**提高 DNS 的解析性能**，很多网络都会**就近部署 DNS 缓存服务器**。

- 电脑客户端会发出一个 DNS 请求，问 www.163.com 的 IP 是啥啊，并发给本地域名服务器 (本地 DNS)。
- 本地 DNS 收到来自客户端的请求。你可以想象这台服务器上缓存了一张域名与之对应 IP 地址的大表格。
- 根 DNS 收到来自本地 DNS 的请求，发现后缀是 .com，说：“哦，www.163.com 啊，这个域名是由.com 区域管理，我给你它的顶级域名服务器的地址，你去问问它吧。”
- 权威 DNS 服务器查询后将对应的 IP 地址 X.X.X.X 告诉本地 DNS。

- **本地 DNS** 再将 IP 地址返回客户端，**客户端和目标**建立连接。

![img](https://static001.geekbang.org/resource/image/ff/f2/ff7e8f824ebd1f7e16ef5d70cd79bdf2.jpg)

---

### 负载均衡

站在客户端角度，这是一次 **DNS 递归查询过程**。

因为本地 DNS 全权为它效劳，它只要坐等结果即可。在这个过程中，DNS 除了**可以通过名称映射为 IP 地址**，它还可以做另外一件事，就是**负载均衡**。

> 还是以访问“外婆家”为例，还是我们开头的“外婆家”，但是，它可能有很多地址，因为它在杭州可以有很多家。所以，如果一个人想去吃“外婆家”，他可以**就近**找一家店，**而不用大家都去同一家**，这就是负载均衡。

DNS 首先可以做**内部负载均衡**。

> 例如，一个应用要访问数据库，在这个应用里面应该配置这个数据库的 IP 地址，还是应该配置这个数据库的域名呢？**显然应该配置域名**，因为一旦这个数据库，因为某种原因，换到了另外一台机器上，而如果有多个应用都配置了这台数据库的话，一换 IP 地址，就需要将这些应用全部修改一遍。但是如果配置了域名，则只要在 DNS 服务器里，将域名映射为新的 IP 地址，这个工作就完成了，大大简化了运维。

> 在这个基础上，我们可以再进一步。例如，某个应用要访问另外一个应用，如果配置另外一个应用的 IP 地址，那么这个访问就是一对一的。但是当被访问的应用撑不住的时候，我们其实可以部署多个。但是，访问它的应用，如何在多个之间进行负载均衡？只要配置成为域名就可以了。在域名解析的时候，我们**只要配置策略，这次返回第一个 IP，下次返回第二个 IP**，就可以实现负载均衡了。

另外一个更加重要的是，DNS 还可以做**全局负载均衡**。

> 为了保证我们的应用高可用，往往会部署在多个机房，每个地方都会有自己的 IP 地址。当用户访问某个域名的时候，这个 IP 地址可以轮询访问多个数据中心。如果一个数据中心因为某种原因挂了，只要在 DNS 服务器里面，将这个数据中心对应的 IP 地址删除，就可以实现一定的高可用。

另外，我们肯定希望北京的用户访问北京的数据中心，上海的用户访问上海的数据中心，这样，客户体验就会非常好，访问速度就会超快。这就是**全局负载均衡**的概念。

---

### 示例：DNS 访问数据中心中对象存储上的静态资源

假设全国有多个数据中心，托管在多个运营商，每个数据中心三个可用区（Available Zone）。对象存储通过跨可用区部署，实现高可用性。在每个数据中心中，都至少部署两个**内部负载均衡器**，内部负载均衡器后面**对接多个对象存储的前置服务器**（Proxy-server）。

- 当一个客户端要访问 object.yourcompany.com 的时候，需要**将域名转换为 IP 地址**进行访问，所以它要请求本地 DNS 解析器。

- 本地 DNS 解析器先查看看**本地的缓存**是否有这个记录。如果有则直接使用，因为上面的过程太复杂了，如果每次都要递归解析，就太麻烦了。

- 如果本地**无缓存**，则需要请求本地的 DNS 服务器。

- 本地的 DNS 服务器一般部署在你的数据中心或者你所在的运营商的网络中，本地 DNS 服务器**也需要看本地是否有缓存**，如果有则返回，因为它也不想把上面的递归过程再走一遍。

- 至 7. 如果本地没有，本地 DNS 才需要递归地**从根 DNS 服务器**，查到.com 的**顶级域名服务器**，最终查到 yourcompany.com 的**权威 DNS 服务**器，**给本地 DNS 服务器**，权威 DNS 服务器按说会返回真实要访问的 IP 地址。

对于不需要做全局负载均衡的简单应用来讲，yourcompany.com 的权威 **DNS 服务器**可以直接**将** object.yourcompany.com 这个**域名****解析为一个或者多个 IP 地址**，然后**客户端可以通过多个 IP 地址，进行简单的**轮询**，实现**简单的负载均衡**。

---

但是对于复杂的应用，尤其是跨地域跨运营商的大型应用，则需要更加复杂的全局负载均衡机制，因而需要专门的设备或者服务器来做这件事情，这就是**全局负载均衡器**（GSLB，Global Server Load Balance）。

两层的 GSLB，是因为**分运营商和地域**。我们希望不同运营商的客户，可以访问相同运营商机房中的资源，这样不跨运营商访问，有利于提高吞吐量，减少时延。

>  第二层 GSLB，通过查看请求它的本地 DNS 服务器所在的地址，就知道用户所在的地理位置，然后将距离用户位置比较近的 Region 里面，**六个内部负载均衡（SLB，Server Load Balancer）的地址**，返回给本地 DNS 服务器。

- 本地 DNS 服务器将结果返回给本地 DNS 解析器。
- 本地 DNS 解析器将结果缓存后，返回给客户端。

---

### 小结

- DNS 是网络世界的地址簿，可以通过域名查地址，因为域名服务器是按照树状结构组织的，因而域名查找是使用递归的方法，并**通过缓存的方式增强性能**；

- 在域名和 IP 的映射过程中，给了应用基于域名做负载均衡的机会，可以是**简单的负载均衡**，也可以**根据地址和运营商做全局的负载均衡**。