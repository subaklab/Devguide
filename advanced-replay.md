# 시스템 수준에서 재연 (System-wide Replay)
ORB 메시지를 바탕으로 시스템 특정 부분의 동작을 기록하고 이를 재연하는 것이 가능합니다. 이를 가능하게 하기 위해서 새로운 logger를 활성화시켜야 합니다.(`SYS_LOGGER`를 1로 설정)

재연은 실제 데이터를 기반으로 다른 파라미터 값의 영향을 테스트하는데 효과적입니다. 다른 estimator들을 비교할 수도 있습니다.

## 전제조건
가장 먼저 해야하는 일은 재연해야 하는 모듈을 구별해내는 것입니다.
다음으로 이 모듈의 들어오는 모든 입력을 아는 것입니다.(subscribed ORB topic을 통해서) 시스템 규모에서 재연을 위해서는 모든 하드웨어 입력 즉 센서, RC 입력, mavlink 명령, 파일 시스템이 필요합니다.

구별가능한 모든 topic들은 최대 rate로 로그를 남겨야 합니다.([logging](advanced-logging.md) 참고) `ekf2` 경우에는 이미 기본적으로 관련 topic에 대해서 로그를 남기고 있습니다.

재연하는 모든 topic은 단일 timestamp를 사용하는 것이 중요합니다. `timestamp` 필드에 자동으로 생성됩니다. 더 많은 timestamp가 있어야 한다면, main timestamp와 관련되어 있어야 합니다. 예제로 [sensor_combined.msg](https://github.com/PX4/Firmware/blob/master/msg/sensor_combined.msg)를 참고하세요.
이에 대한 이유는 아래에서 설명합니다.


## 사용

- 먼저 재연할 파일을 선택하고 타겟(Firmware 디렉토리 내에 있는)을 빌드합니다.
```sh
export replay=<absolute_path_to_log_file.ulg>
make posix_sitl_default
```
  여기서는 `build_posix_sitl_default_replay`라는 별도의 빌드 디렉토리에 결과물을 생성합니다. (이 파라미터는 일반 빌드에 영향을 주지 않음) 재연을 위해 posix SITL build target을 선택하는 것이 가능합니다. 빌드 시스템은 `replay` 환경변수를 통해 현재 재연 모드에 있는지 알 수 있습니다.
- ORB publisher rule 파일을
  `build_posix_sitl_default_replay/tmp/rootfs/orb_publisher.rules`에 추가합니다.
  이 파일은 어떤 모듈이 어떤 메시지를 publish하도록 허용하는지 정의합니다.
  다음과 같은 포맷으로 :
```
restrict_topics: <topic1>, <topic2>, ..., <topicN>
module: <module>
ignore_others: <true/false>
```

  It means that the given list of topics should only be published by `<module>`
  (which is the command name). Publications to any of these topics from another
  module are silently ignored. If `ignore_others` is `true`, then publications
  to other topics from `<module>` are ignored.

  For replay, we only want the `replay` module to be able to publish the
  previously identified list of opics. So for replaying `ekf2`, the rules file
  looks like this:
```
restrict_topics: sensor_combined, vehicle_gps_position, vehicle_land_detected
module: replay
ignore_others: true
```
  This allows that the modules, which usually publish these topics, don't need
  to be disabled for replay.

- Optional: setup parameter overrides in the file
  `build_posix_sitl_default_replay/tmp/rootfs/replay_params.txt`.
  This file should contain a list of `<param_name> <value>`, like:
```
EKF2_GB_NOISE 0.001
```
  By default, all parameters from the log file are applied. When a parameter
  changed during recording, it will be changed as well at the right time during
  replay. A parameter in the `replay_params.txt` will override the value and
  changes to it from the log file will not be applied.
- Optional: copy `dataman` missions file from the SD card to the build
  directory. Only necessary if a mission should be replayed.
- Start the replay:
```sh
  make posix_sitl_default jmavsim
```
  This will automatically open the log file, apply the parameters and start
  to replay. Once done, it will be reported and the process can be exited. Then
  the newly generated log file can be analyzed, it has `_replayed` appended to
  its file name.

  Note that the above command will show the simulator as well, but depending on
  what is being replayed, it will not show what's actually going on. It's
  possible to connect via QGC and e.g. view the changing attitude during replay.

- Finally, unset the environment variable, so that the normal build targets
  are used again:
```sh
unset replay
```

### Important Notes

- During replay, all dropouts in the log file are reported. These have a
  negative effect on replay and thus it should be taken care that dropouts are
  avoided during recording.
- It is currently only possible to replay in 'real-time', meaning as fast as the
  recording was done. This is planned to be extended in the future.
- A message that has a timestamp of 0 will be considered invalid and not be
  replayed.


## Behind the Scenes

Replay is split into 3 components:
- a replay module
- ORB publisher rules
- time handling

The replay module reads the log and publishes the messages with the same speed
as they were recorded. A constant offset is added to the timestamp of each
message to match the current system time (this is the reason why all other
timestamps need to be relative). The command `replay tryapplyparams` is executed
before all other modules are loaded and applies the parameters from the log and
user-set parameters. Then as the last command, `replay trystart` will again
apply the parameters and start the actual replay. Both commands do nothing if
the environment variable `replay` is not set.


The ORB publisher rules allow to select which part of the system is replayed, as
described above. They are only compiled for the posix SITL targets.


The **time handling** is still an **open point**, and needs to be implemented.
