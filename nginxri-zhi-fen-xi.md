## nginx日志分析

    #!/bin/bash



    pv_c=0
    uv_c=0
    for k in $(seq 1 28)
    do
       f=""
       if [ $k -lt 10 ]
       then
        f=access_2017060${k}.log
       else
        f=access_201706${k}.log
       fi

       cat ${f} | awk '{print $1 " " $7}' | grep /inspire/detail/ > pv_${f}

       pv=$(cat pv_${f} | sort | wc -l)
       uv=$(cat pv_${f} | sort | uniq -c | wc -l)

       echo ${f}    ${pv}   ${uv} 

       pv_c=`expr $pv_c + $pv`
       uv_c=`expr $uv_c + $uv`

       if [ $k -eq 14 ]
       then
        echo ${pv_c}
        echo ${uv_c}
        pv_c=0
        uv_c=0
       fi

    done

    echo ${pv_c}
    echo ${uv_c}

[https://my.oschina.net/362228416/blog/844713](https://my.oschina.net/362228416/blog/844713)

[http://longzhun.iteye.com/blog/2356748](http://longzhun.iteye.com/blog/2356748)

[http://codewiki.wikidot.com/shell-script:if-else](http://codewiki.wikidot.com/shell-script:if-else)

```
//按ip排序
//cut-c -b -f 分别对应character byte 和 filed截取方式，-f时要指定分割符号 -d
//cut -f1 -d " " 截取以空格分割的第一个部分
//sort -r 升序 -n以数值来排序
//sort -k 2 -t " " 以" "分割，用第二个来排序
//uniq -c 显示重复的次数，只检查相邻的是否重复，所以一般要sort先
cat access_20170601.log | awk '{print $1 " " $7}' | grep /inspire/detail/ | cut -f1,7 -d ' ' | sort | uniq -c | sort -g -r
cat access_20170601.log | awk '{print $1 " " $7}' | grep /inspire/detail/ | sort -k 1 -t ' '

//split的角标是从1开始的。。。蛋碎，这个是把url的?后面去掉
cat access_20170601.log | awk '{print $1 " " $7}' | grep /inspire/detail/ | awk '{split($2,b,"?"); print $1 " " b[1]}'
```

cat 看文件

grep 过滤文本

sort 排序

uniq 去重

awk 文本处理



http://www.cnblogs.com/FrankTan/archive/2010/03/01/1634516.html

http://blog.csdn.net/liuxiao723846/article/details/47754479

