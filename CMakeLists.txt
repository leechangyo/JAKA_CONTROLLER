cmake_minimum_required(VERSION 2.8.3)
project(opencv_bridge)

set( CMAKE_CXX_COMPILER "/usr/bin/clang++" )
set( CMAKE_BUILD_TYPE "Release" )
set( CMAKE_CXX_FLAGS "-std=c++11 -O3" )

# Add catkin and required ROS packages
find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  image_transport
  roscpp
  rospy
  sensor_msgs
  std_msgs
  message_generation
  ur_msgs
  tf
  geometry_msgs
  moveit_kinematics
  #joint_state_publisher
  robot_state_publisher
  ur_description
)

# add plain cmake packages 
find_package( PCL REQUIRED )
find_package( OpenCV 2.4.8 REQUIRED )

# catkin include dirs
include_directories(
  ${catkin_INCLUDE_DIRS} 
  include
)

## Generate messages in the 'msg' folder
 add_message_files(
   FILES
   Frame.msg
   FrameJD.msg
   MsgLinemodBoundingbox.msg
   RobotMsg.msg
   Command.msg
 )

## Generate added messages and services with any dependencies listed here
generate_messages(DEPENDENCIES sensor_msgs std_msgs ur_msgs)

# Describe catkin Project
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS message_runtime cv_bridge image_transport roscpp rospy sensor_msgs std_msgs geometry_msgs
#  DEPENDS system_lib
)

#eigen
include_directories( "/usr/include/eigen3/" )

#pcl
include_directories( ${PCL_INCLUDE_DIRS} )
link_directories( ${PCL_LIBRARY_DIRS} )
add_definitions( ${PCL_DEFINITIONS} )

#openMP (which for multiple thread program design)
find_package( OpenMP REQUIRED)  
if(OPENMP_FOUND)  
  message("OPENMP FOUND")  
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")  
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")  
endif() 

# Set sourcefiles
list(APPEND SOURCEFILES
#  src/add_library_src/config.cpp
  src/add_library_src/point_cloud.cpp
  src/add_library_src/object.cpp
  src/add_library_src/frame.cpp
  src/add_library_src/ps_machinery.cpp
)

# Set PPF libs
list(APPEND PPF_LIBS
  libpose_3d.so
  libppf_helpers.so
  libppf_match_3d.so
  libt_hash_int.so
)

#set link directory
link_directories(lib)

# Create eh library which already includes lib bellow
add_library(eh_system SHARED ${SOURCEFILES})

#my own library (included in the libeh_system above)
add_library(config SHARED src/add_library_src/config.cpp)
#add_library(frame SHARED src/frame.cpp)
#add_library(object SHARED src/object.cpp)

#######-------------------------executables-----------------------------
#add pose estimator
add_executable(pose_estimator src/pose_estimation.cpp)
target_link_libraries(pose_estimator ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${PPF_LIBS} eh_system config)
add_dependencies(pose_estimator opencv_bridge_generate_message_cpp)

#add point cloud saver
add_executable(pc_saver src/pc_saver.cpp)
target_link_libraries(pc_saver ${catkin_LIBRARIES} ${PCL_LIBRARIES} config)
add_dependencies(pc_saver opencv_bridge_generate_message_cpp)

#add pc_publisher
add_executable(pc_publisher src/pc_publisher.cpp)
target_link_libraries(pc_publisher ${catkin_LIBRARIES} ${PCL_LIBRARIES} config)
add_dependencies(pc_publisher opencv_bridge_generate_message_cpp)

#add image_collector
add_executable(image_collect src/image_collect.cpp)
target_link_libraries(image_collect ${catkin_LIBRARIES} ${PCL_LIBRARIES} config)
add_dependencies(image_collect opencv_bridge_generate_message_cpp)

#add multi-thread commander
add_executable(mt_commander src/mt_commander.cpp)
target_link_libraries(mt_commander ${catkin_LIBRARIES} config)
add_dependencies(mt_commander  opencv_bridge_generate_message_cpp)

#add fcn result publisher
add_executable(pub_fcn src/pub_fcn.cpp)
target_link_libraries(pub_fcn ${catkin_LIBRARIES})
add_dependencies(pub_fcn  opencv_bridge_generate_message_cpp)


###----------------请把所有与实验演示ＤＥＭＯ无关的代码放在以下部分--------------------------------------------------------------------------------------------------



#######--------------------used to test code in src/test --------------------
#add multi-thread commander
add_executable(test_mt src/test/multi-thread.cpp)
target_link_libraries(test_mt ${catkin_LIBRARIES} config)

#add pose publisher
add_executable(pose_publisher src/test/pose_pub.cpp)
target_link_libraries(pose_publisher ${catkin_LIBRARIES} config)
add_dependencies(pose_publisher opencv_bridge_generate_message_cpp)

#add test file for xtion short range
add_executable(xtion_test src/test/xtion_test.cpp)
target_link_libraries(xtion_test ${catkin_LIBRARIES} ${PCL_LIBRARIES} config)
add_dependencies(xtion_test opencv_bridge_generate_message_cpp)

#add MoveRobot demo
add_executable(commander src/test/MoveRobot.cpp)
target_link_libraries(commander ${catkin_LIBRARIES} config)
add_dependencies(commander  opencv_bridge_generate_message_cpp)

#add publish demo
add_executable(testrobot src/test/publish.cpp)
target_link_libraries(testrobot ${catkin_LIBRARIES} config)
add_dependencies(testrobot  opencv_bridge_generate_message_cpp)

