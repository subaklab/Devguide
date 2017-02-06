# QGroundControl에서 비디오 스트리밍
이 페이지에서는 카메라를 companion computer (Odroid C1)와 셋업하는 방법에 대해서 알아봅니다. 비디오 스트림은 Odroid C1을 통해 네트워크 컴퓨터로 전송되며 QGroundControl application에서 볼 수 있습니다.

전체 하드웨어 셋업은 아래 그림과 같습니다. 다음과 같은 부품이 필요합니다. :
* Odroid C1
* Logitech camera C920
* WiFi 모듈 TP-LINK TL-WN722N

![](images/videostreaming/setup-whole.png)

## Odroid C1에 Linux 환경 설치하기

Linux 환경 (Ubuntu 14.04)을 설치하려면, [Odroid C1 tutorial](https://pixhawk.org/peripherals/onboard_computers/odroid_c1)의 지시를 참고합니다. 이 튜터리얼에서는 UART 케이블을 가지고 Odroid C1에 접근하는 방법과 이더넷 연결하는 방법에 대해서 알아봅니다.

## 전원 연결 셋업 Set up alternative power connection

Odroid C1에 전원공급은 5V DC잭을 통해서 가능합니다. 만약 Odroid를 드론에 부착한 경우라면, 아래 이미지와 같이 through-hole 납땜 [방법](https://learn.sparkfun.com/tutorials/how-to-solder---through-hole-soldering)을 이용해서 5V DC잭에 2개 핀을 납땜하는 것을 추천합니다. Odroid C1의 점퍼 케이블(빨간색)을 통해 DC 전압 소스(5 V) 전원을 공급하며 Odroid C1의 ground 핀의 점퍼 케이블(검은색)을 회로 ground에 연결합니다.

![](images/videostreaming/power-pins.png)

## Odroid C1에 WiFi 연결 활성화
이 튜터리얼에서 WiFi 모듈 TP-LINK TL-WN722N을 사용합니다. Odroid C1에서 WiFi를 사용하기 위해서는 [Odroid C1 tutorial](https://pixhawk.org/peripherals/onboard_computers/odroid_c1)의 wifi 연결 설정하기 섹션을 참고합니다.


## WiFi 액세스 포인트 설정
이 섹션에서는 Odroid C1을 액세스 포인트로 셋업하는 방법을 알아봅니다. [tutorial](https://pixhawk.org/peripherals/onboard_computers/access_point)를 참고하였습니다. 카메라의 비디오 스트림을 Odroid C1을 통해 QGroundControl로 활성화시키는데 이 섹션의 내용을 따라서 할 필요는 없습니다. 하지만 Odroid C1을 액세스 포인트로 셋업하면 시스템을 독립적으로 사용하는 방법을 알 수 있습니다. TP-LINK TL-WN722N는 WiFi 모듈로 사용합니다. 명확히 하기 위해서 Odroid C1에서 WiFi 모듈 이름을 wlan0으로 정합니다. 다른 인터페이스(wlan1)을 사용한다면 wlan0의 수정 내용을 적용하면 됩니다.

### 온보드 컴퓨터를 액세스 포인트로
좀더 깊게 설명을 원한다면 [RPI-Wireless-Hotspot](http://elinux.org/RPI-Wireless-Hotspot)를 참고하세요.

필요한 소프트웨어를 설치

<div class="host-code"></div>

```bash
sudo apt-get install hostapd udhcpd
```

DHCP를 설정. `/etc/udhcpd.conf` 파일 수정

<div class="host-code"></div>

```bash
start 192.168.2.100 # This is the range of IPs that the hostspot will give to client devices.
end 192.168.2.200
interface wlan0 # The device uDHCP listens on.
remaining yes
opt dns 8.8.8.8 4.2.2.2 # The DNS servers client devices will use (if routing through the ethernet link).
opt subnet 255.255.255.0
opt router 192.168.2.1 # The Onboard Computer's IP address on wlan0 which we will set up shortly.
opt lease 864000 # 10 day DHCP lease time in seconds
```
모든 그외 'opt' 엔트리들은 비활성화시켜거나 여러분이 원하는대로 적절하게 설정하면 됩니다.

`/etc/default/udhcpd` 파일을 수정하고 해당 라인을 수정 :

<div class="host-code"></div>

```bash
DHCPD_ENABLED="no"
```

아래와 같이 수정

<div class="host-code"></div>

```bash
#DHCPD_ENABLED="no"
```

온보드 컴퓨터에 고정 IP 주소를 할당할 수 있습니다.  `/etc/network/interfaces` 파일을 수정하고 `iface wlan0 inet dhcp`(혹은 `iface wlan0 inet manual`) 라인을 다음과 같이 수정 :

```sh
auto wlan0
iface wlan0 inet static
address 192.168.2.1
netmask 255.255.255.0
network 192.168.2.0
broadcast 192.168.2.255
wireless-power off
```

원래(WiFi 클라이언트) 자동 설정을 비활성화시킵니다. 해당 라인을 수정(바로 붙어 있지 않을 수도 있고 없을 수도 있음) :

<div class="host-code"></div>

```sh
allow-hotplug wlan0
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```
to:

<div class="host-code"></div>

```sh
#allow-hotplug wlan0
#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
#iface default inet dhcp
```

WiFi 연결 셋업할 때 [Odroid C1 tutorial](https://pixhawk.org/peripherals/onboard_computers/odroid_c1)를 따라했다면, `/etc/network/intefaces.d/wlan0` 파일을 생성했을 것입니다. 이 설정을 비활성화시키기 위해서 이 파일 내에 모든 라인을 코멘트처리 합니다.

HostAPD 설정 : WPA-secured 네트워크를 만들려면, `/etc/hostapd/hostapd.conf` 파일을 수정(기존에 없는 경우 생성)하고 아래 라인을 추가합니다. :

```
auth_algs=1
channel=6            # Channel to use
hw_mode=g
ieee80211n=1          # 802.11n assuming your device supports it
ignore_broadcast_ssid=0
interface=wlan0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
# Change the to the proper driver
driver=nl80211
# Change these to something else if you want
ssid=OdroidC1
wpa_passphrase=QGroundControl

```

`ssid=`, `channel=`와 `wpa_passphrase=`을 적절하게 입력합니다. SSID는 핫스팟 이름으로 다른 장치에 브로드캐스트됩니다. 채널은 핫스팟이 실행되는 frequency를 말하며, wpa_passphrase는 무선 네트워크의 패스워드입니다. 더 다양한 옵션들은 `/usr/share/doc/hostapd/examples/hostapd.conf.gz` 파일을 참고하세요. 해당 공간에서 사용하고 있지 않은 채널을 찾습니다. wavemon 이라는 도구를 사용할 수 있습니다.

`/etc/default/hostapd` 파일을 수정하고 해당 라인을 수정 :

<div class="host-code"></div>

```
#DAEMON_CONF=""
```
아래와 같이:
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
여러분의 온보드 컴퓨터는 무선 핫스팟을 호스팅하게 됩니다. 부팅될때 핫스팟을 시작되도록 하려면 아래와 같은 추가 명령을 실행시킵니다. :

<div class="host-code"></div>

```
sudo update-rc.d hostapd enable
sudo update-rc.d udhcpd enable
```

온보드 컴퓨터가 액세스 포인트로 스스로 동작하게 되었습니다. 그라운드 스테이션도 연결이 가능합니다. 실제 액세스 포인트와 같게 동작하기를 원한다면, 라우팅을 설정하고 네트워크 주소 변환(NAT)을 설정해야 합니다. 커널에서 IP 포워딩을 활성화시킵니다. :

<div class="host-code"></div>

```sh
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
```
커널에서 NAT를 활성화 시키려면 다음 명령을 실행합니다. :

<div class="host-code"></div>

```sh
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

계속 설정을 유지하려면, 다음 명령을 실행합니다. :

<div class="host-code"></div>

```sh
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

이제 /etc/network/interfaces 파일을 수정하고 파일의 맨마지막에 다음과 같은 라인을 추가합니다. :

<div class="host-code"></div>

```sh
up iptables-restore < /etc/iptables.ipv4.nat
```

# gstreamer 설치하기

gstreamer 패키지를 컴퓨터와 Odroid C1에 설치하면 스트림이 시작됩니다. [QGroundControl README](https://github.com/mavlink/qgroundcontrol/blob/master/src/VideoStreaming/README.md) 지시를 참고합니다.

uvch264s 플러그인을 설치된 Odroid에서 스트림을 시작할 수 없는 경우, v4l2src 플러그인으로 실행시켜 보세요.:

<div class="host-code"></div>

```sh
 gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-h264,width=1920,height=1080,framerate=24/1 ! h264parse ! rtph264pay ! udpsink host=xxx.xxx.xxx.xxx port=5000
```
`xxx.xxx.xxx.xxx`는 QGC가 실행되는 IP 주소입니다. 만약 `Permission denied`와 같은 시스템 에러가 발생한다면, 위에 명령을 `sudo`로 실행시킵니다.

잘 동작하면, QGroundControl의 flight-mode의 왼쪽 아래 모서리에서 비디오 스트림을 볼 수 있습니다. 스크린샷은 아래와 같습니다.

![](images/videostreaming/qgc-screenshot.png)

비디오 스트림을 클릭하면, 위성지도가 왼쪽 아래 모서리에 보이며 비디오는 전체 배경으로 보이게 됩니다.
