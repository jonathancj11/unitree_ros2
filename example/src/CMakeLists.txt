cmake_minimum_required(VERSION 3.5)
project(unitree_ros2)

# Si deseas C++14:
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

# Opcional: warnings extra
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# ---------------------------------------------
# 1. Declarar dependencias de ROS2 y Unitree
# ---------------------------------------------
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rosbag2_cpp REQUIRED)
find_package(unitree_go REQUIRED)
find_package(unitree_hg REQUIRED)
find_package(unitree_api REQUIRED)

# ---------------------------------------------
# 2. Definir variable con la lista de dependencias ament
#    (Para no repetir en cada ament_target_dependencies)
# ---------------------------------------------
set(DEPENDENCY_LIST
  rclcpp
  std_msgs
  rosbag2_cpp
  unitree_go
  unitree_hg
  unitree_api
)

# ---------------------------------------------
# 3. Incluir directorios (headers) y link-directories si hace falta
# ---------------------------------------------
include_directories(include include/common include/nlohmann)
link_directories(src)

# ---------------------------------------------
# 4. Crear la librería 'unitree_ros2'
#    (elige qué .cpp van dentro de la librería)
# ---------------------------------------------
add_library(unitree_ros2 SHARED
  # Ejemplo: si quieres usar ros2_sport_client como parte de la librería
  src/common/ros2_sport_client.cpp
  # Agrega aquí más .cpp que desees compartir en la librería
)

ament_target_dependencies(unitree_ros2
  ${DEPENDENCY_LIST}
)

# Instalar la librería y exportarla
install(TARGETS unitree_ros2
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)
ament_export_libraries(unitree_ros2)


# ---------------------------------------------
# 5. Crear los ejecutables
#    (antes estaban en "unitree_ros2_example"; ahora todos en este único paquete)
# ---------------------------------------------

add_executable(low_level_ctrl src/low_level_ctrl.cpp src/common/motor_crc.cpp)
ament_target_dependencies(low_level_ctrl ${DEPENDENCY_LIST})
target_link_libraries(low_level_ctrl unitree_ros2)

add_executable(low_level_ctrl_hg src/h1-2/lowlevel/low_level_ctrl_hg.cpp src/common/motor_crc_hg.cpp)
ament_target_dependencies(low_level_ctrl_hg ${DEPENDENCY_LIST})
target_link_libraries(low_level_ctrl_hg unitree_ros2)

add_executable(g1_low_level_example src/g1/lowlevel/g1_low_level_example.cpp src/common/motor_crc_hg.cpp)
ament_target_dependencies(g1_low_level_example ${DEPENDENCY_LIST})
target_link_libraries(g1_low_level_example unitree_ros2)

add_executable(read_low_state src/read_low_state.cpp)
ament_target_dependencies(read_low_state ${DEPENDENCY_LIST})
target_link_libraries(read_low_state unitree_ros2)

add_executable(read_low_state_hg src/read_low_state_hg.cpp)
ament_target_dependencies(read_low_state_hg ${DEPENDENCY_LIST})
target_link_libraries(read_low_state_hg unitree_ros2)

add_executable(read_motion_state src/read_motion_state.cpp)
ament_target_dependencies(read_motion_state ${DEPENDENCY_LIST})
target_link_libraries(read_motion_state unitree_ros2)

add_executable(read_wireless_controller src/read_wireless_controller.cpp)
ament_target_dependencies(read_wireless_controller ${DEPENDENCY_LIST})
target_link_libraries(read_wireless_controller unitree_ros2)

# Si deseas usar record_bag (descomentarlo si existe):
# add_executable(record_bag src/record_bag.cpp)
# ament_target_dependencies(record_bag ${DEPENDENCY_LIST})
# target_link_libraries(record_bag unitree_ros2)

add_executable(sport_mode_ctrl src/sport_mode_ctrl.cpp src/common/ros2_sport_client.cpp)
ament_target_dependencies(sport_mode_ctrl ${DEPENDENCY_LIST})
target_link_libraries(sport_mode_ctrl unitree_ros2)


# ---------------------------------------------
# 6. Instalar los binarios
#    (van a ir a install/lib/unitree_ros2)
# ---------------------------------------------
install(TARGETS
  low_level_ctrl
  low_level_ctrl_hg
  g1_low_level_example
  read_low_state
  read_low_state_hg
  read_motion_state
  read_wireless_controller
  sport_mode_ctrl
  # record_bag
  DESTINATION lib/${PROJECT_NAME}
)


# ---------------------------------------------
# 7. Finalizar
# ---------------------------------------------
ament_package()
