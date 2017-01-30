# 통합 테스팅(Integration Testing)

양끝단 통합 테스팅에 대해서 설명합니다. 테스트는 자동으로 실행됩니다. ([Jenkins CI](advanced-jenkins-ci.md))

## ROS / MAVROS Tests

전제 조건:

  * [SITL 시뮬레이션](simulation-sitl.md)
  * [Gazebo](simulation-gazebo.md)
  * [ROS와 MAVROS](simulation-ros-interface.md)

### 테스트 실행

전체 MAVROS 테스트 세트 실행 :

```sh
cd <Firmware_clone>
source integrationtests/setup_gazebo_ros.bash $(pwd)
rostest px4 mavros_posix_tests_iris.launch
```

혹은 GUI로 실행하면 어떻게 실행되고 있는지 확인 가능 :

```sh
rostest px4 mavros_posix_tests_iris.launch gui:=true headless:=false
```

### 새로운 MAVROS 테스트 작성 (Python)

<aside class="note">
현재는 초기 단계로, 향후에는 더 많은 streamline을 지원할 예정입니다.
</aside>

####1.) 새로운 테스트 스크립트 생성

테스트 스크립트는 `integrationtests/python_src/px4_it/mavros/`에 있습니다. 기존의 다른 스크립트들을 참고합니다. [unittest](http://wiki.ros.org/unittest)를 사용법은 공식 ROS 문서를 참고합니다.


빈 테스트 뼈대:

```python
#!/usr/bin/env python
# [... LICENSE ...]

#
# @author Example Author <author@example.com>
#
PKG = 'px4'

import unittest
import rospy
import rosbag

from sensor_msgs.msg import NavSatFix

class MavrosNewTest(unittest.TestCase):
    """
    Test description
    """

    def setUp(self):
        rospy.init_node('test_node', anonymous=True)
        rospy.wait_for_service('mavros/cmd/arming', 30)

        rospy.Subscriber("mavros/global_position/global", NavSatFix, self.global_position_callback)
        self.rate = rospy.Rate(10) # 10hz
        self.has_global_pos = False

    def tearDown(self):
        pass

    #
    # General callback functions used in tests
    #
    def global_position_callback(self, data):
        self.has_global_pos = True

    def test_method(self):
        """Test method description"""

        # FIXME: hack to wait for simulation to be ready
        while not self.has_global_pos:
            self.rate.sleep()

        # TODO: execute test

if __name__ == '__main__':
    import rostest
    rostest.rosrun(PKG, 'mavros_new_test', MavrosNewTest)
```

####2.) 새로 추가한 테스트만 실행하기

```sh
# Start simulation
cd <Firmware_clone>
source integrationtests/setup_gazebo_ros.bash $(pwd)
roslaunch px4 mavros_posix_sitl.launch

# Run test (in a new shell):
cd <Firmware_clone>
source integrationtests/setup_gazebo_ros.bash $(pwd)
rosrun px4 mavros_new_test.py
```

####3.) 새로운 테스트 노드를 launch 파일에 추가하기

`launch/mavros_posix_tests_irisl.launch`에 테스트 그룹내부에 새로운 entry 추가:

```xml
	<group ns="$(arg ns)">
		[...]
        <test test-name="mavros_new_test" pkg="px4" type="mavros_new_test.py" />
    </group>
```

위에서 설명한 바와 같이 전체 테스트 세트를 실행합니다.
