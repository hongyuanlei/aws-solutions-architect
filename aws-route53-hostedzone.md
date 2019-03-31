#### Route53
> Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service. You can use Route 53 to perform three main functions in any combination: domain registration, DNS routing, and health checking.

Amazon Route 53是一种高度可用且可扩展的域名系统（DNS）Web服务。 你可以使用Route 53以任意组合执行三个主要功能：**域名注册**，**DNS路由**和**运行状况检查**。

#### Hosted Zone

> A hosted zone is a container for records, and records contain information about how you want to route traffic for a specific domain, such as example.com, and its subdomains (acme.example.com, zenith.example.com). A hosted zone and the corresponding domain have the same name.

一个Hosted Zone是Records的一个容器，Records含有关于你想怎样将流量导向特定域名的信息，例如`example.com`和它的子域名(`acme.example.com`和`zenith.example.com`)。一个Hosted Zone和它相应的域名拥有相同的名字。

#### 划分策略

以豆瓣网站为例，以下是豆瓣域名和其子域名的关系：
```
douban.com
|
|-- www.douban.com
|
|-- movie.douban.com
|
|-- music.douban.com
```

我们假设豆瓣的服务会部署在AWS Cloud的三个环境上，并且这个三个环境在不同的AWS Account上：

- Dev (开发环境)
- Staging (类生产环境)
- Production (生产环境)

同时，我们要求Staging和Production环境都支持Region级别的高可用性，服务需要同时在AWS Singpapore Region和AWS Tokyo Region上部署。那么服务的部署环境有：

- Dev (开发环境)
- Staging Tokyo (类生产环境)
- Staging Singapore (类生产环境)
- Production Tokyo(生产环境)
- Production Singapore(生产环境)

#### Production HostedZone 划分

基于这样的假设，我们来划分一下Hosted Zone。首先，我们要在Production的Route53中创建douban.com Hosted Zone:

Name                          | Type                                  | Value
----------------------------- | ------------------- | ------------------------------
douban.com                |  NS                  |  ns-283.awsdns-35.com. \| ns-1461.awsdns-54.org. \| ns-1837.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
douban.com                | SOA                |  xxx
www.douban.com        | CNAME          | sg.douban.com
www.douban.com        | CNAME          | tk.douban.com
sg.douban.com            | NS                  |  ns-271.awsdns-35.com. \| ns-1441.awsdns-54.org. \| ns-1817.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
tk.douban.com             | NS                  | ns-281.awsdns-35.com. \| ns-1451.awsdns-54.org. \| ns-1827.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
dev.douban.com          | NS                   | ns-253.awsdns-35.com. \| ns-1261.awsdns-54.org. \| ns-1637.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
staging.douban.com    | NS                   | ns-273.awsdns-35.com. \| ns-1361.awsdns-54.org. \| ns-1737.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
movie.douban.com      | CNAME           | movie.sg.douban.com
movie.douban.com      | CNAME           | movie.tk.douban.com
music.douban.com      | CNAME           | music.sg.douban.com
music.douban.com      | CNAME           | music.tk.douban.com

因为我们需要支持Singapore Region和Tokyo Region。所以，我们要在Production的Route53中创建tk.douban.com Hosted Zone:

Name                          | Type                | Value
----------------------------- | ------------------- | ------------------------------
tk.douban.com            |  NS                  |  ns-281.awsdns-35.com. \| ns-1451.awsdns-54.org. \| ns-1827.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
tk.douban.com            | SOA                |  xxx
tk.douban.com            | A/CNAME       |  main site endpoint
movie.tk.douban.com | A/CNAME       |  movie service endpoint
music.tk.douban.com | A/CNAME       |  music service endpoint

和sg.douban.com Hosted Zone:

Name                          | Type                | Value
----------------------------- | ------------------- | ------------------------------
sg.douban.com            |  NS                  |  ns-271.awsdns-35.com. \| ns-1441.awsdns-54.org. \| ns-1817.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
sg.douban.com            | SOA                |  xxx
sg.douban.com            | A/CNAME       |  main site endpoint
movie.sg.douban.com | A/CNAME       |  movie service endpoint
music.sg.douban.com  | A/CNAME      |  music service endpoint

#### Staging HostedZone 划分

Name                                     | Type                                  | Value
-------------------------------------- | ------------------- | ------------------------------
staging.douban.com              |  NS                  |  ns-273.awsdns-35.com. \| ns-1361.awsdns-54.org. \| ns-1737.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
staging.douban.com              | SOA                |  xxx
staging.douban.com              | CNAME          | sg.staging.douban.com
staging.douban.com              | CNAME          | tk.staging.douban.com
sg.staging.douban.com         | NS                  |  ns-261.awsdns-35.com. \| ns-1341.awsdns-54.org. \| ns-1717.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
tk.staging.douban.com          | NS                  | ns-271.awsdns-35.com. \| ns-1351.awsdns-54.org. \| ns-1727.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
movie.staging.douban.com   | CNAME           | movie.sg.staging.douban.com
movie.staging.douban.com   | CNAME           | movie.tk.staging.douban.com
music.staging.douban.com   | CNAME           | music.sg.staging.douban.com
music.staging.douban.com   | CNAME           | music.tk.staging.douban.com

因为我们需要支持Singapore Region和Tokyo Region。所以，我们要在Staging的Route53中创建tk.staging.douban.com Hosted Zone:

Name                                       | Type                | Value
--------------------------------------- | ------------------- | ------------------------------
tk.staging.douban.com            |  NS                  |  ns-271.awsdns-35.com. \| ns-1351.awsdns-54.org. \| ns-1727.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
tk.staging.douban.com            | SOA                |  xxx
tk.staging.douban.com            | A/CNAME       |  main site endpoint
movie.tk.staging.douban.com | A/CNAME       |  movie service endpoint
music.tk.staging.douban.com | A/CNAME       |  music service endpoint

我们要在Staging的Route53中创建sg.staging.douban.com Hosted Zone:

Name                                       | Type                | Value
--------------------------------------- | -------------------  | ------------------------------
sg.staging.douban.com            |  NS                  |  ns-261.awsdns-35.com. \| ns-1341.awsdns-54.org. \| ns-1717.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
sg.staging.douban.com            | SOA                |  xxx
sg.staging.douban.com            | A/CNAME       |  main site endpoint
movie.sg.staging.douban.com | A/CNAME       |  movie service endpoint
music.sg.staging.douban.com | A/CNAME       |  music service endpoint

#### Dev HostedZone 划分
因为Dev环境无需做Region级别的高可用性支持，我们只将服务部署在一个Region中，所以只需要创建一个HostedZone即可。

Name                                     | Type                                  | Value
-------------------------------------- | ------------------- | ------------------------------
dev.douban.com                    |  NS                  |  ns-253.awsdns-35.com. \| ns-1261.awsdns-54.org. \| ns-1637.awsdns-38.co.uk. \| ns-682.awsdns-21.net.
dev.douban.com                    | SOA                |  xxx
dev.douban.com                    | A/CNAME       | main site endpoint
movie.dev.douban.com         | A/CNAME       | movie service endpoint
music.dev.douban.com         | A/CNAME       | music service endpoint


