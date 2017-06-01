### 第一种方法:
通过配置 /etc/network/interfaces 文件实现
修改后文件内容如下：

```$xslt
auto lo
iface lo inet loopback
iface eth0 inet dhcp
auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-ssid “你的wifi名称”
wpa-psk “你的wifi密码”
```

具体各行配置的意思如下:
```$xslt
auto lo //表示使用localhost
iface eth0 inet dhcp //表示如果有网卡ech0, 则用dhcp获得IP地址 (这个网卡是本机的网卡，而不是WIFI网卡)
auto wlan0 //表示如果有wlan设备，使用wlan0设备名
allow-hotplug wlan0 //表示wlan设备可以热插拨
iface wlan0 inet dhcp //表示如果有WLAN网卡wlan0 (就是WIFI网卡), 则用dhcp获得IP地址
wpa-ssid “你的wifi名称”//表示连接SSID名
wpa-psk “你的wifi密码”//表示连接WIFI网络时，使用wpa-psk认证方式，认证密码
```
上述定义后，如果有网线连接，则采取DHCP自动连接获得地址，使用命令

```$xslt
udo /etc/init.d/networking restart
sudo service networking restart    #跟上面行的作用一样。
```
成功后，用 ifconfig 命令可以看到 wlan0 设备，且有了IP地址(已连接)。

有时需要执行sudo ifdown wlan0和sudo ifup wlan0才能发生作用，如果还不行，sudo reboot吧。

如果做服务器，建议设置静态IP

要做服务器的话，最好开机启动设置静态IP地址，在上面加上下面的部分：
```$xslt
iface default inet static    #将上面的iface wlan0 inet dhcp改为这一行。
address 192.168.1.2          #静态IP地址。
netmask 255.255.255.0        #IP掩码，为0的部分地址可通过。
gateway 192.168.1.1          #网关，一般就是路由器的主地址。
dns-nameservers x.x.x.x      #你的本地dns地址
```
### 第二种方法：
修改sudo nano /etc/wpa_supplicant/wpa_supplicant.conf实现 ps:与安卓手机wifi配置文件几乎相同
```$xslt
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=0
ap_scan=2
network={
    ssid=“WIFI名称“
    proto=WPA2
    key_mgmt=WPA-PSK
    pairwise=TKIP
    group=TKIP
    psk=”WIFI密码“
    priority=200
    }
```
然后修改文件sudo nano /etc/network/interfaces,修改后的文件内容如下：
```$xslt
auto lo
iface lo inet loopback
iface eth0 inet dhcp.auto wlan0
iface wlan0 inet dhcp
pre-up wpa_supplicant -B -Dwext -iwlan0 -c/etc/wpa_supplicant/wpa_supplicant.conf
post-down killall -q wpa_supplicant
```
修改完成后，使用以下命令重启网络
```$xslt
sudo /etc/init.d/networking restart
```
成功后，用 ifconfig 命令可以看到 wlan0 设备，且有了IP地址(已连接)。

附注：上述两种方法我们都是使用的DHCP动态IP，如果要设置静态ip方法和以及连接隐藏SSID AP的方法：

####（1）设置静态ip
修改文件sudo nano /etc/network/interfaces
```$xslt
auto lo
iface lo inet loopback
iface eth0 inet dhcp
allow-hotplug wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet static
address 192.168.1.2
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers x.x.x.x #你的本地dns地址
```

####（2）连接WIFI不广播隐藏SSID
在ssid=”XXXX”下面加一行scan_ssid=1后重启,具体如下：
```$xslt
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=0
ap_scan=2
network={
    ssid=“网络id“
    scan_ssid=1
    proto=WPA2
    key_mgmt=WPA-PSK
    pairwise=TKIP
    group=TKIP
    psk=”密码“}
```
重启后就可以连上这个不广播SSID的无线网络。


注：转载自https://my.oschina.net/u/2306127/blog/392442