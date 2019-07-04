---
title: Java串口编程：串口数据的发送与监听读取
excerpt: true
thumbnail: https://yongshengcnd.oss-cn-beijing.aliyuncs.com/paintings/%E9%98%BF%E5%B0%94%E5%8D%91%E6%96%AF%E5%B1%B1%E7%9A%84%E9%9B%AA%E5%B4%A9%20.jpeg
menu:
  Go Home: /index.html
---

　　本人在近期的开发工作中遇到向串口发送设备控制指令的需求，遂对串口编程进行了略微深入的钻研，在此对自己的一些心得和经验进行总结，以供大家参考与交流。
<!-- more -->
#串口介绍 #
　　串口全称为串行接口，一般指COM接口，是采用串行通信方式的扩展接口。其特点是数据位的传送按位顺序进行，最少只需一根传输线即可完成，成本低但传送速度慢。由于串口（COM）不支持热插拔及传输速率较低，目前部分新主板和大部分便携电脑已取消该接口。现在串口多用于工业控制和测量设备以及部分通信设备中。
　　根据美国电子工业协会(EIA: Electronic Industry Association）制定的标准，串口可以分为RS-232、RS-422以及RS-485等种类，其中以RS-232类型的接口最为典型和常见，本文所使用的是RS-232类型的9针串口（RS-232类型有25接口，但是现在几乎不再使用）。如图 1所示，是RS-232类型9针串口的实物示意图。RS-232类型9针串口每一个引脚的作用说明如图 2所示。
　　　　![图 1 RS232 9针串口实物示意图](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTUwMzE0MjExODU4Mjg2)
　　　　　　　　图 1 RS232 9针串口实物示意图

　　　　![图 2 RS232 9针串口的针脚示意图](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTUwMzE0MjEyMTQzMDc1)
　　　　　　　　图 2 RS232 9针串口的针脚示意图

　　想更加深入了解串口知识的读者请参阅以下内容：[串行接口](http://baike.baidu.com/link?url=JMmSltIsEH4Y5F36l6UPv5GbR_E_eOj2kFQUfxPDc__rSSXRMYrusbuBPvombbWHlxE2AbmRvRS8eFa1l2aQV8qdxyC9YCccnfSb7YbEntrGppG6cH4H3FMkQgGck9faFD1aRONpvmvb64SoYefeCK)、[RS-232](http://baike.baidu.com/link?url=YHh-sC5MTVLxPLxJfj4TPzxRAwlUUxTii7Uqkca5JIOWMaizIvwunGzv2ZCIUlneeByAYPuHlZ0TYHie9Q9Seq)、[RS-422](http://baike.baidu.com/link?url=of--Zy8XRLlnI6ixVlqtMy1AQge5iIJf09ekutAf4Daz6-JT8McRUAkx3snsdNkoPXpJEIAVsTHPpBTrpdk6la)、[rs485](http://baike.baidu.com/link?url=x6Lt3_NL3VdTGzS_9y-GpldPuIRY1O1MKjOdsKxQvq3rT2RyJ-Oht4qc5A9_584iNTOtvRqaap9cOOMVsyEU9K)

# Java对串口编程的API包#
　　目前比较常见的针对Java的串口包有3个来源：一是1998年SUN发布的串口通信API：comm2.0.jar（Windows环境下）和comm3.0.jar（Linux/Solaris环境下）；二是IBM的串口通信API；三是一些开源的API。本文介绍的是在Windows环境下使用java语言对串口进行编程，所以选取SUN的官方API（comm2.0.jar）。comm2.0.jar和comm3.0.jar的下载地址如下：
　　comm2.0.jar：http://pan.baidu.com/s/1pJKBOcN
　　comm3.0.jar：http://pan.baidu.com/s/1o6wrpwA

# 对串口编程的环境搭建 #
## 软件环境搭建##
　　在本文写作时，本人所使用的软件开发环境为：Windows7，Jdk1.6.0_10，Eclipse3.4.1。Java对串口编程的环境搭建分为以下步骤：
　　1.下载并安装jdk，本人jdk的根目录是“D:\ProgramFiles\Java\jdk1.6.0_10”，在接下来的文章中路径“D:\ProgramFiles\Java\jdk1.6.0_10”将使用“%JAVA_HOME%”来代替；
　　2.下载comm2.0.jar（下载链接见上文）并将串口编程必须的3个文件拷贝到jdk对应的文件夹中：
　　　　2.1.将win32com.dll文件拷贝到“%JAVA_HOME%\bin”以及“%JAVA_HOME%\jre\bin”目录下
　　　　2.2	将comm.jar文件拷贝到“%JAVA_HOME%\lib”以及“%JAVA_HOME%\jre\lib\ext”目录下
　　　　2.3	将javax.comm.properties文件拷贝到“%JAVA_HOME%\lib”以及“%JAVA_HOME%\jre\lib”下
　　3	新建工程并将comm.jar添加到工程文件中：
　　　　3.1	新建java工程serialPortProgramming，并将该工程的运行时环境（JRE）指定为步骤1中新安装的jdk。
　　　　3.2	在工程中新建“lib”文件夹，并将comm.jar文件拷贝到该文件夹下，右键点击该文件选择【Build Path】—【Add to Build Path】。
##“硬件” 环境准备 ##
　　Java对串口编程，首先设备上需要有串口（这不废话吗），但如今的大多数电脑主板上并不带串口，所以本人用Virtual Serial Port Driver软件虚拟出一对串口COM11和COM21，以方便文章的写作和实验的进行。
　　下载Virtual Serial Port Driver7.1（http://pan.baidu.com/s/1qW6wCsW）并安装，使用压缩包中的“vspdctl.dll”文件替换软件安装根目录中的“vspdctl.dll”文件即可完成破解。安装Virtual Serial Port Driver之后用该软件创建一对端口（COM11和COM21），在此创建的一对串口将在之后的实验中再次使用到。因为串口COM11和COM21是通过软件虚拟的、相互连接的一对串口，所以从COM11发送的数据COM21会接收到，反之亦然。
　　当然如果自己的设备上有串口的话也可以不用创建虚拟串口，只需要将一个串口的数据发送引脚（引脚3，如图 2所示）和另一个串口的数据接收引脚（引脚2）使用一根铜线链接即可实现数据的收发。如果设备上只有一个串口，要实现串口数据的收发，可以将串口的引脚2和引脚3使用铜线相连接，这样从本串口发送的数据就会通过本串口接收到。

# 实例一：获取本地串口并实现打开与关闭 #
　　在上文创建好的工程中新建包“com.serialPort.writer”并新建类OpenerAndCloser，该类实现串口的获取、打开与关闭。

OpenerAndCloser.java
```
package com.serialPort.writer;

import java.util.Enumeration;
import javax.comm.CommPortIdentifier;
import javax.comm.NoSuchPortException;
import javax.comm.PortInUseException;
import javax.comm.SerialPort;
import javax.comm.UnsupportedCommOperationException;

/**
 * 该类实现3个功能
 * 1.列举出本地所有的串口;
 * 2.打开所有串口(但是未向串口中写数据);
 * 3.关闭打开的串口。
 */
public class OpenerAndCloser {
	public static void main(String[] args){
		//1.获取本地所有的端口并输出其名称：
		//1.1.用于标识端口的变量
		CommPortIdentifier portIdentifier = null;
		
		//1.2.记录所有端口的变量
		Enumeration<?> allPorts 
			= CommPortIdentifier.getPortIdentifiers();
		
		//1.3.输出每一个端口
		while(allPorts.hasMoreElements()){
			portIdentifier 
				= (CommPortIdentifier) allPorts.nextElement();
			System.out.println("串口：" + portIdentifier.getName());
		}
		
		//2.打开COM11和COM21端口
		//2.1.获取两个端口
		CommPortIdentifier com11 = null;
		CommPortIdentifier com21 = null;
		try {
			com11 = CommPortIdentifier.getPortIdentifier("COM11");
			com21 = CommPortIdentifier.getPortIdentifier("COM21");
		} catch (NoSuchPortException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		//2.2.打开两个端口，但是什么都没干
		SerialPort serialCom11 = null;
		SerialPort serialCom21 = null;
		try {
			//open方法的第1个参数表示串口被打开后的所有者名称，
			//第2个参数表示如果串口被占用的时候本程序的最长等待时间，以毫秒为单位。
			serialCom11 
				= (SerialPort)com11.open("OpenerAndCloser", 1000);
			serialCom21 
				= (SerialPort)com21.open("OpenerAndCloser", 1000);
		} catch (PortInUseException e) {
			//要打开的端口被占用时抛出该异常
			e.printStackTrace();
		}
		
		//2.3.设置两个端口的参数
		try {
			serialCom11.setSerialPortParams(
					9600, //波特率
					SerialPort.DATABITS_8,//数据位数
					SerialPort.STOPBITS_1,//停止位
					SerialPort.PARITY_NONE//奇偶位
			);
			
			serialCom21.setSerialPortParams(
					9600, //波特率
					SerialPort.DATABITS_8,//数据位数
					SerialPort.STOPBITS_1,//停止位
					SerialPort.PARITY_NONE//奇偶位
			);
		} catch (UnsupportedCommOperationException e) {
			e.printStackTrace();
		}
		
		//3.关闭COM11和COM21端口
		//关闭端口的方法在SerialPort类中
		serialCom11.close();
		serialCom21.close();
	}
}

```
　　在以上的代码中，有两个较为重要的类，在此做以说明，它们是类CommPortIdentifier和类SerialPort。这两个类都来自于comm.jar，CommPortIdentifier类代表本地串口，可以通过该类的静态方法getPortIdentifier或getPortIdentifiers获取本地的串口，该类的实例方法open用于打开串口。SerialPort类同样代表本地串口，不过其代表的是打开的串口，可以通过该类的实例方法close关闭已经打开的串口，也可以通过该类的实例方法获取串口的输入输出流，实现往串口数据的读写操作。
　　执行Com11Writer类的main方法，就会发现控制台输出了本地机器的所有串口（包括虚拟串口和物理串口）。

# 实例二：串口数据的读写 #
## 向串口写数据##
　　在包“com.serialPort.writer”下新建Com11Writer类，该类实现往COM11写入数据“Hello World!”的功能，向串口COM11写入的数据会发送到与其相连的另一个串口COM21，并被COM21所接收，从串口接收数据的方式将在下文讲到，以下是Com11Writer的源代码：

Com11Writer.java
```
package com.serialPort.writer;

import java.io.IOException;
import java.io.OutputStream;
import javax.comm.CommPortIdentifier;
import javax.comm.NoSuchPortException;
import javax.comm.PortInUseException;
import javax.comm.SerialPort;

/**
 * Com11Writer类的功能是向COM11串口发送字符串“Hello World!”
 */
public class Com11Writer {
	
	public static void main(String[] args) {
		//1.定义变量
		CommPortIdentifier com11 = null;//用于记录本地串口
		SerialPort serialCom11 = null;//用于标识打开的串口
		
		try {
			//2.获取COM11口
			com11 = CommPortIdentifier.getPortIdentifier("COM11");
			
			//3.打开COM11
			serialCom11 = (SerialPort) com11.open("Com11Writer", 1000);
			
			//4.往串口写数据（使用串口对应的输出流对象）
			//4.1.获取串口的输出流对象
			OutputStream outputStream = serialCom11.getOutputStream();
			
			//4.2.通过串口的输出流向串口写数据“Hello World!”：
			//使用输出流往串口写数据的时候必须将数据转换为byte数组格式或int格式，
			//当另一个串口接收到数据之后再根据双方约定的规则，对数据进行解码。
			outputStream.write(new byte[]{'H','e','l','l','o',
					' ','W','o','r','l','d','!'});
			outputStream.flush();
			//4.3.关闭输出流
			outputStream.close();
			
			//5.关闭串口
			serialCom11.close();
		} catch (NoSuchPortException e) {
			//找不到串口的情况下抛出该异常
			e.printStackTrace();
		} catch (PortInUseException e) {
			//如果因为端口被占用而导致打开失败，则抛出该异常
			e.printStackTrace();
		} catch (IOException e) {
			//如果获取输出流失败，则抛出该异常
			e.printStackTrace();
		}
	}
}

```

## 从串口读数据##
　　从串口COM11发送的数据最终将到达与其连通的串口COM21，如果COM21处于可用状态，则到达的数据将被缓存，等待程序的读取。从串口读入数据有多种模式，本文将介绍“轮询模式”和事件监听模式。
　　“轮询模式”是指程序（线程）每隔固定的时间就对串口进行一次扫描，如果扫描发现串口中有可用数据，则进行读取。Com21PollingListener类使用“事件监听模式”读取串口COM21接收到的数据：

Com21PollingListener.java
```
package com.serialPort.listener;

import java.io.IOException;
import java.io.InputStream;
import javax.comm.CommPortIdentifier;
import javax.comm.NoSuchPortException;
import javax.comm.PortInUseException;
import javax.comm.SerialPort;

/**
 * Com21PollingListener类使用“轮训”的方法监听串口COM21，
 * 并通过COM21的输入流对象来获取该端口接收到的数据（在本文中数据来自串口COM11）。
 */
public class Com21PollingListener {
	public static void main(String[] args){
		//1.定义变量
		CommPortIdentifier com21 = null;//未打卡的端口
		SerialPort serialCom21 = null;//打开的端口
		InputStream inputStream = null;//端口输入流
		
		try{
			//2.获取并打开串口COM21
			com21 = CommPortIdentifier.getPortIdentifier("COM21");
			serialCom21 = (SerialPort) com21.open("Com21Listener", 1000);
			
			//3.获取串口的输入流对象
			inputStream = serialCom21.getInputStream();
			
			//4.从串口读入数据
			//定义用于缓存读入数据的数组
			byte[] cache = new byte[1024];
			//记录已经到达串口COM21且未被读取的数据的字节（Byte）数。
			int availableBytes = 0;
			
			//无限循环，每隔20毫秒对串口COM21进行一次扫描，检查是否有数据到达
			while(true){
				//获取串口COM21收到的可用字节数
				availableBytes = inputStream.available();
				//如果可用字节数大于零则开始循环并获取数据
				while(availableBytes > 0){
					//从串口的输入流对象中读入数据并将数据存放到缓存数组中
					inputStream.read(cache);
					//将获取到的数据进行转码并输出
					for(int j = 0;j < cache.length && j < availableBytes; j++){
						//因为COM11口发送的是使用byte数组表示的字符串，
						//所以在此将接收到的每个字节的数据都强制装换为char对象即可，
						//这是一个简单的编码转换，读者可以根据需要进行更加复杂的编码转换。
						System.out.print((char)cache[j]);
					}
					System.out.println();
					//更新循环条件
					availableBytes = inputStream.available();
				}
				//让线程睡眠20毫秒
				Thread.sleep(20);
			}
		}catch(InterruptedException e){
			e.printStackTrace();
		}catch (NoSuchPortException e) {
			//找不到串口的情况下抛出该异常
			e.printStackTrace();
		} catch (PortInUseException e) {
			//如果因为端口被占用而导致打开失败，则抛出该异常
			e.printStackTrace();
		} catch (IOException e) {
			//如果获取输出流失败，则抛出该异常
			e.printStackTrace();
		}
	}
}
```

　　“事件监听模式”是为串口注册一个事件监听类，当有数据到达串口的时候就会触发事件，在事件的响应方法中读取串口接收到的数据。Com21EventListener类使用“事件监听模式”读取串口COM21接收到的数据：
Com21EventListener.java
```
package com.serialPort.listener;

import java.io.IOException;
import java.io.InputStream;
import java.util.TooManyListenersException;
import javax.comm.CommPortIdentifier;
import javax.comm.NoSuchPortException;
import javax.comm.PortInUseException;
import javax.comm.SerialPort;
import javax.comm.SerialPortEvent;
import javax.comm.SerialPortEventListener;

/**
 * Com21EventListener类使用“事件监听模式”监听串口COM21，
 * 并通过COM21的输入流对象来获取该端口接收到的数据（在本文中数据来自串口COM11）。
 * 使用“事件监听模式”监听串口，必须字定义一个事件监听类，该类实现SerialPortEventListener
 * 接口并重写serialEvent方法，在serialEvent方法中编写监听逻辑。
 */
public class Com21EventListener implements SerialPortEventListener {
	
	//1.定义变量
	CommPortIdentifier com21 = null;//未打卡的端口
	SerialPort serialCom21 = null;//打开的端口
	InputStream inputStream = null;//输入流
	
	//2.构造函数：
	//实现初始化动作：获取串口COM21、打开串口、获取串口输入流对象、为串口添加事件监听对象
	public Com21EventListener(){
		try {
			//获取串口、打开窗串口、获取串口的输入流。
			com21 = CommPortIdentifier.getPortIdentifier("COM21");
			serialCom21 = (SerialPort) com21.open("Com21EventListener", 1000);
			inputStream = serialCom21.getInputStream();
			
			//向串口添加事件监听对象。
			serialCom21.addEventListener(this);
			//设置当端口有可用数据时触发事件，此设置必不可少。
			serialCom21.notifyOnDataAvailable(true);
		} catch (NoSuchPortException e) {
			e.printStackTrace();
		} catch (PortInUseException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (TooManyListenersException e) {
			e.printStackTrace();
		}
	}

	//重写继承的监听器方法
	@Override
	public void serialEvent(SerialPortEvent event) {
		//定义用于缓存读入数据的数组
		byte[] cache = new byte[1024];
		//记录已经到达串口COM21且未被读取的数据的字节（Byte）数。
		int availableBytes = 0;
		
		//如果是数据可用的时间发送，则进行数据的读写
		if(event.getEventType() == SerialPortEvent.DATA_AVAILABLE){
			try {
				availableBytes = inputStream.available();
				while(availableBytes > 0){
					inputStream.read(cache);
					for(int i = 0; i < cache.length && i < availableBytes; i++){
						//解码并输出数据
						System.out.print((char)cache[i]);
					}
					availableBytes = inputStream.available();
				}
				System.out.println();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
	
	//在main方法中创建类的实例
	public static void main(String[] args) {
		new Com21EventListener();
	}
}
```

## 读写程序的联合运行 ##
　　串口能接收到数据的前提是该串口处于打开（可用）状态，如果串口处于关闭状态，那么发送到该串口的数据就会丢失。所以在实验的过程中，如果使用铜线连接同一个串口的引脚2和引脚3，一定要注意的是千万不能在向串口发送完数据之后关闭该串口，然后再次打开串口去读取数据，一定要让串口始终处于打开状态直到程序运行结束。
　　基于以上的说明，在本文所涉及到的实例中，首先运行Com21PollingListener类（或Com21EventListener类）中的main方法打开端口监听程序，然后再运行Com11Writer类的main方法通过COM11向COM21发送数据，这样程序就能从COM21读取数据。

# 参考与鸣谢 #
　　在本文写作的过程中参考了很多网络资源，其中参考了如下几篇文章，在此对所有作者的无私奉献表示衷心的感谢。
　　http://blog.csdn.net/luoduyu/article/details/2182321
　　http://www.cnblogs.com/dyufei/archive/2010/09/19/2573913.html
　　http://www.cnblogs.com/dyufei/archive/2010/09/19/2573912.html
　　http://www.cnblogs.com/dyufei/archive/2010/09/19/2573911.html

　　本文实例的源工程文件可以从以下链接获取
　
- 链接: https://pan.baidu.com/s/1m9qXcYQMcppkEcD8YVxEbQ 
- 密码: `x5xw`