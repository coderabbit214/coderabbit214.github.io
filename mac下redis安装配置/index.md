# mac下redis安装配置


# mac下redis安装配置

## 1.下载[Redis官网](https://redis.io/)

![WeChatc7f9a741d979963c0f4a68fbaf38938d](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChatc7f9a741d979963c0f4a68fbaf38938d.png)

## 2.放在可靠位置

例如：/usr/local/

![WeChat576cffbe86984930da288a38b50f2315](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChat576cffbe86984930da288a38b50f2315.png)

## 3.编译

```
cd /usr/local/redis
sudo make test
sudo make isntall
```

## 4.运行

```
redis-server
```

![WeChatbe0c44ee49c6e77755124eb31de4b2fa](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/WeChatbe0c44ee49c6e77755124eb31de4b2fa.png)

## 5.其他命令

```
- 停止redis server服务
redis-cli shutdown

- 启动redis客户端
redis-cli
```


