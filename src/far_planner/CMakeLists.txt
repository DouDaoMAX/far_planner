cmake_minimum_required(VERSION 3.5)
project(far_planner)

# C++14
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_BUILD_TYPE Release)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# ================== 依赖 ==================
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(std_srvs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(PCL REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(tf2_sensor_msgs REQUIRED)
find_package(visibility_graph_msg REQUIRED)

# ================== 关键：手动添加 OMPL（因为 gcopter 没传）==================
find_package(ompl REQUIRED PATHS /usr/share/ompl/cmake NO_DEFAULT_PATH)

# ================== 包含目录 ==================
include_directories(
  include
  ${rclcpp_INCLUDE_DIRS}
  ${std_msgs_INCLUDE_DIRS}
  ${sensor_msgs_INCLUDE_DIRS}
  ${nav_msgs_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${OpenCV_INCLUDE_DIRS}
  ${OMPL_INCLUDE_DIRS}  # 关键！解决 ompl/util/Console.h 找不到
)

link_directories(
  ${PCL_LIBRARY_DIRS}
  ${OpenCV_LIBRARY_DIRS}
)

add_definitions(${PCL_DEFINITIONS})

# ================== 源文件 ==================
set(SOURCES
  src/dynamic_graph.cpp
  src/planner_visualizer.cpp
  src/utility.cpp
  src/graph_planner.cpp
  src/contour_detector.cpp
  src/contour_graph.cpp
  src/map_handler.cpp
  src/scan_handler.cpp
  src/graph_msger.cpp
  src/terrain_planner.cpp
)

# ================== 可执行文件 ==================
add_executable(${PROJECT_NAME} src/far_planner.cpp ${SOURCES})

ament_target_dependencies(${PROJECT_NAME}
  rclcpp std_msgs std_srvs sensor_msgs nav_msgs
  visualization_msgs pcl_conversions visibility_graph_msg
  tf2 tf2_ros tf2_geometry_msgs tf2_sensor_msgs
)

# ================== 链接库 ==================
target_link_libraries(${PROJECT_NAME}
  ${PCL_LIBRARIES}
  ${OpenCV_LIBS}
  ${OMPL_LIBRARIES}  # 关键！链接 OMPL 库
)

# ================== 安装 ==================
install(TARGETS ${PROJECT_NAME} DESTINATION lib/${PROJECT_NAME})
install(DIRECTORY launch rviz config data DESTINATION share/${PROJECT_NAME})

ament_package()
