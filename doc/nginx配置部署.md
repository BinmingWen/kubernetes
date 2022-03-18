## nginx配置安装

### 一.nginx安装

#### 1.环境安装搭建

##### 1. 安装 gcc 环境

```
$ sudo yum -y install gcc gcc-c++ # nginx 编译时依赖 gcc 环境
复制代码
```

##### 2. 安装 pcre

```
$ sudo yum -y install pcre pcre-devel # 让 nginx 支持重写功能
复制代码
```

##### 3. 安装 zlib

```
# zlib 库提供了很多压缩和解压缩的方式，nginx 使用 zlib 对 http 包内容进行 gzip 压缩
    $ sudo yum -y install zlib zlib-devel 
复制代码
```

##### 4. 安装 openssl

```
# 安全套接字层密码库，用于通信加密
$ sudo yum -y install openssl openssl-devel
```

##### 5.检查环境是否正确

~~~
./configure --prefix=/usr/local/nginx
~~~

**配置正确环境图示**

![image-20220217152617897](https://jepusi-image.oss-cn-beijing.aliyuncs.com/img/image-20220217152617897.png)

### 二.nginx安装

##### 1.安装nginx

> ```js
> yum install -y nginx
> ```

##### 2.启动nginx

>```js
>systemctl start nginx.service
>```

##### 3.设置开机启动

>```js
>systemctl enable nginx.service
>```

##### 4.常用命令

~~~
~~~

##### 5.配置文件

~~~
自定义的配置文件放在/etc/nginx/conf.d
项目文件存放在/usr/share/nginx/html/
日志文件存放在/var/log/nginx/
还有一些其他的安装文件都在/etc/nginx
~~~

