---
title: "VPS Config"
date: 2022-12-30T10:36:23+08:00
tag: "vps"
---

之前购买的腾讯云的套餐还有一周到期了, 看了眼价格500/年, 属实是有点贵. 本着能嫖一年是一年的心态, 于是我又在网上开始找不同

vps厂商新用户的活动, 这次我选择了`ucloud`

- 配置

  ![image-20221230104440731](/image-20221230104440731-1672382812042-5.png)

  > 第一年83, 比腾讯云还贵点...





### windows ssh秘钥登录配置

购买时默认的用户名为`ubuntu`, 密码也是在购买时设置的



首先大体介绍一下`ssh`配置主要过程

- 客户端通过`ssh-keygen`生成私钥和公钥文件
  - 默认的加密算法是RSA
  - 默认的秘钥存放路径是: `C:\User\用户名\.ssh`
- 用户将公钥写入服务器的`/home/用户/.ssh/authorized_keys`文件
  - 假设想要以用户A的身份登录远程服务器, 就要将公钥写入`/home/A/.ssh/authorized_keys`文件



#### 实战

- 生成RSA秘钥, 包括公钥和私钥

  ```sh
  ssh-keygen 
  ```

  ![image-20221230105955626](/image-20221230105955626-1672382804439-3.png)

  默认生成的秘钥文件为`id_rsa`(私钥) 和`id_rsa.pub`(公钥)

- 将RSA公钥拷贝到`/home/ubuntu/.ssh/authorized_keys`文件

  ![image-20221230110229088](/image-20221230110229088.png)

- 测试配置是否成功

  ![image-20221230110414506](/image-20221230110414506-1672382791570-1.png)

  可以发现直接免密码登录成功, 说明秘钥配置成功



### VScode配置文件编写

通常我会使用`vscode`的`remote-ssh`功能直接登录远程服务器, 这需要编写`C:\User\.ssh\config`

![image-20221230110623957](/image-20221230110623957.png)





大功告成!!





- 参考
  - [SSH秘钥登录](https://wangdoc.com/ssh/key)
  - [Windows 10 OpenSSH Equivalent of ssh-copy-id](https://chrisjhart.com/Windows-10-ssh-copy-id/)

