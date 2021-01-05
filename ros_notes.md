#### 1.创建ros工程

```shell
mkdir catkin_ws # 工作空间名称随意
cd catkin_ws
mkdir src
#初始化工作空间
catkin_init_workspace # 这一步在src目录下会出现CMakeLists.txt文件

#返回工作空间根目录
cd..
catkin_make #会出现 build，devel 两个文件夹
catkin_make install # 会出现 install 文件

```

#### 2.创建功能包

```shell
# 进入根目录的src文件中
cd ..
cd src
# 创建功能包，test_pkg:功能包名称，{roscpp, rospy,stdmsgs}:分别是c++依赖，python依赖，头文件库
# 执行完成后根目录src下会新建一个test_pkg文件夹
# test_pkg文件夹下会分别有include，src，CMakeLists.txt, package.xml 四个文件，后两个文件是必须要有的
catkin_create_pkg test_pkg roscpp rospy std_msgs 
# 返回工作空间根目录下
cd..
# 编译工作空间
catkin_make # 此时工作空间虽然为空但是依旧可以编译
# 设置工作空间的环境变量
source devel/setup.bash # 在工作空间的根目录下执行(如果执行这条指令，关闭了终端，设置的环境变量就会失效，只是临时的设置，如果需要一直生效需要在 .bachrc文件 中加入 source/home/user/catkin_ws1/devel/setup.bash   并且需要重新启动终端才能生效)
# 查看工作空间的环境变量
echo $ROS_PACKAGE_PATH
```

#### 3.消息发布

```shell
# 依照前面1，2创建工程和删除第二步创建的功能包
# 创建消息发布的功能包,执行指令后，根目录下的src文件中会有一个learning_topic的文件夹
# learin_topic文件夹下会有include，src，CMakeList.txt，package.xml四个文件
# 在package.xml文件中会看到依赖的引入
catkin_create_pkg learning_topic roscpp std_msgs geometry_msgs turtlesim rospy
# 在创建的功能包中的src中创建编写c++代码代码如下命名为 velocity_publisher.cpp：

```

```c++
/**
 * 该例程将发布turtle1/cmd_vel话题，消息类型geometry_msgs::Twist
 */
 
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>

int main(int argc, char **argv)
{
	// ROS节点初始化
	ros::init(argc, argv, "velocity_publisher");

	// 创建节点句柄
	ros::NodeHandle n;

	// 创建一个Publisher，发布名为/turtle1/cmd_vel的topic，消息类型为geometry_msgs::Twist，队列长度10
	ros::Publisher turtle_vel_pub = n.advertise<geometry_msgs::Twist>("/turtle1/cmd_vel", 10);

	// 设置循环的频率
	ros::Rate loop_rate(10);

	int count = 0;
	while (ros::ok())
	{
	    // 初始化geometry_msgs::Twist类型的消息
		geometry_msgs::Twist vel_msg;
		vel_msg.linear.x = 0.5;
		vel_msg.angular.z = 0.2;

	    // 发布消息
		turtle_vel_pub.publish(vel_msg);
		ROS_INFO("Publsh turtle velocity command[%0.2f m/s, %0.2f rad/s]", 
				vel_msg.linear.x, vel_msg.angular.z);

	    // 按照循环频率延时
	    loop_rate.sleep();
	}

	return 0;
}
```

##### 如何实现一个发布者

* 初始化ROS节点；
* 向ROS Master 注册节点信息，包括发布话题名和话题中的消息类型；
* 创建消息数据；
* 按照一定的频率循环发布消息

###### c++版

```shell
# 配置发布者代码编译规则（c++）
# 在功能包（此处为learin_topic)文件夹中的CMakeList.txt中****install***的上方加入下面两行
add_executable(velocity_publisher src/velocity_publisher.cpp) # 第一个参数是生成的可执行文件的名称， 第二个参数是需要编译的文件的路径
target_link_libraries(velocity_publisher ${catkin_LIBRARIES}) # 设置链接库

# 回到工作空间根目录下编译
#执行完成后devel/lib/learing_topic/目录下会有一个velocity_publisher文件生成
catkin_make
# 设置环境变量
source devel/setup.bash # 在工作空间的根目录下执行(如果执行这条指令，关闭了终端，设置的环境变量就会失效，只是临时的设置，如果需要一直生效需要在 .bachrc文件 中加入 source/home/user/catkin_ws1/devel/setup.bash    并且需要重新启动终端才能生效)

# 让乌龟跑起来
roscore
rosrun turtlesim turtlesim_node
rosrun learning_topic velocity_publisher
```

###### python版

python版不需要在 CMakeList.txt 进行配置只需要在前面的基础上在 learning_topic功能包中新建一个scripts文件夹并且在创建编写对应的python代码（velocity_publisher.py）代码如下：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# 该例程将发布turtle1/cmd_vel话题，消息类型geometry_msgs::Twist

'''
##### 如何实现一个发布者:
* 初始化ROS节点；
* 向ROS Master 注册节点信息，包括发布话题名和话题中的消息类型；
* 创建消息数据；
* 按照一定的频率循环发布消息
'''
import rospy
from geometry_msgs.msg import Twist

