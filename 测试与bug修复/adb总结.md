## 1、基础命令

### 1.1 设备信息

####获取序列号：
​           adb **get**-serialno

####获取机器MAC地址：
​           adb shell cat /sys/**class**/net/wlan0/address

####获取CPU序列号：
​          adb shell cat /proc/cpuinfo

####获取设备名称：
​          adb shell cat /system/build.prop

#### 查看连接计算机的设备：
​           adb devices



###1.2 设备操作

####重启机器：
​           adb reboot

####重启到bootloader，即刷机模式：
​           adb reboot     bootloader

####重启到recovery，即恢复模式：
​           adb reboot     recovery

####将system分区重新挂载为可读写分区：
​          adb remount



1.2 设备状态

####查看设备cpu和内存占用情况：
​          adb shell top

####查看占用内存前6的app：
​          adb shell top -m 6

####刷新一次内存信息，然后返回：
​          adb shell top -n 1

####查询各进程内存使用情况：
​          adb shell procrank

####查看当前内存占用：
​          adb shell cat /proc/meminfo

####查看IO内存分区：
​          adb shell cat /proc/iomem



1. 杀死一个进程：
             adb shell kill [pid]
2. 查看进程列表：
             adb shell ps
3. 查看指定进程状态：
             adb shell ps -x [PID]
4. 查看后台services信息：
             adb shell service list
5. 



###文件操作

1. 从本地复制文件到设备：
             adb push <local>     <remote>
2. 从设备复制文件到本地：
             adb pull <remote> <local>
3. 列出目录下的文件和文件夹，等同于dos中的dir命令：
             adb shell ls
4. 进入文件夹，等同于dos中的cd 命令：
             adb shell cd <folder>
5. 重命名文件：
             adb shell rename path/oldfilename path/newfilename
6. 删除system/avi.apk：
             adb shell rm /system/avi.apk
7. 删除文件夹及其下面所有文件：
             adb shell rm -r <folder>
8. 移动文件：
             adb shell mv path/file newpath/file
9. 设置文件权限：
             adb shell chmod 777     /system/fonts/DroidSansFallback.ttf
10. 新建文件夹：
              adb shell mkdir path/foldelname
11. 查看文件内容：
              adb shell cat <file>
12. 查看wifi密码：
              adb shell cat /data/misc/wifi/*.conf



1. 跑monkey：
             adb shell monkey -v -p your.**package**.name 500