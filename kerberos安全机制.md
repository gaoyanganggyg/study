# [从kinit到kerberos安全机制](https://www.jianshu.com/p/2039fe8c62a1)
>Kerberos（/ˈkərbərəs/）是一种计算机网络授权协议，用来在非安全网络中，对个人通信以安全的手段进行身份认证。这个词又指麻省理工学院为这个协议开发的一套计算机软件。软件设计上采用客户端/服务器结构，并且能够进行相互认证，即客户端和服务器端均可对对方进行身份认证。可以用于防止窃听、防止重放攻击、保护数据完整性等场合，是一种应用对称密钥体制进行密钥管理的系统。Kerberos的扩展产品也使用公开密钥加密方法进行认证。


## 基本描述
Kerberos使用Needham-Schroeder协议作为它的基础。它使用一个由两个独立的逻辑部分：认证服务器和票据授权服务器组成的"可信赖的第三方"，术语称为密钥分发中心（KDC）。Kerberos工作在用于证明用户身份的"票据"的基础上。

KDC持有一个密钥数据库；每个网络实体——无论是客户还是服务器——共享一套只有他自己和KDC知道的密钥。密钥的内容用于证明实体的身份。对于两个实体间的通信，KDC产生一个会话密钥，用来加密他们之间的交互信息。

## 协议内容
协议的安全主要依赖于参加者对时间的松散同步和短周期的叫做Kerberos票据的认证声明。 下面是对这个协议的一个简化描述，将使用以下缩写：

- AS（Authentication Server）= 认证服务器
- KDC（Key Distribution Center）= 密钥分发中心
- TGT（Ticket Granting Ticket）= 票据授权票据，票据的票据
- TGS（Ticket Granting Server）= 票据授权服务器
- SS（Service Server）= 特定服务提供端

客户端用户发送自己的用户名到KDC服务器以向AS服务进行认证。KDC服务器会生成相应的TGT票据，打上时间戳，在本地数据库中查找该用户的密码，并用该密码对TGT进行加密，将结果发还给客户端用户。该操作仅在用户登录或者kinit申请的时候进行。 客户端收到该信息，并使用自己的密码进行解密之后，就能得到TGT票据了。这个TGT会在一段时间之后失效，也有一些程序(session manager)能在用户登陆期间进行自动更新。 当客户端用户需要使用一些特定服务(Kerberos术语中用"principal"表示)的时候，该客户端就发送TGT到KDC服务器中的TGS服务。当该用户的TGT验证通过并且其有权访问所申请的服务时，TGS服务会生成一个该服务所对应的ticket和session key，并发还给客户端。客户端将服务请求与该ticket一并发送给相应的服务端即可。具体的流程请看下面的描述。

其在网络通讯协定中属于显示层。

简单地说，用户先用共享密钥从某认证服务器得到一个身份证明。随后，用户使用这个身份证明与SS通信，而不使用共享密钥。

## 具体流程
（注意：此流程使用了对称加密；此流程发生在某一个Kerberos领域中；小写字母c,d,e,g是客户端发出的消息，大写字母A,B,E,F,H是各个服务器发回的消息。）

