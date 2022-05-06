# 2020-6-30

###### 为c盘扩容：（编译libzmq和cppzmq使用vs的IDE，但是c盘空间不够安装）

​	首先要求能够为c盘扩容的盘块（d盘）和c盘处于同一磁盘。在磁盘管理中可以查看，如和d盘在一个磁盘内，则可以为c盘从d盘中扩容空间，如果d盘已经被存放了文档，程序已经应用程序等，需要先备份d盘数据，再删除d盘，最后选择c盘，扩展即可。

​	对于安装在d盘的应用程序，应该先卸载，再删除文档，否则由于之前应用程序的注册表内容没有被改变，在重新下载的时候可能会报错。

```c++
The drivers or UNC share you selected does not exist or is not  accessible. Please select another .
```

解决方法：用电脑管家之类的程序扫描电脑，选择删除注册表里面的内容即可。



###### CodeBlocks安装

​	自定义安装目录（非默认目录下安装）会导致IDE找不到编译器，无法编译代码。

解决方法：codeblocks中setting->compiler->Toolchain excutables下修改编译器的安装目录为自定义的目录。



###### ZeroMQ的搭建

​	首先下载适合C++的libzmq和cppzmq。这是两个库文件，下载地址是基于github分享的。

------

# 2020-7-1

环境vs2019

zmq4.3.2.zip

E:\zeromq-4.3.2\builds\deprecated-msvc\vs2015

从github  https://github.com/zeromq/libzmq  上面下载到本地的libzmq文件夹安装E盘 E:\libzmq

源码需要编译：cmake gui（网页下载过于缓慢）

思路：基于zeromq库，进行REP-REQ模式的客户端和服务端程序，可正常收发数据，然后将数据报文采用protobuf/JSON进行封装，简单设计自己的应用层交互协议，比如你服务器有XXX函数，需要传递XXX参数，返回结果中需要返回XXX格式的数据，如何在协议中进行体现和设计以实现服务远程函数调用。

------

# 2020-7-25

```c++
//getline函数的重载及各种使用
getline()

//vector<>向量的用法

//map<element_type, element_type>

//SplitString的用法及重载

//stoi，stod，atof
```























