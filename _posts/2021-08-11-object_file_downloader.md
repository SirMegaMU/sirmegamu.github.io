---
layout: post
title: 项目：文件下载器
date: 2021-08-10
tags: Python
---


# 目标
服务器根据客户端发来的信息向客户端发送指定文件
# 思路
客户端向服务器发送要下载的文件名
服务器在当前文件夹寻找所需文件
分次发送文件的每一个部分
客户端接受文件的每一个部分
组合成完整的文件，解码
将文件储存到指定目录

# 实现

使用TCP协议，服务器与客户端分别构造。
信息发送时都是以二进制格式发送，可以在文件打开时就选择以二进制读取，省去解码的麻烦。
文件分批发送。服务器和客户端都要判断文件是否接受/读取完。

## 服务器

~~~python
# 1、导入模块
import socket
# 2、创建套接字
tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 设置套接字地址可以重用
# tcp_server_socket.setsockopt(当前套接字, 属性名, 属性值)
# socket.SO_REUSEADDR  地址是否可以重用   True可以重用  False 不可以重用
tcp_server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
# 3、绑定端口
tcp_server_socket.bind(("", 8081))
# 4、设置监听
tcp_server_socket.listen(128)
# 5、接受客户端连接
while True:
    new_client_socket, ip_port = tcp_server_socket.accept()
    print("客户端", ip_port[0],"端口",ip_port[1],"已连接")
    # 6、接收客户端发送的文件名
    recv_data = new_client_socket.recv(1024)
    file_name = recv_data.decode()
    print(file_name)
	# 文件不存在时提示文件无法下载
    try:
        # 7、读取文件内容
        with open(file_name, "rb") as file:
            # 8、发送文件内容（循环）
            while True:
            	# 文件分批发送
                file_data = file.read(1024)
                # 判断是否读取到了文件的末尾
                if file_data:
                    # 发送文件
                    new_client_socket.send(file_data)
                else:
                    break
    # 捕获错误
    except Exception as e:
        print("文件%s下载失败!" % file_name)
    else:
        print("文件%s下载成功" % file_name)
    # 9、关闭和当前客户端的连接
    new_client_socket.close()
# 10、关闭服务器
tcp_server_socket.close()
~~~



## 客户端

~~~python
# 1、导入模块
import socket
# 2、创建套接字
tcp_client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 3、连接服务器
tcp_client_socket.connect(("000.000.000.000", 8081))
# 4、接收用户输入的文件名
file_name = input("请输入要下载的文件名:")
# 5、发送文件名到服务端
tcp_client_socket.send(file_name.encode())
# 6、创建文件，保存服务器传来的信息
try:
    # 创建文件并用二进制写入
	with open("/home/sirmegamu/Desktop/"+file_name, "wb") as file:
 	   # 7、接收并保存服务端发送的数据（循环）
 	   while True:
 	       file_data = tcp_client_socket.recv(1024)
 	       # 判断数据是否传送完毕
  	      if file_data:
  	          file.write(file_data)
  	      else:
   	         break
except:
    print("保存错误！")
# 8、关闭套接字
tcp_client_socket.close()
~~~

<script data-ad-client="ca-pub-3585063968143785" async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
