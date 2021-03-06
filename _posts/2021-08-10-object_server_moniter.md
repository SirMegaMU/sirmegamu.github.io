---
layout: post
title: 项目：实时监视服务器的运行信息
date: 2021-08-10
tags: Python
---

# 项目：实时监视服务器的运行信息

**服务器**：Ubuntu虚拟机
**访问者**：PC主机

## _目标_：可直接在终端运行，可以自由设置运行信息刷新频率，可以同时显示多种不同的信息
### 终端运行解决：
将文件设置成_可执行_文件

~~~Ubuntu
$ chmod u+x main.py
~~~


### 刷新频率解决：
利用**time**模块使程序循环每执行一次就**休眠**指定秒数

~~~python
# 导入模块
import time
# 定义主程序函数,进入程序循环
def main():
	time_set_sleep = input("请输入刷新频率（秒/次）：")
	while True:
		time.sleep(time_set_sleep)
		pass
# 进入程序
if __name__ == "__main__":
	main()
	
~~~

### 显示信息解决：

使用**psutil**模块获取关于**CPU，硬件，内存，网络**的信息

~~~python
# 导入模块
import psutil
# 获取关于CPU的信息
cpu_core = psutil.cpu_count(logical=False) # 获取CPU物理核数
cpu_percent = psutil.cpu_percent(interval = time) # 获取CPU占用半分比(指定秒数刷新)
# 获取内存相关信息
memory_info = psutil.virtual_memory() # 得到一个字典
# 获取硬盘相关信息
disk_info = psutil.disk_partitions() # 得到一个字典
# 获取网络相关信息
net_info = psutil.net_io_counters() # 得到一个含有上传/下载信息的字典

~~~
### 进一步获取具体信息

~~~python
# 获取根目录的使用量和百分比
disk_info = psutil.disk_usage("/")
disk_percent = psutil.virtual_memory().percent
# 
# 获取网络上行/下载量
up_load = net_info.bytes_sent
down_load = net_info.bytes_recv

~~~
### 添加邮件发送模块

利用**yagmail**模块进行发邮件

_示例_

~~~python
# yagmail 发送邮件

# 导入模块
import yagmail
# 使用yagmail类创建对象（发件人，授权码，服务器）
# 使用megawang@foxmail.com发件
# 授权码：wqmizdtzrihldfgd
ya_dbj = yagmail.SMTP(user="megawang@foxmail.com",password="wqmizdtzrihldfgd",host="smtp.qq.com")
# 使用yagmail对象发送邮件（指定收件人，主题，内容）
content = "Hello World!"
ya_dbj.send("69071178@qq.com","测试",content)

~~~





## 最终效果
~~~python
#!/usr/bin/python3
# 导入模块
import psutil
import datetime
import yagmail
# 1，定义函数实现信息显示和日志保存
def linux_moniter(time):

    # 保存CPU使用率
    cpu_per = psutil.cpu_percent(interval=time)

    # 保存内存信息
    memory_info = psutil.virtual_memory()

    # 保存硬盘信息
    disk_info = psutil.disk_usage("/")

    # 获取系统时间
    custom_time = datetime.datetime.now().strftime("%F %T")
    # 保存网络信息
    net_info = psutil.net_io_counters()

    # 拼接字符串显示
    log_str ="▛▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▝▜\n"
    log_str+="▍      监控时间      ▍ CPU使用率  ▍ 内存使用率  ▍  硬盘使用率   ▍              网络收发               ▍\n"
    log_str+="▍        时间       ▍   共%d核    ▍ 共%.2fG内存 ▍共计%.2fG硬盘 ▍               比特               ▍\n"%(psutil.cpu_count(logical=False),memory_info.total/1024/1024/1024,disk_info.total/1024/1024/1024)
    log_str+="▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃\n"
    log_str+="▍%s ▍   %.2f%%   ▍    %s%%     ▍     %s%%    ▍  收：%s / 发：%s  ▍\n"%(custom_time,cpu_per,memory_info.percent,disk_info.percent,net_info.bytes_recv,net_info.bytes_sent)
    log_str+="▙▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▃▟\n"
    print(log_str)
    # 保存到日志文件
    f =open("log.txt","a")
    f.seek(0)
    f.write(log_str + "\n\n")
    f.close

    # 进行邮件示警
    # 使用yagmail类创建对象（发件人，授权码，服务器）
    ya_dbj = yagmail.SMTP(user="megawang@foxmail.com",password="wqmizdtzrihldfgd",host="smtp.qq.com")
    # 使用yagmail对象发送邮件（指定收件人，主题，内容）
    if cpu_per >= 90:
        content ="CPU占有率过高！！"
        ya_dbj.send("wtr666@qq.com","CPU占有率过高！",content)
    if memory_info.percent >= 90 :
        content ="内存占有率过高！！"
        ya_dbj.send("wtr666@qq.com","内存占有率过高！",content)
    if disk_info.percent >= 90:
        content ="硬盘占有率过高！！"
        ya_dbj.send("wtr666@qq.com","硬盘占有率过高！",content)

    
def main():
    time = input("请输入刷新频率（秒/次）：")
    try :
        while True:
            linux_moniter(int(time))
    except:
        print("输入错误")
    
# 2，进入死循环
if __name__ == "__main__":
    main()
    
~~~