# 텔레메트리 (Telemetry)
텔레메트리는 QGroundControl과 통신하는데 사용할 수 있습니다. 그리고 매번 케이블을 연결하지 않고 파라미터를 변경해서 튜닝하는데 매우 유용합니다.

## 3DR WIFI 텔레메트리

3DR WIFI 텔레메트리를 이용하는 경우 트랜스미터 한쪽만 필요합니다.(일반적으로 컴퓨터/타블릿에 WIFI가 제공되기 때문) 모듈을  ```TELEM``` 포트에 꽂기만하면 WIFI 스테이션처럼 동작합니다.
```sh
essid: APM_PIX
password: 12345678
```
일단 WIFI에 연결되면, 자동으로 QGroundControl에 연결됩니다.
![](images/hardware/3dr_wifi_1.JPG)
![](images/hardware/3dr_wifi_2.png)
