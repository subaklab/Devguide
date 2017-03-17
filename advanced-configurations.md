# 파라미터와 설정 (Parameters & Configurations)

PX4는 설정관련 정보를 저장을 위해 param subsystem(float과 inte32_t 값의 일반적인 테이블)과 텍스트 파일(mixer와 startup 스크립트)을 사용합니다.

[system startup](advanced-system-startup.md)과 [airframe configurations](airframes-adding-a-new-frame.md)가 어떻게 동작하는지는 다른 페이지에서 다룹니다. 이 섹션에서는 param subsystem에 대해서 상세하게 알아봅시다.

## Commandline 사용법

PX4의 [system console](advanced-system-console.md)은 ```param```이라는 툴을 제공합니다. 이 툴을 이용하면 파라미터를 설정, 설정값 읽기 및 저장 그리고 파일에서 설정값을 가지고 오거나 저장하기가 가능합니다.

### 파라미터 값 읽기/쓰기

param show 명령은 모든 시스템 파라미터를 보여줍니다. :

```sh
param show
```

파라미터 이름에 와일드카드를 사용하면 조합가능한 모든 파라미터를 보여줍니다. :

```sh
nsh> param show RC_MAP_A*
Symbols: x = used, + = saved, * = unsaved
x   RC_MAP_AUX1 [359,498] : 0
x   RC_MAP_AUX2 [360,499] : 0
x   RC_MAP_AUX3 [361,500] : 0
x   RC_MAP_ACRO_SW [375,514] : 0

 723 parameters total, 532 used.
```

### 파라미터 익스포팅과 로딩 (Exporting and Loading Parameters)

표준 save 명령은 현재 기본 파일에 parameter를 저장합니다.:

```sh
param save
```

인자를 제공하는 경우라면, 파라미터를 새로운 위치에 저장합니다.:

```sh
param save /fs/microsd/vtol_param_backup
```

파라미터를 로드하는 2가지 명령이 있습니다: ```param load```는 파일을 로드하고 이 파일에 있는 값으로 현재 파라미터를 교체합니다. 결국 이 파라미터가 저장될 때 있던 상태를 결과적으로 1:1로 복사하는 것이 됩니다. ```param import```는 좀더 영리합니다. : 기본값에서 변경된 파라미터 값만 변경합니다. 처음으로 보드를 칼리브레이션할 때 유용하며 따라서 시스템 설정의 나머지 부분을 덮어쓰기 하지 않고 칼리브레이션 데이터를 import합니다.

현재 파라미터를 덮어쓰기 :

```sh
param load /fs/microsd/vtol_param_backup
```

현재 파라미터를 저장했던 파라미터와 합치기(기본값이 아니면서 저장되어 있는 값이 우선권을 갖습니다) :

```sh
param import /fs/microsd/vtol_param_backup
```

## C / C++ API

파라미터 값에 접근하는데 사용하는 C와 별도 C++ API가 있습니다.

<aside class="todo">
Discuss param C / C++ API.
</aside>

<div class="host-code"></div>

```C
int32_t param = 0;
param_get(param_find("PARAM_NAME"), &param);
```

## 파라미터 메타 데이터 (Parameter Meta Data)

PX4는 사용자가 직접 파라미터를 접하는 기회를 주기 위해서 확장 파라미터 메타 데이타 시스템을 사용용합니다. 적절한 메타 데이터는 ground station에서 훌륭한 사용자 경험을 제공하는데 결정적인 역할을 합니다.

일반적인 파리미터 메타 데이터 섹션은 다음과 같습니다.
A typical parameter meta data section will look like this:

```C++
/**
 * Pitch P gain
 *
 * Pitch proportional gain, i.e. desired angular speed in rad/s for error 1 rad.
 *
 * @unit 1/s
 * @min 0.0
 * @max 10
 * @decimal 2
 * @increment 0.0005
 * @reboot_required true
 * @group Multicopter Attitude Control
 */
PARAM_DEFINE_FLOAT(MC_PITCH_P, 6.5f);
```

코드 라인 어디서 이런 방식을 사용하는 곳:

```C++
/**
 * <title>
 *
 * <longer description, can be multi-line>
 *
 * @unit <the unit, e.g. m for meters>
 * @min <the minimum sane value. Can be overriden by the user>
 * @max <the maximum sane value. Can be overriden by the user>
 * @decimal <the minimum sane value. Can be overriden by the user>
 * @increment <the "ticks" in which this value will increment in the UI>
 * @reboot_required true <add this if changing the param requires a system restart>
 * @group <a title for parameters which form a group>
 */
PARAM_DEFINE_FLOAT(MC_PITCH_P, 6.5f);
```
