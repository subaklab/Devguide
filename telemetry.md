# 텔레메트리 (Telemetry)
텔레메트리는 QGroundControl과 통신하는데 사용할 수 있습니다. 특별히 파라미터를 튜닝하는데 유용한데 매번 케이블을 꽂지 않아도 파라미터를 변경할 수 있기 때문입니다.

## 3DR WIFI Telemetry
3DR WIFI 텔레메트리를 사용하면 트랜스미터 하나만 있으면 됩니다.(컴퓨터/타블릿에 이미 WIFI 카드/스틱이 있기 때문에) 이 모듈을 ```TELEM``` 포트에 꽂으면 WIFI 스테이션처럼 동작합니다.
```sh
essid: APM_PIX
password: 12345678
```
일단 WIFI에 연결되면 자동으로 QGroundControl에 연결됩니다.
![](images/hardware/3dr_wifi_1.JPG)
![](images/hardware/3dr_wifi_2.png)
