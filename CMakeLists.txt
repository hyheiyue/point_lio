cmake_minimum_required(VERSION 3.8)
project(point_lio)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fexceptions")
set(CMAKE_CXX_FLAGS "-std=c++17 -O3 -pthread -fexceptions")
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

message(STATUS "ROOT_DIR set to: ${CMAKE_CURRENT_SOURCE_DIR}/")

# Find packages
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(pcl_ros REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(livox_ros_driver2 REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(glog REQUIRED)
find_package(OpenMP QUIET)
find_package(PythonLibs REQUIRED)

if(OPENMP_FOUND)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
endif()

include_directories(
  include
  ${EIGEN3_INCLUDE_DIR}
  ${PCL_INCLUDE_DIRS}
  ${PYTHON_INCLUDE_DIRS}
)

link_directories(
  include
  ${PCL_LIBRARY_DIRS}
)

# Declare a ROS2 executable
add_executable(pointlio_mapping
  src/laserMapping.cpp
  src/li_initialization.cpp
  src/parameters.cpp
  src/preprocess.cpp
  src/Estimator.cpp
  src/IMU_Processing.cpp
)

target_compile_definitions(pointlio_mapping PRIVATE ROOT_DIR="${CMAKE_CURRENT_SOURCE_DIR}/")

ament_target_dependencies(pointlio_mapping
  rclcpp
  rclpy
  geometry_msgs
  nav_msgs
  sensor_msgs
  visualization_msgs
  pcl_ros
  pcl_conversions
  tf2_ros
  livox_ros_driver2
)

target_link_libraries(pointlio_mapping ${PCL_LIBRARIES} ${PYTHON_LIBRARIES})
target_include_directories(pointlio_mapping PRIVATE ${PYTHON_INCLUDE_DIRS})

# Install the executable
install(TARGETS
  pointlio_mapping
  DESTINATION lib/${PROJECT_NAME}
)

install(
  DIRECTORY config launch rviz_cfg
  DESTINATION share/${PROJECT_NAME}
)

# Export dependencies
ament_export_dependencies(
  rclcpp
  rclpy
  geometry_msgs
  nav_msgs
  sensor_msgs
  pcl_ros
  pcl_conversions
  tf2_ros
  livox_ros_driver2
  Eigen3
  PCL
)

ament_package()

