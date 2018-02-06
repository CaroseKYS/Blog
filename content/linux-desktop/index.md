# 阿里云ECS CentOS 7 图形桌面安装、远程连接及chrome浏览器的安装与使用

由于题主的特殊应用场景，需要使用linux的图形界面进行远程链接，并在linux上运行chrome浏览器，现将操作过程记录如下，以备后查，但愿能够帮助到路过的吃瓜群众。

## 1. 购买ECS

+ 实例规格：ecs.c5.large（2核 4GB，计算型 c5）
+ 操作系统：CentOS 7.4 64位
+ 操作用户：root

## 2. 安装图形化界面

+ 安装 MATE Desktop： `yum groups install "MATE Desktop"`
+ 安装 GNOME Desktop：`yum groupinstall "GNOME Desktop"`


