1. rm -f webmagic.log  删除文件

2. java -cp geek-calendar-spider.jar com.geek.calendar.spider.spilder.FatePageProcessor  【运行main  關閉連接窗口會導致程序停止執行】

3. "&" 【符号结尾表示，让程序在后台运行，但是關閉鏈接窗口會導致程序停止】

4. nohup 【命令可以阻止窗口关闭是的挂断信号，使程序继续运行】

5. nohup java  -cp geek-calendar-spider.jar com.geek.calendar.spider.spilder.FatePageProcessor &   【讓程序在後臺執行，關閉窗口時程序繼續執行； 命令執行會返回一個進程號，記住此進程號，可以使用kill -9 進程號殺死它】（3600）

6. nohup java -jar xxx.jar &    【也可以這樣】

7. nohup java -jar xxx.jar >logs.txt &   【>logs.txt 設置日志輸出目錄，不設置為程序根目錄】

8. 只记录异常日志:     nohup java -jar xxx.jar >/dev/null  2>error.log    &

9. 不記錄日志： nohup java -java  xxx.jar  >/dev/null  2>&1  &

10. ```
    /dev/null 属于字符特殊文件，属于空设备，它是一个特殊的设备文件，会丢弃所有一切写入其中的数据，写入它的内容都会永远丢失。一般会把/dev/null当成一个垃圾站，所有不需要的信息丢进去。
    
    Linux的重定向
        0：表示标准输入；
        1：标准输出,在一般使用时，默认的是标准输出；
        2：表示错误信息输出
        
     nohup python -u Job.py >/dev/null 2>error.log  & 
        表示将Job.py程序的错误信息输出到error.log 文件，其他信息丢进/dev/null。
    
    nohup python -u Job.py >/dev/null  2>&1 & 
    	表示将Job.py程序的错误信息重定向到标准输出，其他信息丢进/dev/null。
    ```

    

11. ps -ef|grep xxx.jar  【查看進程號PID】

12. ps -ef|grep 4283  通過pid查看進程詳細信息

13. Kill -9 4283 殺死PID 為4283的進程

14. df  查看磁盤大小   

15. df -Lh  以更優化的方式查看磁盤大小   

16. df -a 是全部的文件系统的使用情况

17. 查看某個進程有多少個綫程  pstree -p PID | wc -l

18. ps -mp 10469  查看綫程詳細

19. jstack pid 查看堆棧信息

20. ps -mp <pid> -o THREAD,tid,time | sort -k2r   按照cpu占用情況排序







```
 nohup java -cp yscy-196.jar com.geek.calendar.spider.spilder.YiShengCaifuProcessor 1962-09-29_14:00:00  >/dev/null  2>log/error196.log  &







```



