# HTTPS加解密

**简介**
HTTPS是运行在SSL/TLS上的HTTP，HTTPS = HTTP（应用层） + TLS（会话层）。
HTTPS主要作用就是建立一个信息安全通道，来保证数据传输的安全，同时也可以确认网站的真实性。
目前运用最广泛的TLS是v1.2版本，最新的v1.3兼容了v1.2，且强大了很多。TLS由记录协议，握手协议，警告协议，变更密码协议，扩展协议等几个子协议组成，综合使用了对称加密，非对称加密，身份验证等多个前沿密码学技术。

**HTTPS优点**
确保数据安全
- 内容加密
- 身份认证

一定程度上防止中间人攻击，确保数据完整性
- 信息篡改
- 信息窃取
- 会话劫持

**HTTPS缺点**
Web页面响应变慢
证书申请需要付费



## 加密算法介绍

### 对称加密
加密密钥和解密密钥都是同一个，加解密速度快，密钥安全性和密钥长度正相关，常见的对称加密算法有AES、DES、3DES、RC4等
- 序列密码：
包含三个要素，一段无穷序列、明文、密文
加密的原理就是从无穷序列中选取一段和明文等长的序列，二者做异或运行，得到的即为密文；解密逻辑同理，只需让密文和上述序列再做一次异或运算即可得到原始明文。
![](https://img-blog.csdnimg.cn/20201216235004602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1dHVhbnR1YW4xMDAz,size_16,color_FFFFFF,t_70)
-
-
- 分组密码模式
将数据分块，以AES-128举例，将明文按128位（16字节）大小的块分割，针对每一个分块进行加密，数据长度不够一个块大小的就需要进行填充，常见的分组模式有ECB、CBC、CFB、GCM等
ECB：它是最简单的分组密码模式，相同的密钥对每个块都进行一次加密，因为分组密码是确定的，那么每个块输出的密文肯定是确定的。
CBC：CBC模式是为了解决ECB天生的确定性推出的，引入了一个初始向量IV，过程是首先生成一个随机IV，长度与加密块等长，然后明文第一块内容与IV进行异或操作后再进行加密，等到的密文会被作为下一块的IV使用，以此类推，这样每次加密操作都是同一个加密链条中的一部分。
![CBC模式](https://img2018.cnblogs.com/blog/422171/201909/422171-20190905171806386-1456272303.png "CBC模式")

### 非对称加密
它有两个密钥，一个是“公钥”，一个是“私钥”，两个密钥不同，所以“不对称”，私钥必须严格保密，公钥可以公开给任何人使用。
这里有个特别的“单向性”，公钥加密的数据只可以用私钥解密，反之私钥加密的数据只可以用公钥解密，前者可以用到加解密中，后者可以用到数据签名中。
非对称加密的常见算法有RSA、DSA、ECDSA等。

### 散列
散列函数是将任意长度的输入转化为定长输出的算法，常见的有SHA-1、SHA-256、MD5等
它有几个特性：
- 单向性：给定一个散列，计算上无法找到或者构造出生成它的消息
- 弱抗碰撞性：给定一条消息和它的散列，计算上无法找到一条不同的消息和它具有相同的散列
- 强抗碰撞性：计算上无法找到两条散列相同的消息
散列函数最常用的使用场合是以紧凑的方式表示并比较大量数据，为了避免直接比较两个大文档是否相同，可以直接对比两个文档的散列值


## 中间人攻击

### 被动网络攻击
监听收集双方的会话内容，
针对未加密的流量，直接就能从中获取明文数据并监听记录；
针对加密的流量，可以先保存抓取的加密数据，直到破解之后再使用。当前困难的事情在十年以后可能就会变得简单，随着时间的推移，计算机会越来越强大，越来越便宜，也有可能发现加密基元的弱点。TLS中最常见的密钥交换算法是基于RSA算法的，RSA密钥多年不变，一旦被破解，那么也就可以解密之前所有的会话内容了，也就是不支持向前保密。
被动攻击的效果非常好，这既是因为还有非常多的数据未被加密，也是因为大量收集信息的过程可以全自动化。

### 主动网络攻击
会主动改变数据流或影响双方会话。通常会攻击目标的身份验证系统，也就是攻击者会扮演公钥证书体系的角色，将伪造的证书发送给客户端，让客户端认为证书有效并接受证书。或者会向客户端发送无效证书，看客户端会不会强行无视证书警告。
主动攻击的攻击力非常强大，但更难扩展，难以实现自动化，需要投入更多资源来跟踪个体。重定向大流量也容易被发现，同样，大规模证书欺诈攻击也很难奏效，原因是应付这么多个人和组织对网站证书的跟踪监测非常困难。

## 通俗讲解https加解密过程
①：
我们要实现A能发一个hello消息给B，需要实现的是：
A发给B的hello消息包，即使被中间人拦截到了，也无法得知消息的内容，同时有且只有A和B有能力看到通信的真正内容。
![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwSWY7zP3ZEpfY9alKlo5k4hE6HHHzay9lbqiaP6s8QRPXZRoxRt8jpicNic262iaceibTlydYDxtLtjjg/640?wx_fmt=png)

②：
但是密钥协商的过程是需要加密的，否则会被中间人拦截，这时候就需要对协商过程进行加密。
我们引入非对称加密，客户端使用非对称加密的公钥加密对称秘钥，发送给服务端，并且也只有拥有私钥的服务端才能正确解密，这样就保证了协商密钥过程的安全。
![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwSWY7zP3ZEpfY9alKlo5k4AwRZkclo5EPCFiaNM5L7ZHp1hPz6T3UhRyZsYL3KxziaWMhmiaiagrNklg/640?wx_fmt=png)

③：
但是此方案需要客户端一开始就持有服务端的公钥
现在又需要解决的问题是：如果让A、B等客户端安全的得到公钥
方案1. 服务器端将公钥发送给每一个客户端
方案2. 服务器端将公钥放到一个远程服务器，客户端可以请求得到
我们选择方案1，因为方案2又多了一次请求，还要另外处理公钥的放置问题。
但是方案1存在被中间人掉包公钥的风险。如下图
![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwSWY7zP3ZEpfY9alKlo5k4so4DDYrEsdhwSuS205zrIXiaq6tBIL5oNIlYbpb7LdrF5aGcRApyg3Q/640?wx_fmt=png)

④：
公钥被调包的问题出现，是因为我们的客户端无法分辨返回公钥的人到底是中间人，还是真的服务器。这其实就是密码学中提的身份验证问题。
这时候引入证书的概念，证书中只有服务器交给第三方的公钥，并且这个公钥是被第三方机构的私钥加密后的
![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwSWY7zP3ZEpfY9alKlo5k4CZR09YlW8xSA25rb2icgibtHEviaIIbpsmOvS0FlT6sF6aBnj499jI5XA/640?wx_fmt=png)

⑤：
但是第三方机构不可能只给一家公司制作证书，也可能会给“中间人”这样的公司发放证书，这样的话中间人还是调包证书
![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwSWY7zP3ZEpfY9alKlo5k4MeZvmwMJKyOkfLeicI50ac4flkZAUqjkwQQWRdiavfXH6SI7tSt8hcwA/640?wx_fmt=png)

⑥：
这时候就需要客户端具备辨别同一机构下不同证书的能力，我们证书中添加证书签名。
两个证书编号一致，则表明证书是有效的
服务器域名一致则证明证书未被中间人调包
![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRwSWY7zP3ZEpfY9alKlo5k4AwyEZmY8NHAE7xn4ZuzyOY0D3eHbibtVWhlOvFHPcYBfRibmjyWRQrlA/640?wx_fmt=png)

ps：第三方机构的公钥客户端可以直接获得；
浏览器和操作系统都会维护一个权威的第三方机构列表（包括它们的公钥）。因为客户端接收到的证书中会写有颁发机构，客户端就根据这个颁发机构的值在本地找相应的公钥。

上述整个过程可以总结为：
HTTPS要使客户端与服务器端的通信过程得到安全保证，必须使用的对称加密算法，但是协商对称加密算法的过程，需要使用非对称加密算法来保证安全，然而直接使用非对称加密的过程本身也不安全，会有中间人篡改公钥的可能性，所以客户端与服务器不直接使用公钥，而是使用数字证书签发机构颁发的证书来保证非对称加密过程本身的安全。这样通过这些机制协商出一个对称加密算法，就此双方使用该算法进行加密解密。从而解决了客户端与服务器端之间的通信安全问题。


## 深入介绍TLS协议
https://www.processon.com/diagraming/5fda2d67e0b34d06f4fd3416

ClientHello:
--
    Handshake protocol: ClientHello
    	Version: TLS 1.2
    	Random
    		Client time: May 22, 2030 02:43:46 CMT
    		Random bytes: b76b0e61829557eb4c611adfd2d36eb232dc1332fe29802e321e871 
    	Session ID: (empty)
    	Cipher Suites
    		Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    		Suite: TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
    		Suite: TLS_RSA_WITH_AES_128_GCM_SHA256
    		Suite: TLS_ECDHE_RSA_WITH_AES_128_BC_SHA
    		Suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA
    		Suite: TLS_RSA_WITH_AES_128_CBC_SHA
    		Suite: TLS_RSA_WITH_3DES_EDE_CBC_SHA
    		Suite: TLS_RSA_WITH_RC4_128_SHA
    	Compression methods
    		Method: null
    	Extensions
    		Extension: server_name
    			Hostname: www.feistyduck.com
    		Extension: renegotiation_info
    		Extension: elliptic curves
    			Named curve: secp256r1
    			Named curve: secp384r1
    		Extension: signature_algorithms
    			Algorithm: sha1/rsa
    			Algorithm: sha256/rsa
    			Algorithm: sha1/ecdsa
    			Algorithm: sha256/ecdsa




ServerHello:
--
    Handshake protocol: ServerHello
    	Version: TLS 1.2
    	Random
    		Server time: Mar 10，2059 02:35:57 GMT
    		Random bytes: 8469b09b480c1978182ce1b59290487609f41132312ca22aacaf5012
    	Session ID: 4cae75c91cf5adf55f93c9fb5dd36d19903b1182029af3d527b7a42ef1c32c80
    	Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    	Compression method: null
    	Extensions
    		Extension: server name
    		Extension: renegotiation info


## 快借KMS加解密介绍（可选）
和HTTPS加解密不同的是，缺少了密钥协商这一过程
https://www.processon.com/view/link/5f8edcacf346fb06e1e095a8
