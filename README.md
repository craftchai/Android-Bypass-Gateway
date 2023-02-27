# Android手机制作旁路网关 旁路由

安卓shell
电脑使用adb
下载地址：
Windows版本：https://dl.google.com/android/repository/platform-tools-latest-windows.zip
Mac版本：https://dl.google.com/android/repository/platform-tools-latest-darwin.zip
Linux版本：https://dl.google.com/android/repository/platform-tools-latest-linux.zip

使用方式：
需开启“Android调试”，在「设置」-「开发者选项」-「Android 调试」，如果找不到“开发者选项”，需要在「设置」-「关于手机」连续点击「版本号」7 次

查看设备：adb devices

无线连接：adb connect 192.168.0.111

无线连接需要开启网络ADB调试

进入shell：adb shell

上传文件到手机：adb push 电脑路径 手机路径

下载文件到电脑：adb pull 手机路径 电脑路径

安装APK：adb install APK路径

手机使用Termux
下载地址：
https://github.com/termux/termux-app/releases

使用方式：(略)
配置旁路网关
建议先将手机设置为固定IP，方式很多请自行Google

一键脚本




#!/system/bin/sh

tun='tun0' #虚拟接口名称
dev='wlan0' #物理接口名称，eth0、wlan0
interval=3 #检测网络状态间隔(秒)
pref=18000 #路由策略优先级

# 开启IP转发功能
sysctl -w net.ipv4.ip_forward=1

# 清除filter表转发链规则
iptables -F FORWARD

# 添加NAT转换，部分第三方VPN需要此设置否则无法上网，若要关闭请注释掉
iptables -t nat -A POSTROUTING -o $tun -j MASQUERADE

# 添加路由策略
ip rule add from all table main pref $pref
ip rule add from all iif $dev table $tun pref $(expr $pref - 1)

contain="from all iif $dev lookup $tun"

while true ;do
    if [[ $(ip rule) != *$contain* ]]; then
            if [[ $(ip ad|grep 'state UP') != *$dev* ]]; then
                echo -e "[$(date "+%H:%M:%S")]dev has been lost."
            else
                ip rule add from all iif $dev table $tun pref $(expr $pref - 1)
                echo -e "[$(date "+%H:%M:%S")]network changed, reset the routing policy."
            fi
    fi
    sleep $interval
done




赋予可执行权限：chmod +x proxy.sh

执行：nohup ./proxy.sh &

更改网关
全局设备更改：修改主路由的DHCP设置

单一设备更改：更改设备的网关

排错
安卓系统每次切换网络设置都会将部分设置重置，一些“永久生效”的配置方式在手机重启后也会被重置

检查IP转发功能是否启用：cat /proc/sys/net/ipv4/ip_forward

检查iptables是否允许数据包通过：iptables -nvL -t (filter|nat|mangle)

检查路由策略：ip rule

检查网卡接口：ip a
