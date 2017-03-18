# MAVROS offboard 제어 예제

<aside class="caution">
Offboard 제어는 위험합니다. 실제 비행체에서 동작시키는 경우라면 문제가 발생했을때 수동 제어로 돌릴 수 있는 방법을 가지고 있어야 합니다.
</aside>

다음 튜토리얼은 mavros를 통해 offboard 제어의 기본적은 내용들을 Gazebo에서 시뮬레이트한 Iris 쿼드콥터에 적용합니다. 튜토리얼의 마지막에 아래 비디오에서와 같이 동일하게 동작하는 것을 볼 수 있습니다.(2m 고도까지 천천히 이륙)

<video width="100%" autoplay="true" controls="true">
	<source src="images/sim/gazebo_offboard.webm" type="video/webm">
</video>

## Code
ros 패키지에서 offb_node.cpp 파일을 생성하고 다음 내용을 붙여넣기 합니다. :
```C++
/**
 * @file offb_node.cpp
 * @brief offboard example node, written with mavros version 0.14.2, px4 flight
 * stack and tested in Gazebo SITL
 */

#include <ros/ros.h>
#include <geometry_msgs/PoseStamped.h>
#include <mavros_msgs/CommandBool.h>
#include <mavros_msgs/SetMode.h>
#include <mavros_msgs/State.h>

mavros_msgs::State current_state;
void state_cb(const mavros_msgs::State::ConstPtr& msg){
    current_state = *msg;
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "offb_node");
    ros::NodeHandle nh;

    ros::Subscriber state_sub = nh.subscribe<mavros_msgs::State>
            ("mavros/state", 10, state_cb);
    ros::Publisher local_pos_pub = nh.advertise<geometry_msgs::PoseStamped>
            ("mavros/setpoint_position/local", 10);
    ros::ServiceClient arming_client = nh.serviceClient<mavros_msgs::CommandBool>
            ("mavros/cmd/arming");
    ros::ServiceClient set_mode_client = nh.serviceClient<mavros_msgs::SetMode>
            ("mavros/set_mode");

    //the setpoint publishing rate MUST be faster than 2Hz
    ros::Rate rate(20.0);

    // wait for FCU connection
    while(ros::ok() && current_state.connected){
        ros::spinOnce();
        rate.sleep();
    }

    geometry_msgs::PoseStamped pose;
    pose.pose.position.x = 0;
    pose.pose.position.y = 0;
    pose.pose.position.z = 2;

    //send a few setpoints before starting
    for(int i = 100; ros::ok() && i > 0; --i){
        local_pos_pub.publish(pose);
        ros::spinOnce();
        rate.sleep();
    }

    mavros_msgs::SetMode offb_set_mode;
    offb_set_mode.request.custom_mode = "OFFBOARD";

    mavros_msgs::CommandBool arm_cmd;
    arm_cmd.request.value = true;

    ros::Time last_request = ros::Time::now();

    while(ros::ok()){
        if( current_state.mode != "OFFBOARD" &&
            (ros::Time::now() - last_request > ros::Duration(5.0))){
            if( set_mode_client.call(offb_set_mode) &&
                offb_set_mode.response.success){
                ROS_INFO("Offboard enabled");
            }
            last_request = ros::Time::now();
        } else {
            if( !current_state.armed &&
                (ros::Time::now() - last_request > ros::Duration(5.0))){
                if( arming_client.call(arm_cmd) &&
                    arm_cmd.response.success){
                    ROS_INFO("Vehicle armed");
                }
                last_request = ros::Time::now();
            }
        }

        local_pos_pub.publish(pose);

        ros::spinOnce();
        rate.sleep();
    }

    return 0;
}

```
## Code explanation
```C++
#include <ros/ros.h>
#include <geometry_msgs/PoseStamped.h>
#include <mavros_msgs/CommandBool.h>
#include <mavros_msgs/SetMode.h>
#include <mavros_msgs/State.h>
```
The `mavros_msgs` package contains all of the custom messages required to operate services and topics provided by the mavros package. All services and topics as well as their corresponding message types are documented in the [mavros wiki](http://wiki.ros.org/mavros).

```C++
mavros_msgs::State current_state;
void state_cb(const mavros_msgs::State::ConstPtr& msg){
    current_state = *msg;
}
```
We create a simple callback which will save the current state of the autopilot. This will allow us to check connection, arming and offboard flags.

```C++
ros::Subscriber state_sub = nh.subscribe<mavros_msgs::State>("mavros/state", 10, state_cb);
ros::Publisher local_pos_pub = nh.advertise<geometry_msgs::PoseStamped>("mavros/setpoint_position/local", 10);
ros::ServiceClient arming_client = nh.serviceClient<mavros_msgs::CommandBool>("mavros/cmd/arming");
ros::ServiceClient set_mode_client = nh.serviceClient<mavros_msgs::SetMode>("mavros/set_mode");
```
We instantiate a publisher to publish the commanded local position and the appropriate clients to request arming and mode change. Note that for your own system, the "mavros" prefix might be different as it will depend on the name given to the node in it's launch file.
```C++
//the setpoint publishing rate MUST be faster than 2Hz
ros::Rate rate(20.0);
```
px4 flight stack은 2개 offboard 명령사이에 500ms 타임아웃이 있습니다. 이 타임아웃 시간을 초과하게 되면, commander는 offboard 모드로 들어오기 전의 모드로 돌아가게 됩니다. 이것이 publish rate를 반드시 2Hz보다 빠르게 해야하는 이유입니다. 이때 발생할 수 있는 지원시간도 고려해야 합니다. POSCTL 모드에서 offboard 모드로 들어가야하는 이유도 offboard 모드에서 빠져나오게 되는 경우 비행체가 갑자기 움직이게 되는 것을 방지할 수 있습니다.

```C++
// wait for FCU connection
while(ros::ok() && current_state.connected){
    ros::spinOnce();
    rate.sleep();
}
```
publish하기 전에 mavros와 autopilot이 연결되기를 기다려야 합니다. 이 loop은 hartbeat 메시지를 받자마자 빠져나오게 됩니다.
```C++
geometry_msgs::PoseStamped pose;
pose.pose.position.x = 0;
pose.pose.position.y = 0;
pose.pose.position.z = 2;
```
px4 flight stack이 aerospace NED 좌표 프레임에서 동작하지만 mavros는 이 좌표를 표준 ENU 프레임으로 변환하며 역반환도 일어납니다. 이것이 z에 양수 2를 설정하는 이유입니다.
```C++
//send a few setpoints before starting
for(int i = 100; ros::ok() && i > 0; --i){
    local_pos_pub.publish(pose);
    ros::spinOnce();
    rate.sleep();
}
```
offboard 모드에 진입하기 전에, 이미 streaming setpoint을 구동시켜놨어야 하며 그렇지 않은 경우 모드 스위치는 동작은 거부됩니다. 여기서 100은 임의의 숫자를 선택한 것입니다.
```C++
mavros_msgs::SetMode offb_set_mode;
offb_set_mode.request.custom_mode = "OFFBOARD";
```
`OFFBOARD`로 custom 모드를 설정합니다. [지원하는 modes](http://wiki.ros.org/mavros/CustomModes#PX4_native_flight_stack)의 목록을 참고하세요.
```C++
mavros_msgs::CommandBool arm_cmd;
arm_cmd.request.value = true;

ros::Time last_request = ros::Time::now();

while(ros::ok()){
		if( current_state.mode != "OFFBOARD" &&
				(ros::Time::now() - last_request > ros::Duration(5.0))){
				if( set_mode_client.call(offb_set_mode) &&
						offb_set_mode.response.success){
						ROS_INFO("Offboard enabled");
				}
				last_request = ros::Time::now();
		} else {
				if( !current_state.armed &&
						(ros::Time::now() - last_request > ros::Duration(5.0))){
						if( arming_client.call(arm_cmd) &&
								arm_cmd.response.success){
								ROS_INFO("Vehicle armed");
						}
						last_request = ros::Time::now();
				}
		}

		local_pos_pub.publish(pose);

		ros::spinOnce();
		rate.sleep();
}
```
코드의 나머지 부분은 이해하기 쉽습니다. 비행을 하기 위해서 쿼드에 arm을 하고나서, offboard mode로 변환합니다. autopilot에 request를 몰리지 않도록 하기 위해서 5초마다 서비스 호출을 비우게 됩니다. 동일한 loop에서 계속해서 적절한 rate로 pose 요청을 보내게 됩니다.

<aside class="tip">
여기서 보는 코드는 예제로 보여주는 목적으로 단순화 시켰습니다. 규모가 큰 시스템에서 주기적으로 setpoint을 publish하는 역할을 수행하는 thread를 생성하는 것이 유용합니다.
</aside>
