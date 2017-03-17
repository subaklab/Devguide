# Mixing과 Actuators

PX4 아키텍쳐는 에어프레임 레이아웃이 핵심 컨트롤러에게 특별한 처리를 요구하지 않도록 설계하였습니다.

Mixing이란 움직이는 명령(예로 `오른쪽으로 돌기`)을 받아서 이를 모터나 서보를 제어하는 actuator 명령으로 변환하는 것을 말합니다.aileron마다 하나의 서보가 있는 비행체에 대해서 그것들 중에 하나는 high로 명령을 주고 나머지는 low로 명령을 준다는 뜻입니다. 동일하게 멀티콥터에도 적용 : 앞으로 pitching은 모든 모터의 속도를 변경해야 합니다.

실제 attitude controller로부터 mixer 로직을 분리를 통해서 엄청나게 재사용성을 향상시킬 수 있습니다.

## Control Pipeline

특정 컨트롤러는 mixer에 특별히 정규화된 힘이나 필요한 토크(-1..+1로 스케일된)를 보냅니다. 이렇게 해서 개별 actuator를 설정합니다. 출력 드라이버(예로 UART, UAVCAN 혹은 PWM)은 이를 actuator에 맞게 native 단위로 스케일링 시킵니다.(에로 PWM의 값을 1300으로)

{% mermaid %}
graph LR;
  att_ctrl[Attitude Controller] --> act_group0[Actuator Control Group 0]
  gimbal_ctrl[Gimbal Controller] --> act_group2[Actuator Control Group 2]
  act_group0 --> output_group5[Actuator 5]
  act_group0 --> output_group6[Actuator 6]
  act_group2[Actuator Control Group 2] --> output_group0[Actuator 5]

  {% endmermaid %}
## Control Groups

PX4는 control group(입력)과 output group을 사용합니다. 개념상으로 이는 매울 단순합니다. control group은 core flight control에 대해서 `attitude`를 들 수 있고, payload에 대해서는 `gimbal`을 들 수 있습니다. output group은 하나의 물리적인 버스로 서버에 대해서 처음 8개 PWM 출력을 예로 들 수 있습니다. 이들 각 group은 8개의 정규화된(-1..+1) 명령 포트를 가집니다. 이는 mixer를 통해서 매핑되고 스케일링 됩니다. mixer는 해당 control의 8개 신호 각각이 8개 출력에 연결되는 방법을 정의합니다.
단순한 비행기의 경우 제어 0 (roll)이 출력 0 (aileron)에 바로 연결되어 있습니다. 멀티콥터의 경우 약간 다릅니다. control 0 (roll)은 모드 4개 모터와 연결되며 throttle과 결합되어 있습니다.

#### Control Group #0 (Flight Control)

 * 0: roll (-1..1)
 * 1: pitch (-1..1)
 * 2: yaw (-1..1)
 * 3: throttle (0..1 normal range, -1..1 for variable pitch / thrust reversers)
 * 4: flaps (-1..1)
 * 5: spoilers (-1..1)
 * 6: airbrakes (-1..1)
 * 7: landing gear (-1..1)

#### Control Group #1 (Flight Control VTOL/Alternate)

 * 0: roll ALT (-1..1)
 * 1: pitch ALT (-1..1)
 * 2: yaw ALT (-1..1)
 * 3: throttle ALT (0..1 normal range, -1..1 for variable pitch / thrust reversers)
 * 4: reserved / aux0
 * 5: reserved / aux1
 * 6: reserved / aux2
 * 7: reserved / aux3

#### Control Group #2 (Gimbal)

 * 0: gimbal roll
 * 1: gimbal pitch
 * 2: gimbal yaw
 * 3: gimbal shutter
 * 4: reserved
 * 5: reserved
 * 6: reserved
 * 7: reserved (parachute, -1..1)

#### Control Group #3 (Manual Passthrough)

 * 0: RC roll
 * 1: RC pitch
 * 2: RC yaw
 * 3: RC throttle
 * 4: RC mode switch
 * 5: RC aux1
 * 6: RC aux2
 * 7: RC aux3

#### Control Group #6 (First Payload)

 * 0: function 0 (default: parachute)
 * 1: function 1
 * 2: function 2
 * 3: function 3
 * 4: function 4
 * 5: function 5
 * 6: function 6
 * 7: function 7

### Virtual Control Groups

이 그룹들은 mixer 입력이 아닙니다. 하지만 fixed wing과 멀티콥터 controller 출력을 VTOL governor module로 들어가도록 하는 meta-channels로 동작합니다.

#### Control Group #4 (Flight Control MC VIRTUAL)

 * 0: roll ALT (-1..1)
 * 1: pitch ALT (-1..1)
 * 2: yaw ALT (-1..1)
 * 3: throttle ALT (0..1 normal range, -1..1 for variable pitch / thrust reversers)
 * 4: reserved / aux0
 * 5: reserved / aux1
 * 6: reserved / aux2
 * 7: reserved / aux3

