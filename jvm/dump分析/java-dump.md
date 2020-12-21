linux 下载dump文件：



1、获取应用的pid

使用ps -ef | grep java查询服务器上的java应用进程信息，找到应用进程及id

2、使用jmap获取dump信息

jmap -dump:format=b,file=/home/app/dump.out 17740

注：/home/app/dump.out表示生成的dump文件的存放地址及文件名，17740表示1中查询到的应用pid

3、分析dump文件

可使用jstack命令分析，也可以使用eclipse memory analyzer工具分析