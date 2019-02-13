---
layout: post
title: "Raspberry Pi"
subtitle: ""
date: 2019-01-23
author: Aether
category: coding
tags: RaspberryPi INSTALL
finished: true
---

## WIFI
在根目录下添加`wpa_supplicant.conf`, `ssh`两个文件，其中`ssh`为空
### /boot/wpa_supplicant.conf

    country=GB
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1
    
    network={
        ssid="WIFI名"
        psk="WIFI密码"
        key_mgmt=WPA-PSK
        priority=1
    }
    
    network={
        ssid="WIFI名"
        psk="WIFI密码"
        key_mgmt=WPA-PSK
        priority=2
    }
---
## Bluetooth

    安装支持包
    sudo apt-get install pi-bluetooth
    sudo apt-get install bluetooth bluez blueman
    
    添加pi用户到蓝牙组 
    sudo usermod -G bluetooth -a pi
    
    启动/增加SPP
    sudo nano /etc/systemd/system/dbus-org.bluez.service 
      
    修改内容如下:
    ExecStart=/usr/lib/bluetooth/bluetoothd -C
    ExecStartPost=/usr/bin/sdptool add SP
    
    连接蓝牙
    sudo rfcomm watch hci0

---
## GPS  L80-M39

### 1. 安装GPS模块需要的包
    
    sudo apt-get update && sudo apt-get -y install gpsd gpsd-clients python-gps

### 2. 重启树莓派，重新配置GPS服务

    sudo dpkg-reconfigure gpsd
        
### 3. 开启GPS模块服务:

    sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock

### 4. 停止GPS服务:

    sudo killall gpsd

### 5. 你也可以使用`systemctl`命令管理GPS服务，但在那之前需要修改文件`/etc/default/gpsd`

    START_DAEMON="true"
    USBAUTO="true"
    DEVICES="/dev/ttyUSB0"
    GPSD_OPTIONS="-F /var/run/gpsd.socket"
 
### 开始服务:
    
    sudo systemctl enable gpsd.sock sudo systemctl start gpsd.sock

### 停止服务:

    sudo systemctl stop gpsd.sock sudo systemctl disable gpsd.sock

### 获取模块信息

    sudo cgps -s

### 如果不能正常获得信息，使用以下命令检查串口工作是否正常

    cat/dev/ttyUSB0 

### 无定位情况下显示如下:

    pi@raspberrypi ~ $ sudo cat /dev/ttyUSB0
    $GPRMC,144034.00,V,,,,,,,090315,,,N*75
    $GPVTG,,,,,,,,,N*30
    $GPGGA,144034.00,,,,,0,00,99.99,,,,,,*60
    $GPGSA,A,1,,,,,,,,,,,,,99.99,99.99,99.99*30
    $GPGSV,1,1,01,15,,,25*7B
    
`NO FIX` 即未定位，将模块移动到室外空旷处将获得定位信息。

---
## SSH
    
    使用ssh登陆pi 默认用户名pi 密码raspberry
    ssh pi@192.168.2.104
    
    新建vnc服务 VNCViewer中ip:port登陆 默认port为0
    vncservice
    
    停止对应的vnc服务进程
    vncservice -kill :1
    
## 使用python解析串口数据并发送至蓝牙

### 准备
    安装python库
    sudo aptitude install python-dev
    sudo apt-get install python-serial
    python -m pip install pynmea2
    
    赋予串口读写权限
    sudo chmod 777 /dev/rfcomm0
    
    
### python代码：
    import serial
    import pynmea2
    import time
    ser1 = serial.Serial("/dev/ttyUSB0",9600)
    ser2 = serial.Serial('/dev/rfcomm0',9600,parity=serial.PARITY_NONE)
    if ser2.isOpen == False:
        ser2.open()
    ser2.write("serial turn on\n")
    
    try:
        while True:
            line = ser1.readline()
            if line.startswith('$GPRMC'):
                rmc = pynmea2.parse(line)
                ser2.write("Latitude: " +rmc.lat +"Longitude: "+rmc.lon+"\n")
            
    except KeyboardInterrupt:
            ser2.close()

    
    
    
