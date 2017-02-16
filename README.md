# adan_srv-v2-rc
# adan_srv-v2-rc 版本修正了几个地方，但是仍然不是一个完全的服务器软件系统，数据报和业务互交模块流程还没有完成，下列罗列出新增模块：
# 1.对任务队列抛出的socket 也同样用栈进行管理，修改栈，添加批量操作模块，从队列取出socket 集合后，压入栈，然后执行少于20 个socket 
#   任务后侦查一次任务队列，这样既可以减少对队列锁的访问，又可以避免IO 线程繁忙时长时间不提取任务队列，参生集聚后被一次取完，这时
#   提取这次任务的IO 线程就会彻底被喂饱，分配均匀性差。
#
# 2.增加了redis 内存型数据库模块并提供了使用样例，将模块嵌入static_val.h 中，成为全局变量，添加访问临界区，防止访问冲突。
#
# 3.重新启用数据报模块-服务器收到数据请求后：接收数据，解密，对比离线同步码确认数据存活的可行性，然后解析数据，执行业务操作（redis 操作），
#   然后计算离线同步码，打包回发数据，加密，执行发送。整个流程就出来了。
# 
# 4.最后补上了以时间为种子的离线同步算法，其实很简单，也有失误的机率，不完美。
#
# 5.new 文件夹是最新的srv 重写，做了少量优化和提高了代码的可阅读性，最后将关键拓展详情都在opendata.c 中详细描述了一翻...
#   new->srv 优化如下:
#                    1.优化队列工作效率，同时提高线程任务分配效率，减少线程切换，但是导致结果显示不理想，线程0 总是承包了全部的任务，几乎
#                      这就很需要各位添加业务压力，增加业务耗时来消耗资源，让任务产生挤压，其他线程就会被启动
#                    2.优化的网络传输IO 操作，界面回显，程序初始化等一些小修改，同时并没有更新业务模块->即数据报模块并没有被更新过
#                      客户端也没有被更新
#                    3.更新后实测在原来的redis 业务操作量下，再叠加负载sleep(100)=0.1ms 的情况下，仍能跑出1W/S 左右的成绩
#                      而且这时候pthread-1,2,3 负载比较均匀，但是pthread-4 就毕竟少用到，可能我这是双核机的缘故
#                      测试平台：G860+H61+8G-1333 and debian 8 64 bit + xfce4
#
#  作者：
#  短连接服务器，作者会尽量少更新这方面的代码，也就是这个版本未来预计不会被刷新了，作者考虑转向长连接服务器的实现。
#  这个版本不完整...但可作为学习socket短连接服务器的一个例子，代码稳定，作者同时表明：这些版本都是个人分享，并非生产环境下设计的代码。
#  使用方式不明白可以参考第一版的使用说明，主要就是make 编译出二进制程序后，按照程序提示进行操作。
