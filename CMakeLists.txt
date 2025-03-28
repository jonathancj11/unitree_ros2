cmake_minimum_required(VERSION 3.5)
project(unitree_ros2)

# Configurar políticas
cmake_policy(SET CMP0015 NEW)
cmake_policy(SET CMP0081 NEW)

message(STATUS "DEBUG: Proyecto unitree_ros2 iniciado")
message(STATUS "DEBUG: CMAKE_CURRENT_SOURCE_DIR = ${CMAKE_CURRENT_SOURCE_DIR}")
message(STATUS "DEBUG: CMAKE_CURRENT_BINARY_DIR = ${CMAKE_CURRENT_BINARY_DIR}")

# Incluir directorios
include_directories(include include/common include/nlohmann)
link_directories(src)
include_directories(/opt/unitree_robotics/include)
link_directories(/opt/unitree_robotics/lib)
message(STATUS "DEBUG: Directorios de inclusión agregados: ${CMAKE_CURRENT_SOURCE_DIR}/include y /opt/unitree_robotics/include")

# Configuración de estándares de C y C++
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
  message(STATUS "DEBUG: CMAKE_C_STANDARD establecido a 99")
endif()
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
  message(STATUS "DEBUG: CMAKE_CXX_STANDARD establecido a 14")
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
  message(STATUS "DEBUG: Opciones de compilación añadidas: -Wall -Wextra -Wpedantic")
endif()

# Buscar dependencias
find_package(ament_cmake REQUIRED)
find_package(unitree_hg REQUIRED)
find_package(unitree_api REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rosbag2_cpp REQUIRED)
message(STATUS "DEBUG: Dependencias encontradas: ament_cmake, unitree_hg, unitree_api, rclcpp, std_msgs, rosbag2_cpp")

# Lista de dependencias
set(DEPENDENCY_LIST unitree_hg unitree_api rclcpp std_msgs rosbag2_cpp)
message(STATUS "DEBUG: DEPENDENCY_LIST = ${DEPENDENCY_LIST}")

# Buscar la librería unitree_sdk2 en /opt/unitree_robotics/lib
find_library(UNITREE_SDK2_LIB
  NAMES unitree_sdk2
  PATHS /opt/unitree_robotics/lib
  NO_DEFAULT_PATH
  message(STATUS "DEBUG: Buscando unitree_sdk2 en /opt/unitree_robotics/lib...")
)
if(UNITREE_SDK2_LIB)
  message(STATUS "DEBUG: Se encontró unitree_sdk2 en: ${UNITREE_SDK2_LIB}")
else()
  message(FATAL_ERROR "DEBUG: No se encontró unitree_sdk2 en /opt/unitree_robotics/lib")
endif()

# --------------------------------------------------------------------
# 1) Crear la librería unitree_ros2
# --------------------------------------------------------------------
message(STATUS "DEBUG: Creando la librería unitree_ros2")
add_library(unitree_ros2 SHARED
  src/common/ros2_sport_client.cpp
)
ament_target_dependencies(unitree_ros2 ${DEPENDENCY_LIST})
target_link_libraries(unitree_ros2 ${UNITREE_SDK2_LIB})

# Definir y verificar dinámicamente la ruta de los headers de unitree_go
set(UNITREE_GO_INSTALL_DIR "/opt/unitree_ws/install/unitree_go")
if(EXISTS "${UNITREE_GO_INSTALL_DIR}/include")
  message(STATUS "DEBUG: Directorio de headers de unitree_go verificado: ${UNITREE_GO_INSTALL_DIR}/include")
else()
  message(WARNING "DEBUG: Directorio de headers de unitree_go NO existe: ${UNITREE_GO_INSTALL_DIR}/include")
endif()
target_include_directories(unitree_ros2 PUBLIC "${UNITREE_GO_INSTALL_DIR}/include")

# Instalar la librería
install(TARGETS unitree_ros2
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)
message(STATUS "DEBUG: Librería unitree_ros2 instalada en lib/")

ament_export_libraries(unitree_ros2)

# --------------------------------------------------------------------
# 2) Crear ejecutables y asociarlos a dependencias
# --------------------------------------------------------------------
message(STATUS "DEBUG: Creando ejecutables...")

add_executable(low_level_ctrl
  src/low_level_ctrl.cpp
  src/common/motor_crc.cpp
)
ament_target_dependencies(low_level_ctrl ${DEPENDENCY_LIST})
target_link_libraries(low_level_ctrl unitree_ros2)
message(STATUS "DEBUG: Ejecutable low_level_ctrl configurado.")

add_executable(low_level_ctrl_hg
  src/h1-2/lowlevel/low_level_ctrl_hg.cpp
  src/common/motor_crc_hg.cpp
)
ament_target_dependencies(low_level_ctrl_hg ${DEPENDENCY_LIST})
target_link_libraries(low_level_ctrl_hg unitree_ros2)
message(STATUS "DEBUG: Ejecutable low_level_ctrl_hg configurado.")

add_executable(g1_low_level_example
  src/g1/lowlevel/g1_low_level_example.cpp
  src/common/motor_crc_hg.cpp
)
ament_target_dependencies(g1_low_level_example ${DEPENDENCY_LIST})
target_link_libraries(g1_low_level_example unitree_ros2)
message(STATUS "DEBUG: Ejecutable g1_low_level_example configurado.")

add_executable(read_low_state src/read_low_state.cpp)
ament_target_dependencies(read_low_state ${DEPENDENCY_LIST})
target_link_libraries(read_low_state unitree_ros2)
message(STATUS "DEBUG: Ejecutable read_low_state configurado.")

add_executable(read_low_state_hg src/read_low_state_hg.cpp)
ament_target_dependencies(read_low_state_hg ${DEPENDENCY_LIST})
target_link_libraries(read_low_state_hg unitree_ros2)
message(STATUS "DEBUG: Ejecutable read_low_state_hg configurado.")

add_executable(read_motion_state src/read_motion_state.cpp)
ament_target_dependencies(read_motion_state ${DEPENDENCY_LIST})
target_link_libraries(read_motion_state unitree_ros2)
message(STATUS "DEBUG: Ejecutable read_motion_state configurado.")

add_executable(read_wireless_controller src/read_wireless_controller.cpp)
ament_target_dependencies(read_wireless_controller ${DEPENDENCY_LIST})
target_link_libraries(read_wireless_controller unitree_ros2)
message(STATUS "DEBUG: Ejecutable read_wireless_controller configurado.")

add_executable(sport_mode_ctrl src/sport_mode_ctrl.cpp)
ament_target_dependencies(sport_mode_ctrl ${DEPENDENCY_LIST})
target_link_libraries(sport_mode_ctrl unitree_ros2)
message(STATUS "DEBUG: Ejecutable sport_mode_ctrl configurado.")

# --------------------------------------------------------------------
# 3) Instalar los ejecutables
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
  DESTINATION lib/${PROJECT_NAME}
)
message(STATUS "DEBUG: Ejecutables instalados en lib/${PROJECT_NAME}")

# --------------------------------------------------------------------
# 4) Opcional: test y cierre con ament_package
# --------------------------------------------------------------------
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  message(STATUS "DEBUG: Dependencias de testing configuradas.")
endif()

ament_package()
message(STATUS "DEBUG: Configuración de unitree_ros2 completada exitosamente.")