def velocity_publisher():
	# ROS节点初始化
    rospy.init_node('velocity_publisher', anonymous=True)

	# 创建一个Publisher，发布名为/turtle1/cmd_vel的topic，消息类型为geometry_msgs::Twist，队列长度10
    turtle_vel_pub = rospy.Publisher('/turtle1/cmd_vel', Twist, queue_size=10)

	#设置循环的频率
    rate = rospy.Rate(10) 

    while not rospy.is_shutdown():
		# 初始化geometry_msgs::Twist类型的消息
        vel_msg = Twist()
        vel_msg.linear.x = 0.5
        vel_msg.angular.z = 0.2

		# 发布消息
        turtle_vel_pub.publish(vel_msg)
    	rospy.loginfo("Publsh turtle velocity command[%0.2f m/s, %0.2f rad/s]", 
				vel_msg.linear.x, vel_msg.angular.z)

		# 按照循环频率延时
        rate.sleep()

if __name__ == '__main__':
    try:
        velocity_publisher()
    except rospy.ROSInterruptException:
        pass
```

== **注意：在编写完代码后一定要将文件设置为可执行文件** ==

执行以下命令乌龟就可以跑起来了！

```shell
roscore
rosrun turtlesim turtlesim_node
rosrun learning_topic velocity_publisher.py
```

#### 4.订阅消息

##### 如何实现一个订阅者

* 初始化ROS节点
* 订阅需要的话题
* 循环等待话题消息，接收到消息后进入回调函数
* 回调函数中完成消息处理 

###### python版本

```python
# pose_subscriber.py

#!/usr/bin/env python
# -*- coding: utf-8 -*-


# 该例程将订阅/turtle1/pose话题，消息类型turtlesim::Pose

import rospy
from turtlesim.msg import Pose

def poseCallback(msg):
    rospy.loginfo("Turtle pose: x:%0.6f, y:%0.6f", msg.x, msg.y)

def pose_subscriber():
	# ROS节点初始化
    rospy.init_node('pose_subscriber', anonymous=True)

	# 创建一个Subscriber，订阅名为/turtle1/pose的topic，注册回调函数poseCallback
    rospy.Subscriber("/turtle1/pose", Pose, poseCallback)

	# 循环等待回调函数
    rospy.spin()

if __name__ == '__main__':
    pose_subscriber()


```

#### 5.如何实现自定义消息订阅

##### python版

1.新建一个工程（catkin_ws)

2.在src下新建一个功能包（learning_topic）

3.在learning_topic下的src下新建msg文夹

```shell
# 进入msg文件夹下，创建一个Person.msg文件
touch Person.msg

# Person.msg 内容如下
string name
uint8 sex
uint8 age

uint8 unknown = 0
uint8 male = 1
uint8 female = 2
```

4.在第二步中新建的功能包中的package.xml文件中加入以下代码：

```5.xml
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```

5.在对应功能包下的CMakelists.txt文件下  

在COMPONENTS的列表里增加message_generation，就像这样： 

```txt
# Do not just add this line to your CMakeLists.txt, modify the existing line
find_package(catkin REQUIRED COMPONENTS roscpp rospy std_msgs message_generation) 
```

6.确保设置了运行依赖

```
catkin_package(
  ...
  CATKIN_DEPENDS message_runtime ...
  ...)
```

7.找到如下代码块：

```
# add_message_files(
#   FILES
#   Message1.msg
#   Message2.msg
# )

#修改为
add_message_files(
  FILES
  Person.msg
)

#在此方法的下方加上
generate_messages(DEPENDENCIES std_msgs)
```

8.进入新建的功能包（learning_topic）

```shell
mkdir sripts
cd sripts
touch person_publisher.py # 消息发布代码
touch person_subscriber.py # 消息订阅代码

#设置成为可执行文件
chmod +x person_publisher.py
chmod +x person_subscriber.py
```

9.对应的代码如下

**person_publisher.py**

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# 该例程将发布/person_info话题，自定义消息类型learning_topic::Person

import rospy
from learning_topic.msg import Person

def velocity_publisher():
	# ROS节点初始化
    rospy.init_node('person_publisher', anonymous=True)

	# 创建一个Publisher，发布名为/person_info的topic，消息类型为learning_topic::Person，队列长度10
    person_info_pub = rospy.Publisher('/person_info', Person, queue_size=10)

	#设置循环的频率
    rate = rospy.Rate(10) 

    while not rospy.is_shutdown():
		# 初始化learning_topic::Person类型的消息
    	person_msg = Person()
    	person_msg.name = "Tom";
    	person_msg.age  = 18;
    	person_msg.sex  = Person.male;

		# 发布消息
        person_info_pub.publish(person_msg)
    	rospy.loginfo("Publsh person message[%s, %d, %d]", 
				person_msg.name, person_msg.age, person_msg.sex)

		# 按照循环频率延时
        rate.sleep()

if __name__ == '__main__':
    try:
        velocity_publisher()
    except rospy.ROSInterruptException:
        pass

```

**person_subscriber.py**

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# 该例程将订阅/person_info话题，自定义消息类型learning_topic::Person

import rospy
from learning_topic.msg import Person

def personInfoCallback(msg):
    rospy.loginfo("Subcribe Person Info: name:%s  age:%d  sex:%d", 
			 msg.name, msg.age, msg.sex)

def person_subscriber():
	# ROS节点初始化
    rospy.init_node('person_subscriber', anonymous=True)

	# 创建一个Subscriber，订阅名为/person_info的topic，注册回调函数personInfoCallback
    rospy.Subscriber("/person_info", Person, personInfoCallback)

	# 循环等待回调函数
    rospy.spin()

if __name__ == '__main__':
    person_subscriber()

```

10.进入工作空间根目录

```
roscore
source devel/setup.bash
rosrun learning_topic person_publisher.py
rosrun learning_topic person_subscriber.py
```

