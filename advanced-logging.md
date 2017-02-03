# Logging

여기서는 새로운 logger에 대해서 알아봅니다. `SYS_LOGGER`를 1로 설정합니다.

logger는 포함하는 모든 필드와 함께 ORB topic을 log할 수 있습니다. 모든 것은 `.msg` 파일에서 생성되며 topic 이름은 지정해줘 합니다. 선택적으로 interval 파라미터는 특정 topic에 대해서 최대 logging rate를 지정합니다. 모든 기존 topic 인스턴스는 로그를 남기게 됩니다.

log 출력 포맷은 [ULog](advanced-ulog-file-format.md)입니다.

## 사용
기본적으로 logging은 arming이 되면 자동으로 시작되고 disarming이 되면 멈추게 됩니다. 매번 arming 세션마다 새로운 log 파일 SD카드에 생성됩니다. 현재 상태를 보여주기 위해서 콘솔에서 `logger status`를 사용하세요. 즉시 logging을 시작시키고자 한다면 `logger on`를 사용합니다. 이렇게하면 시스템이 이미 arm 상태가 되어서 arming 상태를 무시하게 됩니다. `logger off`를 하면 원상태로 돌아갑니다.

Use
```
logger help
```
지원하는 모든 logger 명령과 파라미터의 목록을 보여줍니다.


## Configuration
로그로 기록되는 topic의 목적은 SD카드에 파일로 커스터마이즈될 수 있습니다. topic의 목록을 가지고 카드에 `etc/logging/logger_topics.txt` 파일을 생성합니다:
```
<topic_name>, <interval>
```
`<interval>`은 선택사항이며 만약 지정하면 해당 topic이 로그 메시지를 기록하는 사이에 ms 단위의 시간 차이를 정의합니다. 만약 지정하지 않으면 topic은 최대 rate로 로그를 기록합니다.

이 파일에 있는 topic은 모두 기본 로그의 topic으로 대체됩니다.

## Scripts
[pyulog](https://github.com/PX4/pyulog) 저장소에 
There are several scripts to analyze and convert logging files in the
[pyulog](https://github.com/PX4/pyulog) repository.

## Dropouts

Logging dropouts are undesired and there are a few factors that influence the
amount of dropouts:
- Most SD cards we tested exhibit multiple pauses per minute. This shows
  itself as a several 100 ms delay during a write command. It causes a dropout
  if the write buffer fills up during this time. This effect depends on the SD
  card (see below).
- Formatting an SD card can help to prevent dropouts.
- Increasing the log buffer helps.
- Decrease the logging rate of selected topics or remove unneeded topics from
  being logged (`info.py <file>` is useful for this).

## SD Cards
The following provides performance results for different SD cards.
Tests were done on a Pixracer; the results are applicable to Pixhawk as well.

| SD Card | Mean Seq. Write Speed [KB/s] | Max Write Time / Block (average) [ms] |
| -- | -- | -- |
| SanDisk Extreme U3 32GB | 461 | **15** |
| Sandisk Ultra Class 10 8GB | 348 | 40 |
| Sandisk Class 4 8GB | 212 | 60 |
| SanDisk Class 10 32 GB (High Endurance Video Monitoring Card) | 331 | 220 |
| Lexar U1 (Class 10), 16GB High-Performance | 209 | 150 |
| Sandisk Ultra PLUS Class 10 16GB | 196 | 500 |
| Sandisk Pixtor Class 10 16GB | 334 | 250 |
| Sandisk Extreme PLUS Class 10 32GB | 332 | 150 |

More important than the mean write speed is the maximum write time per block (of
4 KB). This defines the minimum buffer size: the larger this maximum, the larger
the log buffer needs to be to avoid dropouts. Logging bandwidth with the default
topics is around 50 KB/s, which all of the SD cards satisfy.

By far the best card we know so far is the **SanDisk Extreme U3 32GB**. This
card is recommended, because it does not exhibit write time spikes (and thus
virtually no dropouts). Different card sizes might work equally well, but the
performance is usually different.

You can test your own SD card with `sd_bench -r 50`, and report the results to
https://github.com/PX4/Firmware/issues/4634.

## Log Streaming
The traditional and still fully supported way to do logging is using an SD card
on the FMU. However there is an alternative, log streaming, which sends the
same logging data via MAVLink. This method can be used for example in cases
where the FMU does not have an SD card slot (eg. Intel Aero) or simply to avoid
having to deal with SD cards. Both methods can be used independently and at the
same time.

The requirement is that the link provides at least ~50KB/s, so for example a
WiFi link. And only one client can request log streaming at the same time. The
connection does not need to be reliable, the protocol is designed to handle
drops.

There are different clients that support ulog streaming:
- `mavlink_ulog_streaming.py` script in Firmware/Tools.
- QGroundControl:
![](images/qgc_log_streaming.png)
- [MAVGCL](https://github.com/ecmnet/MAVGCL)

### Diagnostics
- If log streaming does not start, make sure the `logger` is running (see
  above), and inspect the console output while starting.
- Log streaming uses a maximum of 70% of the configured mavlink rate (`-r`
  parameter). If more is needed, messages are dropped. The currently used
  percentage can be inspected with `mavlink status` (1.8% is used in this
  example):
```
instance #0:
        GCS heartbeat:  160955 us ago
        mavlink chan: #0
        type:           GENERIC LINK OR RADIO
        flow control:   OFF
        rates:
        tx: 95.781 kB/s
        txerr: 0.000 kB/s
        rx: 0.021 kB/s
        rate mult: 1.000
        ULog rate: 1.8% of max 70.0%
        accepting commands: YES
        MAVLink version: 2
        transport protocol: UDP (14556)
```
  Also make sure `txerr` stays at 0. If this goes up, either the NuttX sending
  buffer is too small, the physical link is saturated or the hardware is too
  slow to handle the data.
