---
title: 使用Java Socket发送魔术包：实现电脑远程开机(WOL)
excerpt: true
date: 2015-03-17 16:12:06
thumbnail: https://yongshengcnd.oss-cn-beijing.aliyuncs.com/paintings/%E8%85%93%E7%89%B9%E7%83%88%E5%90%AC%E6%A1%91%E7%B4%A2%E8%A5%BF%E9%95%BF%E7%AC%9B%E9%9F%B3%E4%B9%90%E4%BC%9A.jpeg
menu:
	Go Home: /index.html
tags: [Java]
---
<span style="color: #555;font-size: 1rem;display: inline-block;padding-bottom: 15px;text-indent: 2em;">
背景图《腓特烈听桑索西长笛音乐会》，由阿尔道夫·门采尔（德国）于1852年创作，该画的写实手法体现于画面每一处细节。幻妙的音乐，精致的灯盏，婉转的长笛，闪烁的烛光，每一种物象的和谐构置复原了一幅展现上层社会风貌的画面。
</span>

　　远程开机也被称为远程唤醒技术（Wake on Lan: WOL），是指可以通过局域网、互联网或者通讯网实现远程开机，无论目标主机离用户有多远、处于什么位置，只要其与发送命令主机可以通信，就能够被随时启动，该技术被现在的大多数主板与网卡所支持。
<!-- more -->

　　远程开机的实现主要依靠向目标主机发送特定格式的数据包，最初AMD公司推出的MagicPackage用于生成远程唤醒所需的特殊数据包，俗称魔术包（Magic Package）。MagicPackage技术只是AMD公司开发并推广的技术，尚未成为一项国际标准，但是该技术受到大多数网卡制造商的支持，因此具有远程唤醒功能的网卡都兼容这项技术。
# 1 远程唤醒的必备条件
　　远程唤醒只能依赖于主机硬件实现，任何用于远程控制的客户端软件都不能完成远程唤醒，因为这些软件在关机状态下是无法工作的。要实现远程唤醒功能需要满足以下几方面的条件：<br/>

