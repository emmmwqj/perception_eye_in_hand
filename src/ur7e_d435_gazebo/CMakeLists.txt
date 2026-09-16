cmake_minimum_required(VERSION 3.16)
project(ur7e_d435_gazebo)
find_package(ament_cmake REQUIRED)
find_package(gazebo REQUIRED)
find_package(gazebo_ros REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(image_transport REQUIRED)
find_package(point_cloud_transport REQUIRED)
find_package(camera_info_manager REQUIRED)

# Build the provided plugin locally; no dependency on another workspace's .so.
add_library(realsense_gazebo_plugin SHARED
  third_party/realsense_gazebo_plugin/src/RealSensePlugin.cpp
  third_party/realsense_gazebo_plugin/src/gazebo_ros_realsense.cpp)
target_compile_features(realsense_gazebo_plugin PRIVATE cxx_std_17)
target_include_directories(realsense_gazebo_plugin PRIVATE
  third_party/realsense_gazebo_plugin/include ${GAZEBO_INCLUDE_DIRS})
target_link_directories(realsense_gazebo_plugin PRIVATE ${GAZEBO_LIBRARY_DIRS})
ament_target_dependencies(realsense_gazebo_plugin
  gazebo_ros rclcpp sensor_msgs image_transport point_cloud_transport camera_info_manager)
target_link_libraries(realsense_gazebo_plugin ${GAZEBO_LIBRARIES})
install(TARGETS realsense_gazebo_plugin LIBRARY DESTINATION lib)
install(DIRECTORY config urdf meshes launch licenses DESTINATION share/${PROJECT_NAME})
install(FILES README.md THIRD_PARTY.md DESTINATION share/${PROJECT_NAME})
install(PROGRAMS scripts/spawn_scene.py scripts/verify_scene.py DESTINATION lib/${PROJECT_NAME})
install(FILES scripts/gazebo_obstacle_utils.py DESTINATION lib/${PROJECT_NAME})
ament_package()
