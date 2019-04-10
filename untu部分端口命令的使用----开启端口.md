## ubuntu部分端口命令的使用----开启端口/开启防火墙 
转载自[程序小工](https://www.cnblogs.com/zqunor/p/6417938.html)

1.测试远程主机的端口是否开启（windows命令行下执行）
 
```shell
sudo ufw status
```

 

3.打开80端口(ubuntu下执行)
```shell
sudo ufw allow 80
```

 

4.防火墙开启(ubuntu下执行)
```shell
sudo ufw enable
```
　　

5.防火墙重启(ubuntu下执行)
```shell
sudo ufw reload
```
