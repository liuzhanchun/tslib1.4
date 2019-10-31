# tslib1.4
修改过的源码


交叉编译环境：arm-linux-gnueabihf
目标：移植到开发板
安装 tslib1.4
在采用触摸屏的移动终端中，触摸屏性能的调试是一个重要问题之一，因为电磁噪声的缘故，触摸屏容易存在点击不准确，有抖动等问题。
tslib 是一个开源程序，能够为触摸屏驱动获得的采样提供诸如滤波、去抖动、校准等功能，通常作为触摸屏驱动的适配层，为上层的应用提供了一个统一的接口。
1.准备工作
确保已安装 autoconf、automake、autoreconf 和 libtool。如果没有安装或者不确定，可输入下列命令进行安装：
vmuser@Linux-host:~$ sudo apt-get install autoconf
vmuser@Linux-host:~$ sudo apt-get install automake
vmuser@Linux-host:~$ sudo apt-get install dh-autoreconf
vmuser@Linux-host:~$ sudo apt-get install libtool

2.准备源码
采用本压缩文件中的tslib-master
将tslib-master.zip拷贝到 ubuntu 主机上并解压，可参考如下命令：
vmuser@Linux-host:~$unzip tslib-master.zip

3.配置参数
进入解压后的目录~/tslib-master，执行如下命令：
vmuser@Linux-host:~$cd tslib-master
vmuser@Linux-host:~/tslib-master$ ./autogen.sh
vmuser@Linux-host:~/tslib-master$ ./configure --prefix=/opt/tslib --host=arm-linux-gnueabihf ac_cv_func_malloc_0_nonnull=yes
其中，--prefix 指定 tslib 的安装路径，用户也可以自行指定其它目录；而--host 指定交叉编译器，如果交叉编译器是 arm-linux-gnueabihf-gcc，则指定 arm-linux-gnueabihf。

4.编译
执行 make 指令：
vmuser@Linux-host:~/tslib-master$make

5.安装
因为在配置时指定了tslib将安装在/opt的子目录下，因此需要先获得/opt目录的写权限，
否则安装无法进行下去。当然，要在/opt 目录下创建新的文件夹也可以使用 sudo 身份，但是如果用户之前并未将交叉编译工具安在/usr/local/sbin、/usr/local/bin、/usr/sbin、/usr/bin、/sbin 和/bin 中的某个目录，那么 sudo 会因找不到编译工具而失败。因此一个比较好的建议是，在执行安装命令之前先修改/opt 目录的权限属性，然后使用普通用户权限执行安装。
vmuser@Linux-host:~/tslib-master$ sudo chmod 777 /opt
vmuser@Linux-host:~/tslib-master$ make install
此时，编译生成的库和头文件等都将会被拷贝到 prefix 指定的路径中（本文示例为/opt/tslib 目录），如果可以在该路径下看到这 4 个文件夹：bin、etc、lib、include，则表示安装完成。

6.修改 ts.conf  内容
为了让 tslib 软件在移植的时候可以定制输入模块，需要修改 ts.conf 文件的内容。
进入安装目录下的 etc 文件夹，修改 ts.conf 文件的内容。
vmuser@Linux-host:~/tslib-master$ vi /opt/tslib/etc/ts.conf
找到#module_raw input 那一行，去掉注释#
注意：行首不要留空格，要顶格。

7.移植到开发板
将安装好的整个 tslib 文件夹，拷贝到linux开发板的/usr/local 目录下

8.设置环境变量
编辑环境变量文件/etc/profile
#wr vi /etc/profile
在该文件的末尾添加如下内容：
export TSLIB_ROOT=/usr/local/tslib  /* 指定 tslib 目录路径 */
export TSLIB_TSDEVICE=/dev/input/event0 /* 指定触摸屏设备 */
export TSLIB_CALIBFILE=/etc/pointercal  /* 指定校准文件的存放位置*/
export TSLIB_CONFFILE=$TSLIB_ROOT/etc/ts.conf /* 指定 tslib 配置文件的路径*/
export TSLIB_PLUGINDIR=$TSLIB_ROOT/lib/ts /* 指定 tslib 插件文件的路径*/
export TSLIB_FBDEVICE=/dev/fb0  /* 指定帧缓冲设备 */
export QWS_MOUSE_PROTO=/dev/input/event0 /* 指定鼠标设备 */
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$TSLIB_ROOT/lib /* 添加 tslib 库 */

其中 TSLIB_ROOT 要更改为自己实际存放的 tslib 的绝对路径。
TSLIB_TSDEVICE 和QWS_MOUSE_PROTO 这两项需要查看EPC-6G2C-L触摸屏设备实际对应的设备文件（在/dev/input 目录下）。

9.执行测试命令
使用reboot命令重新启动开发板，使系统重新读取/etc/profile 的环境变量，然后执行如下命令：
 wr /usr/local/tslib/bin/ts_calibrate
此时应出现tslib校准界面，可以对触摸屏进行校准，校准后，还可以执行该目录下的其他程序，对触摸屏做进一步测试。

