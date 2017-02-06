# 새 Airframe 설정 추가하기

PX4는 airframe을 시작할때, 기존 설정을 이용합니다. 직관적으로 설정을 추가할 수 있습니다. : 새 파일을 생성해서 [init.d folder](https://github.com/PX4/Firmware/tree/master/ROMFS/px4fmu_common/init.d) 내에 free autosart ID를 추가해서 [빌드 및 업로드](starting-building.md)합니다.

새로 자신만의 설정을 만드는 것을 원치 않는 개발자라면 microSD 카드에 텍스트파일을 이용해서 기존 설정을 수정합니다. 상세한 내용은 [custom system startup](advanced-system-startup.md)을 참고하세요.

## Airframe 설정

airframe 설정은 3개 주요 블록으로 구성 :

  * 구동시킬 app들. (예로 multicopter이나 fixed wing controllers)
  * 시스템의 물리적 설정(예로 plane, wing 혹은 multicopter) mixer라고 부름.
  * 튜닝 게임 (Tuning gains)

이 3가지는 독립적입니다. 이말은 많은 설정이 airframe의 동일한 물리적 레이아웃을 공유하며 동일한 applications을 구동시키며 가장 큰 차이는 튜닝 게인이 다르다는 것입니다.

모든 설정은 [ROMFS/px4fmu_common/init.d](https://github.com/PX4/Firmware/tree/master/ROMFS/px4fmu_common/init.d) 폴더에 저장됩니다.
모든 믹서는 [ROMFS/px4fmu_common/mixers](https://github.com/PX4/Firmware/tree/master/ROMFS/px4fmu_common/mixers) 폴더에 저장됩니다.

### Config file

일반적인 설정 파일은 아래와 같습니다.

```bash
#!nsh
#
# @name Wing Wing (aka Z-84) Flying Wing
#
# @url https://pixhawk.org/platforms/planes/z-84_wing_wing
#
# @type Flying Wing
#
# @output MAIN1 left aileron
# @output MAIN2 right aileron
# @output MAIN4 throttle
#
# @output AUX1 feed-through of RC AUX1 channel
# @output AUX2 feed-through of RC AUX2 channel
# @output AUX3 feed-through of RC AUX3 channel
#
# @maintainer Lorenz Meier <lorenz@px4.io>
#

sh /etc/init.d/rc.fw_defaults

if [ $AUTOCNF == yes ]
then
	param set BAT_N_CELLS 2
	param set FW_AIRSPD_MAX 15
	param set FW_AIRSPD_MIN 10
	param set FW_AIRSPD_TRIM 13
	param set FW_ATT_TC 0.3
	param set FW_L1_DAMPING 0.74
	param set FW_L1_PERIOD 16
	param set FW_LND_ANG 15
	param set FW_LND_FLALT 5
	param set FW_LND_HHDIST 15
	param set FW_LND_HVIRT 13
	param set FW_LND_TLALT 5
	param set FW_THR_LND_MAX 0
	param set FW_PR_FF 0.35
	param set FW_RR_FF 0.6
	param set FW_RR_P 0.04
fi

# Configure this as plane
set MAV_TYPE 1
# Set mixer
set MIXER wingwing
# Provide ESC a constant 1000 us pulse
set PWM_OUT 4
set PWM_DISARMED 1000
```

중요 : 채널을 reverse할려고 할때, RC 트랜스미터나 `RC1_REV`로 절대로 바꾸지 마세요. 채널은 매뉴얼 모드로 비행할 때만 reverse되므로 만약 autopilot flight mode로 변환되면, 채널 출력이 잘못된 상태로 남게 됩니다.(RC 시그널만 반대로 변환) 따라서 채널을 올바르게 할당하려면, `PWM_MAIN_REV1`으로 PWM 신호를 변경하거나 믹서에서 출력 스케일링과 출력 범위를 변경하여야 합니다. (아래 참고)

### Mixer file

일반적인 설정 파일은 아래와 같다.

<aside class="note">
서보/모터 plug는 이 파일에 믹서와 관련이 있습니다.
</aside>

MAIN1은 왼쪽 aileron, MAIN2는 오른쪽 aileron, MAIN3은 빈값(Z: zero mizer)이고 MAIN4는 throttle(일반적으로 fixed wing 설정에 output 4에 throttle 유지)입니다.

믹서는 -1..+1에 대응해서 -10000에서 10000값으로 정규화시켜 인코딩됩니다.

```
M: 2
O:      10000  10000      0 -10000  10000
S: 0 0  -6000  -6000      0 -10000  10000
S: 0 1   6500   6500      0 -10000  10000
```

왼쪽에서 오른쪽으로 각 숫자의 의미 :

  * M: 2개 input에 대한 2개 scaler 표시
  * O: output scaling(*1 in negative, *1 in positive), offset (여기서는 0), output range (여기서는 -1..+1)를 표시. 만약 여러분의 PWM 신호를 invert 시키고자 한다면, 2개 output scaling와 2개 output range 숫자 모두의 sign을 변경해야만 함.(```O:      -10000  -10000      0 10000  -10000```)
  * S: 첫번째 input scaler : control group #0의 입력 (attitude controls)과 첫번째 input (roll)을 입력으로 사용. input * 0.6을 scale하고 sign(-0.6은 -6000으로 스케일)을 revert시킴. 여기서는 offset (0)이 적용되지 않고 output에 full range (-1..+1)를 적용.
  * S: 두번째 input scaler : control group #0의 입력 (attitude controls)과 두번째 input (pitch)을 입력으로 사용. input * 0.65로 scale하고 sign(-0.65는 -6500으로 스케일)을 revert시킴. 여기서는 offset (0)이 적용되지 않고 output에 full range (-1..+1)를 적용.

2개 scaler가 추가되었습니다. 이것은 날개가 
Both scalers are added, which for a flying wing means the control surface takes maximum 60% deflection from roll and 65% deflection from pitch. As it is over-comitted with 125% total deflection for maximum pitch and roll, it means the first channel (roll here) has priority over the second channel / scaler (pitch).

The complete mixer looks like this:


```bash
Delta-wing mixer for PX4FMU
===========================

Designed for Wing Wing Z-84

This file defines mixers suitable for controlling a delta wing aircraft using
PX4FMU. The configuration assumes the elevon servos are connected to PX4FMU
servo outputs 0 and 1 and the motor speed control to output 3. Output 2 is
assumed to be unused.

Inputs to the mixer come from channel group 0 (vehicle attitude), channels 0
(roll), 1 (pitch) and 3 (thrust).

See the README for more information on the scaler format.

Elevon mixers
-------------
Three scalers total (output, roll, pitch).

On the assumption that the two elevon servos are physically reversed, the pitch
input is inverted between the two servos.

The scaling factor for roll inputs is adjusted to implement differential travel
for the elevons.

M: 2
O:      10000  10000      0 -10000  10000
S: 0 0  -6000  -6000      0 -10000  10000
S: 0 1   6500   6500      0 -10000  10000

M: 2
O:      10000  10000      0 -10000  10000
S: 0 0  -6000  -6000      0 -10000  10000
S: 0 1  -6500  -6500      0 -10000  10000

Output 2
--------
This mixer is empty.

Z:

Motor speed mixer
-----------------
Two scalers total (output, thrust).

This mixer generates a full-range output (-1 to 1) from an input in the (0 - 1)
range.  Inputs below zero are treated as zero.

M: 1
O:      10000  10000      0 -10000  10000
S: 0 3      0  20000 -10000 -10000  10000

```
