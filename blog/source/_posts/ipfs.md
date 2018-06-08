---
title: ipfs：星际文件系统
---

#### 写在前面

比特币（Bitcoin）是区块链中最火最大的一个项目，其原因就是因为去中心化（Centralization），或者我们可以认为，是“多中心化”，因为点对点（peer to peer）的操作，是任何一个点（peer），都成为了中心，当每点都成为了中心，也就可以粗略的认为没有了中心，因为中心太多了，也正是由于中心太多，所以这个项目不会受制与某一个人或某一个团体，不会被少部分人控制，大家的权利和义务都是一样的，保障了绝对的公平。
上面是比特币的特点，然后人们根据比特币的这个特点，从底层的技术中，提取出来了区块链（block chain）技术，也就造就了现在繁荣的区块链生态。

#### http

说了那么多废话，其实就是想说，比特币，是区块链技术在货币上引用的产物，当然，区块链还可以运用到我们生活中的各行各业，现在我主要想说的，就是区块链运用到电脑传输协议上的产物，ipfs。

要说一个东西好，那肯定是有比较物的，比如我要说ipfs的优点，那么我就说出ipfs之前的http有什么缺点，并且ipfs怎么解决的这些问题，两者相比较，才能知道具体的好与不好。

所以我现在先说下http，作为tcp/ip的应用层协议，http无疑是很成功的，优点也有众多，有了http，才有浏览器，才有网站，才有客户端，才有各种有趣的网页与应用，所以说http是当今互联网世界的根基，也毫不为过，互联网公司将内容数据放到服务器上，普通用户通过浏览器、客户端等各种形式的终端来访问服务器，获取这些数据，这一过程就是我们常说的“上网”。在此过程中，这些数据的传输都是遵照HTTP的定下的标准。所以说，当前的互联网是运行在HTTP的规则之下的，这就是http协议的优点。

但是很不幸，它遇上了区块链这个划时代的东西，所以从区块链开来，http有一个很严重的弊端，就是服务器，这是一个很明显的中心，在区块链中，是不允许出现的情况，为什么呢？结合当下的社会事情来说，比如说“X涵段子”，“X走大事件”等的下架就是这个问题，谁让它们下架的？当然是相关部门，那相关部门怎么让他们下架的呢？前面就说了，你网站中的所有资源文件，客户端中的资源文件，都放在服务器上，所以要控制这个网站，只要控制住这个网站的服务器就好了，难吗？不难，如果很难的话，天朝这种只会扫黄打非的police，是肯定解决不了的。当然，除了服务器这个明显的中心节点后，还有网络拥堵，DDOS攻击（参照最近阮一峰老师博客事件）等问题，所以当区块链技术遇上网络通信协议，ipfs就出来了。

#### ipfs

ipfs的终极目标：要革http的命。

当然，这只是个很大的口号，就像小时候我想考北大，但是最终去了北大青鸟一样，毕竟“人有多大胆，地有多大产”，之前在炒币时，大家整天想着都是百倍币，千倍币，像我这种最高的时候才赚20%的人，说出来是会丢了炒币人的面子的，毕竟在很多人眼里，数字货币是未来世界的通行证，法币就是一文不值的草纸，我觉得这就有点过激，有点浮躁了，这就像我们当初高喊共产主义，三五年赶超英美，迈向共产主义社会一样，或许人类社会最重发展进步，会走到共产主义社会，不过可能会是“一万年”吧。

扯远了，还是说会ipfs吧，从简单开始说，bt下载估计大家都用过吧，尤其是男生，而且bt下载是封锁不了的，它没有服务器的概念，是从你电脑里直接传输东西给我，没有经过服务器等大型中心节点，比如银行可以禁止我转账给你100元，但是绝不能禁止我当面给你一百元一样，完全不可能禁止住，而bt下载技术又是十分成熟的技术，只是将bt下载里，文件以torrent形式来传播改为了ipfs里文件系统的组织形式。

另一个区别是，内容的传输方式不一样。IPFS的传输方式不像HTTP那样依赖骨干网络，而是通过用户间的连接来传输。每个用户都是一个微型节点，这些节点连接到一起，形成一个立体的拓扑网络，数据从一个节点到另一个节点，跟跑接力赛一样。就算某个节点出错了，那马上换另一个节点走就成。而传统的HTTP路线则是树状的，有关键节点存在，所以很容易被监控或者攻击。

#### start

说了半天废话，作为一个技术人员，我只想说“talk is cheap,show me the code”。

ipfs的工作原理：IPFS 为每个文件根据其内容计算出一个唯一的HASH，每个节点储存部分文件数据以及确定每个文件所在节点位置的分布式哈希表。用户访问文件时提供文件的 HASH，依据哈希表找到文件并返回文件数据。与 CDN 不同的是，每个 IPFS 节点都是一个数据源，不需要再从中心服务器获取数据。

btw，Filecoin 是运行在 IPFS 上的激励层。Filecoin 提供一个巨大云储存市场，使用者支付一定的金额来获得分布式的存储服务，而矿工将自己的机器作为 Filecoin 中的节点存储数据，来获得工作证明，以此来激发更多的人来加入 IPFS 的节点网络中。你可以在gate.io中买到～～～～～～

