如果iobroker还么有安装MQTT Broker/Client适配器，请先安装一个。
然后打开配置Adapter configuration: mqtt.0->CONNECTION->Connection settings->IP address: [IPv4] 192.168.2.11 - wlan0
首先安装一些依赖的软件：
```ruby
$ sudo apt-get install python3-pip libglib2.0-dev
$ sudo pip3 install bluepy paho-mqtt
```
接着，找到一个合适的地方，下载文件，比如
```
$ mkdir /home/pi/python3/
$ cd /home/pi/python3/
$ git clone https://github.com/da0mao/Xiaomi_LYWSD03MMC_iobroker-mqtt.git
```
然后让我们来扫描一下你的温度计`$ sudo hcitool lescan`，获取mac地址,按ctrl+c停止，假设扫描结果如下：
```
LE Scan ...
20:09:50:FB:E4:92 (unknown)
4A:49:97:79:FD:F5 (unknown)
...
E4:E1:E8:E9:AE:E7 LYWSD03MMC
53:93:02:17:E5:C5 (unknown)
53:93:02:17:E5:C5 (unknown)
...
```
获得设备mac：`E4:E1:E8:E9:AE:E7`，来试试看能否正常读取数据
```
$ python3 /home/pi/python3/Xiaomi_LYWSD03MMC_iobroker-mqtt/LYWSD03MMC.py -d E4:E1:E8:E9:AE:E7 -r -b 5 -c 1 -m 192.168.2.11 -del 10
```
其中，-d （device，设备）后面是mac地址， -r（round， 圆整），-b 5（battery，电量）-c（count，每次测量次数），-m （mqtt broker地址，这里即树莓派IP），-del（delay，检测间隔10秒），结果如下：
```
args.device E4:E1:E8:E9:AE:E7
['E4:E1:E8:E9:AE:E7'] 1
Delay set to 10 seconds
Trying to connect to E4:E1:E8:E9:AE:E7
Connection lost
Waiting...
Trying to connect to E4:E1:E8:E9:AE:E7
Temperature: 25.9
Humidity: 57
Battery voltage: 3.004
Battery level: 90
1 measurements collected. Exiting in a moment.
Topic blemqtt/E4E1E8E9AEE7
measurements deque([Measurement(temperature=25.9, humidity=57, voltage=3.004, calibratedHumidity=0, battery=90, timestamp=1599887990)])
25.9
Publishing {"temperature": "25.9", "humidity": "57", "batt": "90", "voltage": "3.004"}
Connection lost
Waiting...
Trying to connect to E4:E1:E8:E9:AE:E7
Temperature: 25.9
...
```
回到iobroker看看是否已经收到数据了，如果都没问题了，可以自动后台运行脚本即可，先打开脚本更新路径和mac地址
```
nano /home/pi/python3/Xiaomi_LYWSD03MMC_iobroker-mqtt/Xiaomi_LYWSD03MMC_iobroker-mqtt/blemqtt.service
```
然后拷贝到系统文件夹，启动自动运行
```
$ sudo cp /home/pi/python3/Xiaomi_LYWSD03MMC_iobroker-mqtt/Xiaomi_LYWSD03MMC_iobroker-mqtt/blemqtt.service /etc/systemd/system/
$ cd /etc/systemd/system/
$ sudo systemctl enable blemqtt
$ sudo systemctl start blemqtt
```