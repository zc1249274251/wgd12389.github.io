---
layout: post
title: 聊一聊互联网安全架构
category: 技术
tags: [tech]
keywords: 安全,架构
description: 聊一聊互联网安全架构的一二三事
---

## web端安全    

在过去的5年到10年之前，互联网一直是web端天的天下，也就是我们常说的一个html页面或者说是一个网站。那么我们今天就聊一聊从互联网发展历程至今安全架构的演变。   

在网页横行的天下里，相信之前做过网站，或者接触过网站开发，或者了解http协议的都应该听说过一个词,那就是`session`。  

什么是`session`不是我们本文的重点,我们这里要讲述的是传统的web网站架构是怎么通过`session`来确保**架构安全性**的?   

那么归结于实现的方式，传统的网站基本都是如下的做法：  

1. 用户带着账号和加密过后的密码去登录网站    
2. 服务器处理登录请求    
3. 如果错误,告知用户  
4. 如果正确，吧userId放到对应的session里   
5. 每次操作校验一下用户session里的userId是否还在，session是否断开   
6. 每次操作都是去session里取对应的用户信息，来判断操作的用户   


## APP架构安全   

### auth协议  

上面就是传统web架构下建立安全架构的方式,但是时过境迁,我们又赶上了互联网发展的大热潮，这一次，我们是移动客户端的发展，也可以叫做移动互联网的浪潮。  

那么这时候我们做过移动端后台开发的同学都会知道,我们是不会使用session的,也就是说我们的后台是一个无session会话的server服务。那么这个时候我们的问题就来了，我们怎么认证对应的每次请求是来自于哪一个用户呢.   

相信研究过session的朋友一定知道cookie的和session的关系,没错,就是每次请求,都是通过http请求中的cookie里的一个值来找到对应的session的,这个cookie的值可以被我叫做服务器给客户端颁发的令牌。     

怎么去理解这个令牌呢，相当于我有一个需要保密的大门,那么我为了保证不让人随便进来,我给我的小弟们一个一个对应的口令，如果这个人不在我手下了，那么我在我自己的认证端吧他的口令删除掉就好了。     

相信大家已经猜到我们要登场的角色了:没错，上面就可以理解为一个简单版的基于token的auth授权协议.现在大多数的移动服务端都是基于此种方式来构建自己的安全架构的.   

### https  

在看完auth协议的实现架构之后，有心的朋友应该发现问题了，这丫的数据不是安全的啊。  

问题出在哪里了呢，就是我在对你的网络劫持了之后，我用抓包软件去抓你的请求，就能拿到你的token，那么我就可以代替你的token来做请求了   

卧槽，这么一听好危险，那么我们怎么解决这个问题？   

非常简单。。。。把我们的服务器变成https的就好了，那么https是怎么上面的安全问题的呢？  

让我们先来了解一下https吧

> HTTPS（全称：Hyper Text Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 它是一个URI scheme（抽象标识符体系），句法类同http:体系。用于安全的HTTP数据传输。https:URL表明它使用了HTTP，但HTTPS存在不同于HTTP的默认端口及一个加密/身份验证层（在HTTP与TCP之间）。  

恩，我想大概你也懂了，其实https就是给我们所有的请求参数蒙上一层蒙娜丽莎的面具。  

### 签名验证   

蒙娜丽莎的微笑是那么迷人，谁又能不陶醉呢？为了一观那神秘的微笑，我用尽了全身的力气。   

好了不装逼了，相信用过Charles的朋友，大概应该都能知道，他有个功能是也能抓到https的请求包，那么我们https或许也不是那么安全了，妈个鸡的，这可怎么办，吓死宝宝了。   

还好本宝宝天资聪慧，想出了一个很好的解决办法： 参数签名。   

那么什么是参数签名呢？他又能做什么呢？   

先说一下怎么去做这个参数签名吧。   

**我举个例子：**  
我有一个请求是这样的`https:aaa.com?a=1&b=2`   
那么相信你也看到了我请求中的参数是`a=1&b=2`,我这时候吧这两个参数加上一个字符串`我最帅`，变成了这样`a=1&b=2我最帅`,然后这个拼接好的字符串就行md5啊，或者sha1之类的单向加密,然后吧单向加密的结果做为一个参数`sgin=sdasfsdfw`拼接到之前的请求里  
然后我服务器在接受到请求的时候,吧除了sign之外的参数做和服务器一样的操作，也就是生成签名字符串，然后和传过来的`sgin=sdasfsdfw`签名结果进行对比，如果是相等的说明这个请求是正确的。    

那么我签名这个有啥用？这还要我说吗？当然是防止内容篡改，防止伪造请求。  

**另外请注意一点**：你一定要确保那个sign签名时候的字符串的(例如之前的`我最帅`)不被他人知道,例如我们的安卓客户端可以吧这个打到so库里。   

### 非对称加密的艺术   

我靠，这时候搞android逆向的朋友不服了,你丫就算给我打到so库里，劳资照样给你弄出来，一个小小的签名就想难倒我？没门。  

我这时候呵呵一笑，出来混的，没俩下子怎么好意思装这个逼呢，对不对？  

那么我这时候签名都靠不住了，我咋办，嗯哼，不知道朋友听没听过非对称加密。   

yeah，我们这里就是要用这种技术，这里关于对称和非对称加密的相关知识不是本文的重点，故不再多提，感兴趣的朋友可以自行谷歌。我们这里还是要说一下，怎么用这种加密方式提高安全性。  

用过非对称加密的朋友知道，这种方式有2个钥匙，一个叫公共钥匙，一个叫私有钥匙，我们一般是吧公钥暴漏在外，私钥存在自己的包里。  

在移动端的架构一般都是这样的，生成公钥和私钥，吧公钥放在客户端存储，或者客户端去拉取。然后把私钥放在基本处于安全状态的服务器端（如果你说黑客能拿到这属于服务器安全问题好伐），然后我们在每次请求的时候可以吧参数用公钥加密，然后在服务器端解密，拿到真正的参数。   
   
用了这种方式之后，保证你自己的代码，你自己都不知道咋个搞丫，就算你丫牛逼到吧路由器劫持了，也休想知道我做了啥子撒，哈哈哈哈哈。   
