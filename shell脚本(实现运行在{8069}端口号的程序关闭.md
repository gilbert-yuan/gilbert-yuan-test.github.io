
## 第一个shell脚本-实现运行在8069端口号的程序关闭

```shell
#!/bin/bash  
:<< '文件头。用来设定一个文件的执行的路径或者是环境  '
loop_index=1
b=10
a=0
c=1
for loop in $(lsof -i:8069)  :<< ' 获取终端中执行 lsof -i:8069 返回的内容 |在这个端口执行的程序的内容 ’
do
    val_chu=`expr $loop_index / $b`    
    val_yu=`expr $loop_index % $b`
    echo " $val_chu The vaule $loop"
:<< ' 获取的内容被空格分割， 每一个子元素 循环赋值给 loop’
    if [[ $val_chu -gt 0 && $val_yu -eq 1 ]]
    then
    echo $(kill -9 $loop)
    fi
    let "loop_index++" 
done
```
