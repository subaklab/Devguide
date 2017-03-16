# Vision 혹은 Motion capture 시스템 사용하기

이 페이지는 PX4 기반 시스템가 GPS가 아닌 다른 소스에서 얻은 위치 데이터를 이용하도록 하는 것이 목적입니다.(VICON과 같은 motion capture 시스템, Optitrack, [ROVIO](https://github.com/ethz-asl/rovio), [SVO](https://github.com/uzh-rpg/rpg_svo) 이나 [PTAM](https://github.com/ethz-asl/ethzasl_ptam) 같은 vision 기반 estimation 시스템)

position estimation은 offboard(VICON같은) 뿐만아니라 onboard 컴퓨터가 전송할 수도 있습니다. 이 데이터는 local origin에 상대적으로 local position estimate를 업데이트하는데 사용할 수 있습니다. vision/motion capture 시스템에서 heading은 attitude estimator가 선택적으로 통합하도록 할 수 있습니다.

이 시스템은 vision을 기반으로 하는 position hold indoors나 waypoint navigation과 같은 application에서 사용할 수 있습니다.

vision의 경우, pose data를 전송하는데 사용하는 mavlink 메시지는 [VISION_POSITION_ESTIMATE](http://mavlink.org/messages/common#VISION_POSITION_ESTIMATE)이고 모든 motion capture 시스템에 대한 메시지는 [ATT_POS_MOCAP](http://mavlink.org/messages/common#ATT_POS_MOCAP) 메시지입니다.

mavros ROS-Mavlink 인터페이스는 이런 메시지를 전송하기 위한 기본 구현부를 가지고 있습니다. 순수 C/C++ 코드와 MAVLink() 라이브러리를 직접 사용해서 전송할 수 있습니다.

## external pose input을 가능하도록 설정하기
vison/mocap 사용을 활성/비활성화 하기 위해서 QGroundControl이나 NSH shell에서 2개 parameter를 설정해야 한다.

<aside class="note">
시스템 parameter인 ```CBRK_NO_VISION```를 0으로 설정하면 vision position integration을 활성화시킬 수 있음
</aside>

<aside class="note">
시스템 parameter인 ```ATT_EXT_HDG_M```를 1이나 2로 설정하면 external heading integration을 활성화시킬 수 있음. 1로 설정하면 vision이 사용되고, 2로 설정하면 mocap heading을 사용을 활성화시킴.
</aside>
