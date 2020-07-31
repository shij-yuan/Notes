

# 常用命令

1. 查看进程、线程、某个进程的线程

    top -H [-p pid], ps -T [-p pid]

2. 查看磁盘的使用状况

    df -h 

3. 查看目录的使用状况

    du -sh 

4. 查看某个端口的使用状况

    lsof -i:8088 

5. 实时的查看日志文件。

    tailf -n 50 [fileName] 

6. 查看某个日志文件中的内容

    grep -i "xxx" file 

7. 取文件的前50行

    head -n 50 file 

8. 查看文件的多少行

    wc -l filename 

9. 分割一个文件，以1000行为一个文件

    spilt -l 1000 filename -d -a 5 




## 查看进程、线程

1. 利用进程名获取进程号

    ps -ef|grep syslog|grep -v "grep"|awk '{print $2}'

2. 利用进程号查看该进程下的线程

    ps -eLf|grep 722|grep -v "grep" / ps -T -p 722

3. 查看线程cpu利用率

    top -H -p 722

# 查看日志

1. 查看日志常用命令

    tail:  -n  是显示行号；相当于nl命令；例子如下：
    tail -100f test.log      实时监控100行日志
    tail  -n  10  test.log   查询日志尾部最后10行的日志;
    tail -n +10 test.log    查询10行之后的所有日志;

    head
    跟tail是相反的，tail是看后多少行日志；
    head -n 10  test.log   查询日志文件中的头10行日志;
    head -n -10  test.log   查询日志文件除了最后10行的其他所有日志;

    cat
    tac是倒序查看，是cat单词反写；例子如下：
    cat -n test.log | grep "debug"   查询关键字的日志



2. **应用场景一：按行号查看---过滤出关键字附近的日志**

    

    cat -n test.log |grep "debug"  得到关键日志的行号

    

    cat -n test.log |tail -n +92|head -n 20  选择关键字所在的中间一行。然后查看这个关键字前10行和后10行的日志:
    tail -n +92 表示查询92行之后的日
    head -n 20 则表示在前面的查询结果里再查前20条记录



3. **应用场景二：根据日期查询日志**

sed -n '/2014-12-17 16:17:20/,/2014-12-17 16:17:36/p'  test.log

特别说明: 上面的两个日期必须是日志中打印出来的日志,否则无效；
先 grep '2014-12-17 16:17:20' test.log 来确定日志中是否有该 时间点

4. **应用场景三：日志内容特别多，打印在屏幕上不方便查看**

- 使用more和less命令
    如： cat -n test.log |grep "debug" |more     这样就分页打印了,通过点击空格键翻页

- 使用 >xxx.txt 将其保存到文件中,到时可以拉下这个文件分析
    如：cat -n test.log |grep "debug"  >debug.txt

