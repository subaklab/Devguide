# OctoMap

[The OctoMap library](http://octomap.github.io/)는 3D 그리드 매핑 방식을 구현합니다. 이 가이드에서는 [Rotors Simulator](https://github.com/ethz-asl/rotors_simulator/wiki/RotorS-Simulator)로 사용하는 방법을 다룹니다.

## 설치

ROS, Gazebo, Rotors Simulator 플러그인이 설치되어 있어야 합니다. Rotors 시뮬레이터에서 다음 [instructions](https://github.com/ethz-asl/rotors_simulator)를 따라서 설치합니다.

다음으로, OctoMap library를 설치합니다.
<div class="host-code"></div>
```sh
	sudo apt-get install ros-indigo-octomap ros-indigo-octomap-mapping
	rosdep install octomap_mapping
	rosmake octomap_mapping
```

~/catkin_ws/src/rotors_simulator/rotors_gazebo/CMakeLists.txt 을 열고 다음 라인을 파일의 맨밑에 추가합니다.
<div class="host-code"></div>
```sh
	find_package(octomap REQUIRED)
	include_directories(${OCTOMAP_INCLUDE_DIRS})
	link_libraries(${OCTOMAP_LIBRARIES})
```
~/catkin_ws/src/rotors_simulator/rotors_gazebo/package.xml 을 열고 다음 라인을 추가합니다.
<div class="host-code"></div>
```sh
	<build_depend>octomap</build_depend>
	<run_depend>octomap</run_depend>
```

다음 2개 라인을 실행합니다.
<aside class="note">
첫번째 라인은 기본 shell editor를 gedit로 변경합니다. vim에 익숙하지 않은 사용자를 위해 추천하며 만약 필요없다면 생략해도 됩니다.
</aside>

<div class="host-code"></div>
```sh
	export EDITOR='gedit'
	rosed octomap_server octomap_tracking_server.launch
```
그리고 다음 두 라인을 변경합니다.
<div class="host-code"></div>
```sh
	<param name="frame_id" type="string" value="map" />
	...
	<!--remap from="cloud_in" to="/rgbdslam/batch_clouds" /-->
```
<div class="host-code"></div>
to
```sh
	<param name="frame_id" type="string" value="world" />
	...
	<remap from="cloud_in" to="/firefly/vi_sensor/camera_depth/depth/points" />
```



## 시뮬레이션 실행하기

다음 3개 라인을 실행하는데 3개 개별 터미널에서 실행합니다. 이렇게 하면 Gazebo, Rviz, octomap server를 시작합니다.

<div class="host-code"></div>
```sh
	roslaunch rotors_gazebo mav_hovering_example_with_vi_sensor.launch  mav_name:=firefly
	rviz
	roslaunch octomap_server octomap_tracking_server.launch
```

Rviz에서 윈도우의 왼쪽상단에 있는 'Fixed Frame' 필드를 'map'을 'world'로 변경합니다. 이제 왼쪽아래에 add 버튼을 클릭하고 MarkerArray를 선택합니다. MarkerArray를 더블클릭하고 'Marker Topic'을 '/free_cells_vis_array'에서 '/occupied_cells_vis_array'로 변경합니다.

이제 바닥 일부가 보입니다.

Gazebo 창에서 붉은 로터 앞에 큐브를 넣고 Rviz에서 볼 수 있습니다.


![](images/sim/octomap.png)
