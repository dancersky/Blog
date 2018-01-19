## 两种通用加密算法（MD5，RSA）使用



#### 概述

​	md5及RSA加密算法是我们比较常见的两种加密算法，也是经常使用到的。我主要是利用md5的C Lib库实现md5加密功能，使用openssl库API实现RSA加密。这里做个笔记，下次用到就可以直接使用了。

#### MD5

​	MD5消息摘要算法（英语：MD5 Message-Digest Algorithm），一种被广泛使用的[密码散列函数](https://zh.wikipedia.org/wiki/%E5%AF%86%E7%A2%BC%E9%9B%9C%E6%B9%8A%E5%87%BD%E6%95%B8)，可以产生出一个128位（16[字节](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82)）的散列值（hash value），用于确保信息传输完整一致。（摘自维基百科）

​	对于MD5加密，现在主要用于校验的一个功能，比如文件传输中，服务器预先给文件计算md5值，在客户下载时同时把md5值发送给客户，客户通过计算文件的md5值，比对两个值是否相等来判断文件下载是否成功。

​	这里是我的代码地址：[https://github.com/dancersky/md5_demo](https://github.com/dancersky/md5_demo).

#### RSA	

​	RSA加密算法是一种[非对称加密算法](https://zh.wikipedia.org/wiki/%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95)。在[公开密钥加密](https://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86)和[电子商业](https://zh.wikipedia.org/wiki/%E7%94%B5%E5%AD%90%E5%95%86%E4%B8%9A)中RSA被广泛使用。（摘自维基百科）

​	非对称加密需要两个密钥，一个公钥一个私钥。一般都是私钥是自己保管好不给任何人，公钥可以发布。对于明文，使用公钥加密则需用私钥解密，反之用私钥加密则需用公钥解密。

​	我这边主要是利用openssl的C库，实现了对明文的RSA加解密。主要环境在ubuntu14.04上。

​	这里是我代码地址以及公钥私钥生成的一些说明：[https://github.com/dancersky/rsa_demo](https://github.com/dancersky/rsa_demo).



​																    Peace&Love ---------By Sky

