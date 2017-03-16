# Snapdragon Flight Autopilot

Snapdragon Flight 플랫폼은 하이엔드 autopilot / onboard 컴퓨터입니다. PX4 Flight Stack이 DSP에서 실행되며 POSIX 인터페이스를 위해 [DSPAL API](https://github.com/ATLFlight/dspal)를 사용하는 QuRT 실시간 운영체제를 사용합니다. [Pixhawk](hardware-pixhawk.md)와 비교하면 카메라와 WiFi, 고성능 프로세싱 파워 그리고 다른 IO를 지원합니다.

Snapdragon Flight 플랫폼에 대해서 상세한 정보는 링크를 참조하세요. [Snapdragon-Flight-Details](https://www.intrinsyc.com/qualcomm-snapdragon-flight-details/)

![](images/hardware/hardware-snapdragon.jpg)

## 간단 요약

  * System-on-Chip: [Snapdragon 801](https://www.qualcomm.com/products/snapdragon/processors/801)
    * CPU: Quad-core 2.26 GHz Krait
    * DSP: Hexagon DSP (QDSP6 V5A) – 801 MHz+256KL2 (running the flight code)
    * GPU: Qualcomm® Adreno™ 330 GPU
    * RAM: 2GB LPDDR3 PoP @931 MHz
  * Storage: 32GB eMMC Flash
  * Video: Sony IMX135 on Liteon Module 12P1BAD11
    * 4k@30fps 3840×2160 video capture to SD card with H.264 @ 100Mbits (1080p/60 with parallel FPV), 720p FPV
  * Optic Flow: Omnivision OV7251 on Sunny Module MD102A-200
    * 640x480 @ 30/60/90 fps
  * Wifi: Qualcomm® VIVE™ 1-stream 802.11n/ac with MU-MIMO † Integrated digital core
  * BT/WiFi: BT 4.0 and 2G/5G WiFi via QCA6234
    * 802.11n, 2×2 MIMO with 2 uCOAX connectors on-board for connection to external antenna
  * GPS: Telit Jupiter SE868 V2 module (use of an external u-Blox module is recommended by PX4 instead)
    * uCOAX connector on-board for connection to external GPS patch antenna
    * CSR SiRFstarV @ 5Hz via UART
  * Accelerometer / Gyro / Mag: Invensense MPU-9250 9-Axis Sensor, 3x3mm QFN, on bus SPI1
  * Baro: Bosch BMP280 barometric pressure sensor, on bus I2C3
  * Power: 5VDC via external 2S-6S battery regulated down to 5V via APM adapter
  * Availability: [Intrinsyc Store](http://shop.intrinsyc.com/products/snapdragon-flight-dev-kit)

## Connectivity

  * One USB 3.0 OTG port (micro-A/B)
  * Micro SD card slot
  * Gimbal connector (PWB/GND/BLSP)
  * ESC connector (2W UART)
  * I2C
  * 60-pin high speed Samtec QSH-030-01-L-D-A-K expansion connector
    * 2x BLSP ([BAM Low Speed Peripheral](http://www.inforcecomputing.com/public_docs/BLSPs_on_Inforce_6540_6501_Snapdragon_805.pdf))
    * USB

## Pinouts

<aside class="warning">
Snapdragon은 DF13 커넥터를 사용하고 있지만 핀아웃은 Pixhawk와 다릅니다.
</aside>

### WiFi

  * WLAN0, WLAN1 (+BT 4.0): U.FL connector: [Taoglas adhesive antenna (DigiKey)](http://www.digikey.com/product-detail/en/FXP840.07.0055B/931-1222-ND/3877414)


### Connectors

시리얼 포트의 기본 매핑은 다음과 같습니다. :

| Device           | Description                           |
| ---------------- | ------------------------------------- |
| ```/dev/tty-1``` | J15 (next to USB)                     |
| ```/dev/tty-2``` | J13 (next to power module connector)  |
| ```/dev/tty-3``` | J12 (next to J13)                     |
| ```/dev/tty-4``` | J9 (next to J15)                      |

BAM 매핑에 커스텀 UART를 위해 "blsp.config"라는 파일을 생성하고 adb는 ```/usr/share/data/adsp```로 push 합니다. 예로 기본 매핑을 유지하기 위해서 "blsp.config"는 다음과 같아야 합니다.

tty-1 bam-9 2-wire  
tty-2 bam-8 2-wire  
tty-3 bam-6 2-wire  
tty-4 bam-2 2-wire  

각 라인의 끝에 "2-wire"를 포함하고 있는 이유는 아래 테이블에 지정한 UART에서 TX와 RX 핀만 사용을 허용하기 위해서입니다. 만약 2-wire가 지정되어 있지 않다면(타겟에 이 파일에 없는 경우) UART는 기본적으로 4-wire 모드를 사용하며 추가로 RTS/CTS flow control을 위해서 추가로 2 핀이 필요하게 됩니다. 동일한 커넥터에 I/O의 다른 타입에 문제를 발생시킬 수 있으면 핀은 RTS와 CTS 신호를 설정할 것입니다. 만약 에제로 J9(아래 설명한 바와 같이)가 UART와 I2C 장치모두에 연결되도록 했다면, pin 4와 pin 6에 있는 I2C 신호는 RTS와 CTS 신호로 설정되어 I2C SDA와 SCL 신호를 덮어쓰게 됩니다.

#### J9 / GPS

| Pin | 2-wire UART + I2C | 4-wire UART | SPI | Comment |
| -- | -- | -- | -- | -- |
| 1 | 3.3V | 3.3V | 3.3V | |
| 2 | UART2_TX | UART2_TX | SPI2_MOSI | Output (3.3V) |
| 3 | UART2_RX | UART2_RX | SPI2_MISO | Input (3.3V) |
| 4 | I2C2_SDA | UART2_RTS | SPI2_CS | (3.3V) |
| 5 | GND | GND | GND | |
| 6 | I2C2_SCL | UART2_CTS | SPI2_CLK | (3.3V) |

#### J12 / Gimbal bus

| Pin | 2-wire UART + GPIO | 4-wire UART | SPI | Comment |
| -- | -- | -- | -- | -- |
| 1 | 3.3V | 3.3V | 3.3V | |
| 2 | UART8_TX | UART8_TX | SPI8_MOSI | Output (3.3V) |
| 3 | UART8_RX | UART8_RX | SPI8_MISO | Input (3.3V) |
| 4 | APQ_GPIO_47 | UART8_RTS | SPI8_CS | (3.3V) |
| 5 | GND | GND | GND | |
| 6 | APQ_GPIO_48 | UART8_CTS | SPI8_CLK | (3.3V) |

#### J13 / ESC bus

| Pin | 2-wire UART + GPIO | 4-wire UART | SPI | Comment |
| -- | -- | -- | -- | -- |
| 1 | 5V | 5V | 5V | |
| 2 | UART6_TX | UART6_TX | SPI6_MOSI | Output (5V) |
| 3 | UART6_RX | UART6_RX | SPI6_MISO |Input (5V) |
| 4 | APQ_GPIO_29 | UART6_RTS | SPI6_CS | (5V) |
| 5 | GND | GND | GND | |
| 6 | APQ_GPIO_30 | UART6_CTS | SPI6_CLK | (5V) |

#### J14 / Power

| Pin | Signal | Comment |
| -- | -- | -- |
| 1 | 5V DC | Power input |
| 2 | GND | |
| 3 | I2C3_SCL | (5V) |
| 4 | I2C3_SDA | (5V) |

#### J15 / Radio Receiver / Sensors

| Pin | 2-wire UART + I2C | 4-wire UART | SPI | Comment |
| -- | -- | -- | -- | -- |
| 1 | 3.3V | 3.3V | 3.3V | |
| 2 | UART9_TX | UART9_TX | SPI9_MOSI | Output |
| 3 | UART9_RX | UART9_RX | SPI9_MISO | Input |
| 4 | I2C9_SDA | UART9_RTS | SPI9_CS | |
| 5 | GND | GND | GND | |
| 6 | I2C9_SCL | UART9_CTS | SPI9_CLK | |

## Peripherals

### UART to Pixracer / Pixfalcon Wiring

이 인터페이스는 Pixracer / Pixfalcon를 마치 I/O 인터페이스 보드처럼 동작하게 합니다. Pixfalcon에서는 `TELEM1`에 연결하고 Pixracer에서는 `TELEM2`에 연결합니다.

| Snapdragon J13 Pin | Signal | Comment | Pixfalcon / Pixracer Pin |
| -- | -- | -- | -- |
| 1 | 5V | Power for autopilot | 5V |
| 2 | UART6_TX | Output (5V) TX -> RX | 3 |
| 3 | UART6_RX | Input (5V) RX -> TX | 2 |
| 4 | APQ_GPIO_29 | (5V) | Not connected |
| 5 | GND | | 6 |
| 6 | APQ_GPIO_30 | (5V) | Not connected |

### GPS Wiring

3DR GPS가 5v 입력을 가진다고 설명하고 있지만, 3.3v로도 잘 동작하는 걸로 보입니다. (빌트인 regulator MIC5205는 최소 2.5v의 전압에서 동작)

| Snapdragon J9 Pin | Signal   | Comment       | 3DR GPS 6pin/4pin  | Pixfalcon GPS pin | 3DR PIXHAWK MINI GPS |
| ----------------- | ---------| ------------- | ------------------ | ----------------- | -------------------  |
| 1                 | 3.3V     | (3.3V)        | 1                  | 4                 |3 (5V)                |
| 2                 | UART2_TX | Output (3.3V) | 2/-                | 3                 |4                     |
| 3                 | UART2_RX | Input (3.3V)  | 3/-                | 2                 |5                     |
| 4                 | I2C2_SDA | (3.3V)        | -/3                | 5                 |2                     |
| 5                 | GND      |               | 6/-                | 1                 |6                     |
| 6                 | I2C2_SCL | (3.3V)        | -/2                | 6                 |1                     |

## Dimensions

![](images/hardware/hardware-snapdragon-dimensions.png)
