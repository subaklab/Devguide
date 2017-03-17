# UAVCAN Enumeration과 설정

<aside class="note">
아래와 같이 'Enable UAVCAN' 체크박스를 체크하면 기본 모터 출력 버스로 UAVCAN을 활성화 시키게 됩니다. QGroundControl parameter 편집기에서 UAVCAN_ENABLE parameter는 '3'으로 설정할 수도 있습니다. CAN을 활성화하려면 '2'로 설정하고 모터 출력은 PWM 그대로 둡니다.
</aside>

[QGroundControl](qgroundcontrol-intro.md)을 사용합니다. Setup 창으로 변환하고 왼쪽에 있는 Power Configuration을 선택합니다. 'start assignment' 버튼을 클릭합니다.

첫번째 beep이 울리고 난 후, 첫번째 ESC에 있는 프로펠러가 올바른 방향으로 회전합니다. ESC는 각각이 enumeration할 때 매번 beep이 울립니다. [motor map](airframes-motor-map.md)에서 보는 바와 같이 순서대로 모든 motor controller에 대해서 이 단계를 반복합니다. 이 단계는 한번만 수행되며 펌웨어를 업그레이드하고 난 후에는 반복할 필요가 없습니다.

![UAVCAN Enumeration Controls (이미지의 오른쪽 아래)](images/uavcan-qgc-setup.png)
