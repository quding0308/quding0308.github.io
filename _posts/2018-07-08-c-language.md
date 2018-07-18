---
layout: post
title:  "c语言基础"
# date:   2018-07-09 10:000:00 +0800
categories: blog
---

相关文档：
GNU C 文档：
https://www.gnu.org/software/libc/manual/html_mono/libc.html
 
基础学习：
http://wiki.jikexueyuan.com/project/c/c-intro.html
http://akaedu.github.io/book/index.html

内存：
http://www.cnblogs.com/yif1991/p/5049638.html
http://blog.csdn.net/wind19/article/details/5964090

c语言没有引用
http://blog.csdn.net/haoxinqingb/article/details/7716856

c语言中的指针    
http://blog.chinaunix.net/uid-22889411-id-59688.html
errno的使用
errno是全局变量，可以把最后一次调用c函数出现错误是的代码保留下来。如果最后一次调用c函数成功，则errno不会变。因此，只有在c函数返回值异常时，才检测errno。


    // tcp连接服务器
    int error = connect(clientSocketId, (struct sockaddr *)&peerAddr, addrLen);
    // 根据code返回错误描述
    char *str = strerror(errno);    // "Operation timed out"
    参考：http://wiki.jikexueyuan.com/project/c/c-error-handling.html
 
