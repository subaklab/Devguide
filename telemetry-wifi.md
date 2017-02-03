# Wifi 텔레메트리

PX4는 UDP와 Wifi를 통해서 텔레메트리를 지원합니다. ground control station으로부터 heartbeat를 수신할때까지 255.255.255.255 주소의 14550 포트로 heartbeat 를 브로드캐스팅 합니다. 수신하는 시점에는 데이터를 ground control station에게 보내게 됩니다.
