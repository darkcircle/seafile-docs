# 安全机制

## 客户端和服务器间的通信加密

Seafile 在服务器配置了 HTTPS 后，客户端会自动使用 HTTPS 协议和服务器通信。

## 加密资料库如何工作？

当你创建一个加密资料库，你将为其提供一个密码。所有资料库中的数据在上传到服务器之前都将用密码进行加密。

加密流程：

1. 生成一个32字节长的加密的强随机数。它将被用作文件加密键（“文件键”）。
2. 用用户提供的密码对文件键进行加密。我们首先用PBKDF2算法从密码中获取到一个键/值对，然后用AES 256/CBC来加密文件键，所得结果被称之为“加密的文件键”。加密的文件键将会被发送到服务器并保存下来。当你需要访问那部分数据，你可以从加密的文件键中解密出文件键。
3. 所有的文件数据都将用AES 256/CBC加密的文件键进行加密。我们用PBKDF2算法从文件键中获取键/值对。加密完成后，数据将会被传送到服务器端。

上述加密过程即可在桌面客户端执行也可在网站浏览器中执行。浏览器端加密功能可在服务器端使用。当你从加密的资料库中上传和下载时，如下过程将会发生：

* 服务器端发回加密的数据，浏览器将会在客户端用JavaScript解密它们。
* 浏览器在客户端用JavaScript加密后，将加密后的数据发回服务器。服务器端直接保存加密后的结果。

在上述过程中，你的密码将不会在服务器端传输。

当你同步一个加密资料库到桌面客户端或者在网站浏览器中浏览一个资料库，桌面客户端/浏览器需要确认你的密码。当你创建一个资料库，一个“魔力标志”将会在密码和资料库id中获得。这个标志和资料库一起存储到服务器端。客户端用这个标志检查你的密码是否正确在你同步和浏览资料库之前。魔力标志是通过PBKDF2算法经过1000次迭代产生，所以它将非常安全抵抗蛮力破解。

为了最大安全性，纯文本的密码也不会保存在客户端。客户端只保存从“文件键”获得的键/值对，它用来解密数据。所以如果你忘记密码，你将不能恢复和访问服务器端的数据。