##### install

先打开[ipfs官网](https://ipfs.io/docs/install/)，下载，当然，我们优越的天朝早就把这么危险的网址给墙了，这么危险的东西，还是留着去资本主义吧，所以你要翻墙才能下载，这里就不多赘述了。



下载后记得解压，然后执行里面的 install.sh 文件，执行后你可以通过运行 ipfs help来测试是否安装成功。



##### 创建节点

运行 ipfs init 命令，将会自动创建 ipfs 节点，创建成功的话，将会有如下提示信息：

```javascript
initializing IPFS node at /Users/Morning/.ipfs
generating 2048-bit RSA keypair...done
peer identity: QmPh21fdkT9k3SCa83GMdErTwWAxyy3dcvUZiYYF2E5BDe
to get started, enter:

    ipfs cat /ipfs/QmPh21fdkT9k3SCa83GMdErTwWAxyy3dcvUZiYYF2E5BDe/readme
```

ipfs 的所有信息就储存在了 /Users/Morning/.ipfs 这下面。



##### 运行节点

````js
$ ipfs daemon
````

执行daemon命令，启动 ipfs 的守护进程，将本地节点连入ipfs节点网络，同步节点数据，参与数据的储存与分发。执行后将会显示如下信息：

```
Initializing daemon...
Swarm listening on /ip4/127.0.0.1/tcp/4001
Swarm listening on /ip6/::1/tcp/4001
Swarm listening on /p2p-circuit/ipfs/QmPh21fdkT9k3SCa83GMdErTwWAxyy3dcvUZiYYF2E5BDe
Swarm announcing /ip4/127.0.0.1/tcp/4001
Swarm announcing /ip6/::1/tcp/4001
API server listening on /ip4/127.0.0.1/tcp/5001
Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
Daemon is ready
```

这下，你就在 ipfs 网络中了，你自己就是其中的一个节点，你可以自己上传或者下载文件了。



##### 添加文件

命令很简单，一句 “ ipfs add 文件 ” 就好了，比如我当前目录下有个 readme.md 文件，里面就一句 hellp ipfs，

然后我在终端里输入：

```
$ ipfs add readme.md

// 终端返回如下
added Qmc6XfTRVX8FnYhMJ9wdgyq87qhgSZubTxUDYU4aX8m2ps readme.md

// 说明本地 ipfs 网络已经加入了这个 readme.md 文件，并且其对应的 ipfs 里的hash值是Qmc6XfTRVX8FnYhMJ9wdgyq87qhgSZubTxUDYU4aX8m2ps，若你的daemon进程正在运行的话，本地的数据将会被自动同步到ipfs网络。

// 接下来我们通过cat名了来验证文件
ipfs cat Qmc6XfTRVX8FnYhMJ9wdgyq87qhgSZubTxUDYU4aX8m2ps
// 输入将会如下
# hello ipfs

// 我们可以使用 IPFS 官方提供的 HTTP 网关加文件 HASH 来访问文件。
// 官方: https://ipfs.io/ipfs/<HASH>
// 代理: https://ipfs.infura.io/ipfs/<HASH>
// 如果要上传文件夹，就用add -f 文件夹路径 就好了，这里就不多说了
```

##### 修改文件

在 IPFS 上其实并不存在修改文件这么一说，只要你将文件上传至 IPFS 网络后，这份文件将一直保存在 IPFS 网络中，除非所有保存该数据的节点都丢失了该数据。如果想修改文件，只能重新使用 add 命令来上传修改后的文件，生成一个新的 HASH。



any problem else？



我发布了一个网页，地址是xxx，然后我修改了这个网页，再发布，地址就变了。。。万一网站的用户有一千万，你需要去通知一千万用户？？？



为了解决这个问题，我们可以使用 IPNS (Interplanetary Naming System) ，IPFS内部的域名系统， 将新的 HASH 与自己的节点 ID 绑定。使用 ipfs name publish 命令： 

```
$ ipfs name publish QmTQLbdGA9pQcwj7Wgf17AYAUcZcEyuijftupz1YCcf9Mn
Published to QmbBsAzF69zvuseNgsE1t8wPYnJrXaYTpt6z6Kitsnc7ZA: /ipfs/QmTQLbdGA9pQcwj7Wgf17AYAUcZcEyuijftupz1YCcf9Mn

// 然后不管你后面怎么更新文件，域名都只是一个，在有了固定的地址后，我们还可以通过反向代理或者修改 DNS TXT 的方式来绑定自己的域名。
https://ipfs.infura.io/ipns/QmbBsAzF69zvuseNgsE1t8wPYnJrXaYTpt6z6Kitsnc7ZA/
```

##### other

ipfs上的其他有趣的项目：https://github.com/ipfs/awesome-ipfs#apps



也许在不久的将来 IPFS 真的能成为一个流行通用的网络协议，真正实现一个更快更安全以及更加开放的网络。 作为历史的见证者，我们可以在 IPFS 上创造并分享属于我们的永恒的时刻。






