　　1. 主板支持：要实现远程唤醒，目标主机的主板必须支持远程唤醒功能，能在电脑关机时为网卡供电。 目前（2002年以后）的大部分主板都支持这该功能；<br/>
　　2. 在CMOS中打开远程唤醒功能：开机时进入CMOS，并将“Pow Management Setup”的“Wake Up On Lan”或“Resume by Lan”项设置为“Enable”或“On”即可（作者在本文的实验机上并未找到该项设置，也未进行该项设置）；<br/>
　　3. 网卡支持与设置：要实现远程唤醒，主机网卡也必须支持远程唤醒功能，大多数现代网卡都已支持该功能。在硬件支持的前提下还要打开网卡的远程唤醒功能才能实现唤醒，打开网卡的远程唤醒功能有不止以下两种方法：①.右击“我的电脑”并选择“管理”选项，在随后出现的“计算机管理”窗口中找到“设备管理”，在设备列表中找到“网络适配器”下的本地网卡（注意是有线网卡），右击本地网卡并选择“属性”，在弹出的对话框中选择“高级”页签，选择“Wake on Magic Package”或“网络唤醒”选项并将其值设置为“开启”，在同一个窗口中选择“电源管理”页签，在“允许设备唤醒计算机”以及“只允许幻数据包唤醒计算机”选项前打钩，点击【确定】按钮；②.在win7系统中进入“控制面板”->“网络和Internet”->“网络连接”，找到本地连接，右击“本地连接”并选择“属性”，在随后出现的“本地连接 属性”窗口中点击“网络”页签下的【配置】按钮，在随后出现的窗口的“高级”和“电源管理”页签中进行与方法1同样的设置，点击【确定】按钮；<br/>
　　4. 电源支持：主机必须连接电源供电，笔记本电脑必须插继电器。必须使用ATX电源，而且其+5V Standby电流必须比较大，根据Intel的建议，它需要在600mA以上；<br/>
　　5. 目标主机上一次必须正常关机：如果计算机上次是非正常关机（突然断电、强制关机或者关机时发生错误）有可能导致远程唤醒失败，因为一些网卡需要在计算机关机的时候复位一个标记，而这个动作只有在正常关机的时候才会发生；<br/>
　　6. 发送开机命令的主机必须能够与目标主机建立通信：如果发送广播魔术包，那么只要保证广播包能到达目标主机即可，如果发送的是定向包则需要局域网路由器的支持，需要在路由器中配置一个到目标主机的路由信息。<br/>
　　具备以上6个条件之后就可以向目标主机发送魔术包使其自动开机，在展示代码以前先对魔术包以及数据格式的转换进行介绍。
# 2	魔术包与编码转换
## 2.1	魔术包的组成
　　魔术包是用16进制表示的数据包，它由固定的前缀数据以及固定重复次数的目标主机MAC地址所组成。所谓固定前缀数据即6对“FF”，所谓固定重复次数即16次，也就是说魔术包是由12个“F”加重复16次的主机MAC地址组成，例如本文所用试验机的MAC地址为“28-D2-44-35-68-A7”，所以使该机远程开机的魔术包为：
```
0xFFFFFFFFFFFF28D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A728D2443568A7
```
　　在Windows系统中，主机的MAC地址可以通过在命令窗口中输入“ipconfig -all”命令查看。
## 2.2	魔术包的编码转换
　　在发送魔术包之前需要将魔术包的内容进行编码转换，将其转换为二进制格式的数据进行发送，每一个16进制数占用4bit，所以上文的MAC地址（0x28D2443568A7）经过转码之后的二进制结果为：0010 1000 1101 0010 0100 0100 0011 0101 0110 1000 1010 0111。<br/>
　　使用Java发送UDP广播包时需要将发送的数据存储到byte数组缓存中进行发送，所以需要将魔术包的二进制数据转换为byte数组。Java中一个byte类型数据的长度为8bit，而在魔术包中每一个16进制数占用4bit，所以需要将两个16进制数组合表示为一个Java中的byte类型数据，下文将以试验机MAC地址的前两个数“28” 为例来说明编码转换过程：<br/>
　　1.将第1个数转换为byte类型数据：(byte)2 => 0000 0010；
　　2.将第1个数的转换结果右移4bit：(byte)2 << 4 => 0010 0000；
　　3.将第2个数 转换为byte类型数据：(byte)8 => 0000 1000
　　4.将2步和第3步中得到的两个数进行逻辑或运算得到转换结果：(byte)2 << 4) | ((byte)8) => 0010 1000。<br/>
　　参照上述步骤将“0x28D2443568A7”转换为byte数组的结果如下所示：<br/>
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTUwNDAxMTM0NTE2Mzc5)<br/>
　　在介绍完远程唤醒的概念、必备条件以及魔术包之后就可以开始用Java代码发送UDP魔术包，使得目标主机进行远程开机。
# 3	Java发送UDP广播包
　　使用Java代码进行数据编码转换以及发送UDP广播包实现电脑远程开机的的代码如下：
```
package com.wakeonlan;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.InetAddress;
import java.net.MulticastSocket;
import java.net.UnknownHostException;

public class WakeOnLan {
	/**
	 * main方法，发送UDP广播，实现远程开机，目标计算机的MAC地址为：28D2443568A7
	 */
	public static void main(String[] args) {
		String ip = "255.255.255.255";//广播IP地址
		int port = 9;//端口号
		//魔术包数据
		String magicPacage = "0xFFFFFFFFFFFF" +
		  "28D2443568A7" + "28D2443568A7" + "28D2443568A7" + "28D2443568A7" +
		  "28D2443568A7" + "28D2443568A7" + "28D2443568A7" + "28D2443568A7" +
		  "28D2443568A7" + "28D2443568A7" + "28D2443568A7" + "28D2443568A7" +
		  "28D2443568A7" + "28D2443568A7" + "28D2443568A7" + "28D2443568A7";
		//转换为2进制的魔术包数据
		byte[] command = hexToBinary(magicPacage);
		
		//广播魔术包
		try {
			//1.获取ip地址
			InetAddress address = InetAddress.getByName(ip);
			//2.获取广播socket
			MulticastSocket socket = new MulticastSocket(port);
			//3.封装数据包
			/*public DatagramPacket(byte[] buf,int length
			 * 		,InetAddress address
			 * 		,int port)
			 * buf：缓存的命令
			 * length：每次发送的数据字节数，该值必须小于等于buf的大小
			 * address：广播地址
			 * port：广播端口
            */
			DatagramPacket packet = new DatagramPacket(command, command.length, address, port);
			//4.发送数据
			socket.send(packet);
			//5.关闭socket
			socket.close();
		} catch (UnknownHostException e) {
			//Ip地址错误时候抛出的异常
			e.printStackTrace();
		} catch (IOException e) {
			//获取socket失败时候抛出的异常
			e.printStackTrace();
		}
	}
	
	/**
	 * 将16进制字符串转换为用byte数组表示的二进制形式
	 * @param hexString：16进制字符串
	 * @return：用byte数组表示的十六进制数
	 */
	private static byte[] hexToBinary(String hexString){
		//1.定义变量：用于存储转换结果的数组
		byte[] result = new byte[hexString.length()];
		
		//2.去除字符串中的16进制标识"0X"并将所有字母转换为大写
		hexString = hexString.toUpperCase().replace("0X", "");
		
		//3.开始转换
		//3.1.定义两个临时存储数据的变量
		char tmp1 = '0';
		char tmp2 = '0';
		//3.2.开始转换，将每两个十六进制数放进一个byte变量中
		for(int i = 0; i < hexString.length(); i += 2){
			result[i/2] = (byte)((hexToDec(tmp1)<<4)|(hexToDec(tmp2)));
		}
		return result;
	}
	
	/**
	 * 用于将16进制的单个字符映射到10进制的方法
	 * @param c：16进制数的一个字符
	 * @return：对应的十进制数
	 */
	private static byte hexToDec(char c){
		return (byte)"0123456789ABCDEF".indexOf(c);
	}
}
```
# 4	参考与鸣谢 
　　在本文写作的过程中参考了很多网络资源，其中参考了如下几篇文章，在此对所有作者的无私奉献表示衷心的感谢。
　　http://blog.csdn.net/a351945755/article/details/22578221
　　http://kuku789123.blog.163.com/blog/static/13616735120130894146519/