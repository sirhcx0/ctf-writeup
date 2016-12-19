# SharifCTF 7

距离上次打CTF已经有2年的时间了吧，环境准备不充分是一方面，另一方面是逆向的手法也略显生疏。虽然做出来的题目不多，但是再做题目的过程中，对于短时间内快速去解决问题的一种自我突破的感觉正是CTF的魅力所在。写一下做出来的题目的题解。


- [getit(50)](getit)
- [repairme(100)]()
- [SCrack(150)]() 
- [UnloadMe(200)]()
- [Login.apk(300)]()
- [Nanomites(300)]()
- [snake(400)]()


# getit(50)

这个题目从事后总结的角度去总结，讲道理就是让玩家去秒解的题目，低分值(50)的题目不会耽误你太多时间去解决。这道题我主要的时间花费在了找不到输入在哪里，当时就要哭瞎了，逆向的题一般套路是输入的字符串就是flag，而这个题连个输出的地方都没有，简直可怕。IDA看了下，写入文件后立即删除文件，文件内容是重点。所以断在写入后，删除前的时机，看下内存就是flag了。

![getit]()

待学习的姿势：gdb调试神器插件[gdb-depa](https://github.com/longld/peda)

# repairme(100)

直接运行崩溃，放进windbg里看是访问违规

```
(399c.2058): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=77043378 ebx=7efde000 ecx=00000000 edx=00884b76 esi=00000000 edi=00000000
eip=00884b76 esp=0032f86c ebp=0032f874 iopl=0         nv up ei pl zr na pe nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00010246
*** WARNING: Unable to verify checksum for image00880000
*** ERROR: Module load completed but symbols could not be loaded for image00880000
image00880000+0x4b76:
00884b76 ??              ???
```

CFF去看代码段没有可执行权限，给加上可直接权限即可运行成功，然后直接出flag

总结：考察PE文件格式

# SCrack(150)

这个题一点技术含量都没有。写了一段花指令，但是关键的逻辑没有挡住，直接IDA里一直R就能凑出来flag

总结：学习利用ida脚本自动化地清除一定格式的花指令，提取字符信息

#UnloadMe(200)
驱动题，要求能正确卸载驱动。明天向另外一个小伙伴请教下这题。

![unloadMe]()

#login.apk(300)

这题不难，但是从中间遇到的问题，解决问题的过程中学习到了2个小知识点。

首先，这题遇到的问题是，apk安装后跑不起来。解决方法是打开DDMS，设置包名的过滤器，看error级别的日志信息。拿着这个错误信息，可以去网上搜类似问题。

最后是因为这个apk需要加载2个so文件，但是打包进去的只有一个so文件，另外一个缺失了。不过不用慌，再java层调用的2个native函数，实际上在唯一的so中都有实现，所以简单的copy一份重命名下就可以运行了。

其次，静态分析时也遇到了一个问题，有个ARM指令一直不懂是什么意思，影响了分析。最后查资料也没看出来啥意思（捂脸，英语捉急）

![EOR]()

所以最后知道了Exclusive OR 是异或的意思...

```python
# 把输入的字符串循环和固定字符串(key)做异或操作，得到的结果做base64,再和一串明文比较。
# 解密base64之前的字符串s
s = base64.decodestring("fx1uagMGQQMWOWhyFBxnBUdzN35NPWYHUBQHRmozeEY=")
'\x7f\x1dnj\x03\x06A\x03\x169hr\x14\x1cg\x05Gs7~M=f\x07P\x14\x07Fj3xF'
# key的长度是19，这里的边界条件页是19，实际上异或一轮是20次，我在边界条件这里也吃亏了
key = "My_S3cr3t_P@$$W0rD"+'\x00'
key2 = key *2
for i in range(0,len(s)):
    flag += chr(ord(key2[i]) ^ ord(s[i]))
print flag
'2d190e30bf82080557734b543f425c8b'
```

