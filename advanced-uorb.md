# uORB Messaging

## 소개

uORB는 동기화 publish() / subscribe() 메시징 API로 thread 간/프로세스 간 통신에 사용됩니다.

C++에서 어떻게 사용하는지는 [tutorial](tutorial-hello-sky.md)를 참고하세요.

uORB는 많은 application이 사용하므로 부팅시에 자동으로 시작됩니다. `uorb start`명령으로 시작됩니다. 단위 테스트는 `uorb test`로 시작시킬 수 있습니다.

## 새로운 topic 추가하기

새로운 topic을 추가하기 위해서, `msg/` 디렉토리에 새로운 `.msg` 확장자의 파일을 생성하고 파일 이름을 `msg/CMakeLists.txt`목록에 추가합니다. 여기서부터 C/C++ 코드로 자동으로 생성됩니다.

지원하는 타입을 볼려면 기존 `msg` 파일을 참고하세요. 메시지는 다른 메시지에 속해 있을 수도 있습니다. 각 생성된 C/C++ 구조체에 대해서, `uint64_t timestamp` 필드가 추가됩니다. 이것은 logger로 사용되어, 메시지를 로깅할때 여기에 값을 기입하게 됩니다.

코드에 topic을 사용하기 위해서 헤더를 추가합니다:
```
#include <uORB/topics/topic_name.h>
```

`.msg` 파일에 다음과 같이 한 줄 추가하면, 여러개의 독립적인 topic 인스턴스에 대해서 단일 메시지 정의가 사용될 수 있습니다. :
```
# TOPICS mission offboard_mission onboard_mission
```
코드내에서 topic id로 사용 : `ORB_ID(offboard_mission)`


## Publishing

interrupt context(`hrt_call` API가 호출하는 함수)를 포함해서 시스템 어디에서든 topic을 publish할 수 있습니다. 그러나 topic을 advertise하는 것은 interrupt context 외부에서만 가능합니다. 나중에 publish하는 프로세스와 topic을 advertise하는 프로세스는 동일해야 합니다.

## Listing Topics and Listening in

<aside class="note">
'listener' 명령은 Pixracer (FMUv4)와 Linux / OS X 에서만 가능합니다.
</aside>

모든 topic을 리스트로 보기 위해서 파일 핸들의 리스트를 봐야합니다.
To list all topics, list the file handles:

```sh
ls /obj
```

5개 메시지에 대해서 1개 topic의 content를 listen하기 위해서 listener를 실행합니다.: listener:

```sh
listener sensor_accel 5
```

출력은 topic의 n회 내용을 보여줍니다.:

```sh
TOPIC: sensor_accel #3
timestamp: 84978861
integral_dt: 4044
error_count: 0
x: -1
y: 2
z: 100
x_integral: -0
y_integral: 0
z_integral: 0
temperature: 46
range_m_s2: 78
scaling: 0

TOPIC: sensor_accel #4
timestamp: 85010833
integral_dt: 3980
error_count: 0
x: -1
y: 2
z: 100
x_integral: -0
y_integral: 0
z_integral: 0
temperature: 46
range_m_s2: 78
scaling: 0
```
