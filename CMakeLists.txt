cmake_minimum_required(VERSION 3.5)
project(unitree_ros2)

# Opcional: C y C++ por defecto
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Incluir directorios e indicar libs si fuera necesario
include_directories(include include/common include/nlohmann)
link_directories(src)

# Lista de dependencias en bloque
set(DEPENDENCY_LIST
  unitree_hg
  unitree_api
  rclcpp
  std_msgs
  rosbag2_cpp
)

# Buscar dependencias
find_package(ament_cmake REQUIRED)
find_package(unitree_hg REQUIRED)
find_package(unitree_api REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rosbag2_cpp REQUIRED)

# --------------------------------------------------------------------
# 1) Crear la librería 'unitree_ros2'
#    (Opcional: pon aquí los .cpp que quieras compartir entre ejecutables)
# --------------------------------------------------------------------
add_library(unitree_ros2 SHARED
  src/common/ros2_sport_client.cpp
  # Si tienes más ficheros .cpp que quieras en la librería, agrégalos aquí
)

ament_target_dependencies(unitree_ros2
  ${DEPENDENCY_LIST}
)

# Instalar la librería en lib/
install(TARGETS unitree_ros2
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# Exportarla para que otros puedan find_package(unitree_ros2)
ament_export_libraries(unitree_ros2)

# --------------------------------------------------------------------
# 2) Crear ejecutables y asociarlos a dependencias
# --------------------------------------------------------------------
add_executable(low_level_ctrl
  src/low_level_ctrl.cpp
  src/common/motor_crc.cpp
)
ament_target_dependencies(low_level_ctrl ${DEPENDENCY_LIST})
target_link_libraries(low_level_ctrl unitree_ros2)

add_executable(low_level_ctrl_hg
  src/h1-2/lowlevel/low_level_ctrl_hg.cpp
  src/common/motor_crc_hg.cpp
)
ament_target_dependencies(low_level_ctrl_hg ${DEPENDENCY_LIST})
target_link_libraries(low_level_ctrl_hg unitree_ros2)

add_executable(g1_low_level_example
  src/g1/lowlevel/g1_low_level_example.cpp
  src/common/motor_crc_hg.cpp
)
ament_target_dependencies(g1_low_level_example ${DEPENDENCY_LIST})
target_link_libraries(g1_low_level_example unitree_ros2)

add_executable(read_low_state
  src/read_low_state.cpp
)
ament_target_dependencies(read_low_state ${DEPENDENCY_LIST})
target_link_libraries(read_low_state unitree_ros2)

add_executable(read_low_state_hg
  src/read_low_state_hg.cpp
)
ament_target_dependencies(read_low_state_hg ${DEPENDENCY_LIST})
target_link_libraries(read_low_state_hg unitree_ros2)

add_executable(read_motion_state
  src/read_motion_state.cpp
)
ament_target_dependencies(read_motion_state ${DEPENDENCY_LIST})
target_link_libraries(read_motion_state unitree_ros2)

add_executable(read_wireless_controller
  src/read_wireless_controller.cpp
)
ament_target_dependencies(read_wireless_controller ${DEPENDENCY_LIST})
target_link_libraries(read_wireless_controller unitree_ros2)

# Descomenta si usas record_bag
# add_executable(record_bag src/record_bag.cpp)
# ament_target_dependencies(record_bag ${DEPENDENCY_LIST})
# target_link_libraries(record_bag unitree_ros2)

add_executable(sport_mode_ctrl
  src/sport_mode_ctrl.cpp
)
ament_target_dependencies(sport_mode_ctrl ${DEPENDENCY_LIST})
target_link_libraries(sport_mode_ctrl unitree_ros2)

# --------------------------------------------------------------------
# 3) Instalar los ejecutables
#    (van a install/lib/unitree_ros2)
# --------------------------------------------------------------------
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

# --------------------------------------------------------------------
# 4) Opcional: test y cierre con ament_package
# --------------------------------------------------------------------
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