#### Control Group #5 (Flight Control FW VIRTUAL)

 * 0: roll ALT (-1..1)
 * 1: pitch ALT (-1..1)
 * 2: yaw ALT (-1..1)
 * 3: throttle ALT (0..1 normal range, -1..1 for variable pitch / thrust reversers)
 * 4: reserved / aux0
 * 5: reserved / aux1
 * 6: reserved / aux2
 * 7: reserved / aux3

## Mapping

여러 가지 control group(flight control, payload 등등)과 output group(처음 8 PWM 출력, UAVCAN 등등)이 있기 때문에, 하나의 control group은 여러 output group에 명령을 보낼 수 있습니다.

{% mermaid %}
graph TD;
  actuator_group_0-->output_group_5
  actuator_group_0-->output_group_6
  actuator_group_1-->output_group_0
{% endmermaid %}

## PX4 mixer 정의

`ROMFS/px4fmu_common/mixers`에 있는 파일들은 미리 정의한 에어프레임에 사용되는 mixer를 구현하고 있습니다. 커스텀용도의 기초로 사용할 수도 있고 일반 테스팅 목적으로도 사용할 수 있습니다.

### 문법

mixer 정의는 텍스트 파일로 되어 있어서 단일 대문자로 시작되고 뒤에 콜론이 오는 라인이 중요합니다. 그 이외 다른 모든 라인들은 무시되며 서술하는 텍스트는 정의 부분과 자유롭게 섞일 수 있습니다.

각 파일에는 1개 이상의 mixer를 정의할 수 있습니다. actuator로 mixer 할당은 mixer 정의를 읽는 장치에 특화되고 mixer로 생성된 actuator output 갯수는 해당 mixer에 특화됩니다.

예를 들면 : 각각 단순하거나 null mixer는 output 1에서 x까지 순서대로 할당되어 mixer 파일에 나타납니다.

mixer는 아래 형태의 라인으로 시작합니다.

	<tag>: <mixer arguments>

tag는 mixer 타입을 선택하는 부분으로 'M'은 단순한 summing mixer로 'R'은 멀티로터 mixer를 뜻합니다.

#### Null Mixer ####

null mixer는 control을 사용하지 않고 단일 actuator output 을 생성합니다. 이 값은 항상 0입니다. 일반적으로 null mixer는 특정 actuator output의 특정 패턴을 달성하기 위해 mixer의 집합으로 placeholder로 사용됩니다.

null mixer 정의의 형태 :

	Z:

#### Simple Mixer ####

simple mixer는 0 혹은 여러 control input을 단일 actuator output으로 결합시킵니다. 입력은 스케일링하고 output scaler를 적용하기 전에 mixing function은 결과를 sum합니다.

simple mixer 정의는 다음과 같이 시작됩니다 :

	M: <control count>
	O: <-ve scale> <+ve scale> <offset> <lower limit> <upper limit>

If &lt;control count&gt; is zero, the sum is effectively zero and the mixer will
output a fixed value that is &lt;offset&gt; constrained by &lt;lower limit&gt;
and &lt;upper limit&gt;.

The second line defines the output scaler with scaler parameters as discussed
above. Whilst the calculations are performed as floating-point operations, the
values stored in the definition file are scaled by a factor of 10000; i.e. an
offset of -0.5 is encoded as -5000.

The definition continues with &lt;control count&gt; entries describing the control
inputs and their scaling, in the form:

	S: <group> <index> <-ve scale> <+ve scale> <offset> <lower limit> <upper limit>

The &lt;group&gt; value identifies the control group from which the scaler will read,
and the &lt;index&gt; value an offset within that group.  These values are specific to
the device reading the mixer definition.

When used to mix vehicle controls, mixer group zero is the vehicle attitude
control group, and index values zero through three are normally roll, pitch,
yaw and thrust respectively.

The remaining fields on the line configure the control scaler with parameters as
discussed above. Whilst the calculations are performed as floating-point
operations, the values stored in the definition file are scaled by a factor of
10000; i.e. an offset of -0.5 is encoded as -5000.

#### Multirotor Mixer ####

The multirotor mixer combines four control inputs (roll, pitch, yaw, thrust)
into a set of actuator outputs intended to drive motor speed controllers.

The mixer definition is a single line of the form:

	R: <geometry> <roll scale> <pitch scale> <yaw scale> <deadband>

The supported geometries include:

 * 4x - quadrotor in X configuration
 * 4+ - quadrotor in + configuration
 * 6x - hexcopter in X configuration
 * 6+ - hexcopter in + configuration
 * 8x - octocopter in X configuration
 * 8+ - octocopter in + configuration

Each of the roll, pitch and yaw scale values determine scaling of the roll,
pitch and yaw controls relative to the thrust control.  Whilst the calculations
are performed as floating-point operations, the values stored in the definition
file are scaled by a factor of 10000; i.e. an factor of 0.5 is encoded as 5000.

Roll, pitch and yaw inputs are expected to range from -1.0 to 1.0, whilst the
thrust input ranges from 0.0 to 1.0.  Output for each actuator is in the
range -1.0 to 1.0.

In the case where an actuator saturates, all actuator values are rescaled so that
the saturating actuator is limited to 1.0.