首先，用户使用客户端（用户自己的机器）上的程序登录：
- 用户输入用户ID和密码到客户端。
- 客户端程序运行一个单向函数（大多数为杂凑）把密码转换成密钥，这个就是客户端（用户）的“用户密钥”(user's secret key)。
随后，客户端认证（客户端(Client)从认证服务器(AS)获取票据的票据（TGT））：

    - Client向AS发送1条明文消息，申请基于该用户所应享有的服务，例如“用户Sunny想请求服务”（Sunny是用户ID）。（注意：用户不向AS发送“用户密钥”(user's secret key)，也不发送密码）该AS能够从本地数据库中查询到该申请用户的密码，并通过相同途径转换成相同的“用户密钥”(user's secret key)。
    - AS检查该用户ID是否在于本地数据库中，如果用户存在则返回2条消息：
        - 消息A：Client/TGS会话密钥(Client/TGS Session Key)（该Session Key用在将来Client与TGS的通信（会话）上），通过用户密钥(user's secret key)进行加密
        - 消息B：票据授权票据(TGT)（TGT包括：消息A中的“Client/TGS会话密钥”(Client/TGS Session Key)，用户ID，用户网址，TGT有效期），通过TGS密钥(TGS's secret key)进行加密
    - 一旦Client收到消息A和消息B，Client首先尝试用自己的“用户密钥”(user's secret key)解密消息A，如果用户输入的密码与AS数据库中的密码不符，则不能成功解密消息A。输入正确的密码并通过随之生成的"user's secret key"才能解密消息A，从而得到“Client/TGS会话密钥”(Client/TGS Session Key)。（注意：Client不能解密消息B，因为B是用TGS密钥(TGS's secret key)加密的）。拥有了“Client/TGS会话密钥”(Client/TGS Session Key)，Client就足以通过TGS进行认证了。

然后，服务授权（client从TGS获取票据(client-to-server ticket)）：

- 当client需要申请特定服务时，其向TGS发送以下2条消息：
    - 消息c：即消息B的内容（TGS's secret key加密后的TGT），和想获取的服务的服务ID（注意：不是用户ID）
    - 消息d：认证符(Authenticator)（Authenticator包括：用户ID，时间戳），通过Client/TGS会话密钥(Client/TGS Session Key)进行加密
- 收到消息c和消息d后，TGS首先检查KDC数据库中是否存在所需的服务，查找到之后，TGS用自己的“TGS密钥”(TGS's secret key)解密消息c中的消息B（也就是TGT），从而得到之前生成的“Client/TGS会话密钥”(Client/TGS Session Key)。TGS再用这个Session Key解密消息d得到包含用户ID和时间戳的Authenticator，并对TGT和Authenticator进行验证，验证通过之后返回2条消息：
    - 消息E：client-server票据(client-to-server ticket)（该ticket包括：Client/SS会话密钥 (Client/Server Session Key），用户ID，用户网址，有效期），通过提供该服务的服务器密钥(service's secret key)进行加密
    - 消息F：Client/SS会话密钥( Client/Server Session Key)（该Session Key用在将来Client与Server Service的通信（会话）上），通过Client/TGS会话密钥(Client/TGS Session Key)进行加密
- Client收到这些消息后，用“Client/TGS会话密钥”(Client/TGS Session Key)解密消息F，得到“Client/SS会话密钥”(Client/Server Session Key)。（注意：Client不能解密消息E，因为E是用“服务器密钥”(service's secret key)加密的）。

最后，服务请求（client从SS获取服务）：

- 当获得“Client/SS会话密钥”(Client/Server Session Key)之后，Client就能够使用服务器提供的服务了。Client向指定服务器SS发出2条消息：
    - 消息e：即上一步中的消息E“client-server票据”(client-to-server ticket)
    - 消息g：新的Authenticator（包括：用户ID，时间戳），通过Client/SS会话密钥(Client/Server Session Key)进行加密
- SS用自己的密钥(service's secret key)解密消息e从而得到TGS提供的Client/SS会话密钥(Client/Server Session Key)。再用这个会话密钥解密消息g得到Authenticator，（同TGS一样）对Ticket和Authenticator进行验证，验证通过则返回1条消息（确认函：确证身份真实，乐于提供服务）
    - 消息H：新时间戳（新时间戳是：Client发送的时间戳加1，v5已经取消这一做法），通过Client/SS会话密钥(Client/Server Session Key)进行加密
- Client通过Client/SS会话密钥(Client/Server Session Key)解密消息H，得到新时间戳并验证其是否正确。验证通过的话则客户端可以信赖服务器，并向服务器（SS）发送服务请求。
- 服务器（SS）向客户端提供相应的服务。

## 缺陷
- 失败于单点：它需要中心服务器的持续响应。当Kerberos服务宕机时，没有人可以连接到服务器。这个缺陷可以通过使用复合Kerberos服务器和缺陷认证机制弥补。
- Kerberos要求参与通信的主机的时钟同步。票据具有一定有效期，因此，如果主机的时钟与Kerberos服务器的时钟不同步，认证会失败。默认设置要求时钟的时间相差不超过10分钟。在实践中，通常用网络时间协议后台程序来保持主机时钟同步。
- 管理协议并没有标准化，在服务器实现工具中有一些差别。RFC 3244 描述了密码更改。
- 因为所有用户使用的密钥都存储于中心服务器中，危及服务器的安全的行为将危及所有用户的密钥。
- 一个危险客户机将危及用户密码

## 其他安全机制
### OAuth认证
>OAuth（Open Authorization，开放授权）用于第三方授权服务，现常用的第三方账号登陆都是采用该机制。比如我用github账号登陆LeetCode官网，LeetCode并不需要知道我的github账号、密码，它只需要将登陆请求转给授权方（github），由它进行认证授权，然后把授权信息传回LeetCode实现登陆。

### LDAP
>LDAP（Lightweight Directory Access Protocol，轻量级目录访问协议）是一种用于访问目录服务的业界标准方法，LDAP目录以树状结构来存储数据，针对读取操作做了特定优化，比从专门为OLTP优化的关系数据库中读取数据快一个量级。LDAP中的安全模型主要通过身份认证、安全通道和访问控制来实现，它可以把整个目录、目录的子树、特定条目、条目属性集火符合某过滤条件的条目作为控制对象进行授权，也可以把特定用户、特定组或所有目录用户作为授权主体进行授权，也可以对特定位置（如IP或DNS名称）进行授权。

### SSL
>SSL（Secure Sockets Layer，安全套接层）是目前广泛应用的加密通信协议，其基本思路是采用公钥加密法，即客户端先向服务器端索要公钥，然后用公钥加密信息，服务端收到密文后用自己的私钥解密。
