cmake_minimum_required(VERSION 3.5)
project(unitree_go)

# --- Configuración de políticas ---
cmake_policy(SET CMP0015 NEW)
cmake_policy(SET CMP0081 NEW)
message(STATUS "DEBUG: Proyecto ${PROJECT_NAME} iniciado")
message(STATUS "DEBUG: CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}")
message(STATUS "DEBUG: CMAKE_CURRENT_BINARY_DIR: ${CMAKE_CURRENT_BINARY_DIR}")

# --- Configuración de estándares de C y C++ ---
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

# --- Buscar dependencias ---
message(STATUS "DEBUG: Buscando dependencias...")
find_package(ament_cmake REQUIRED)
message(STATUS "DEBUG: ament_cmake encontrada")
find_package(geometry_msgs REQUIRED)
message(STATUS "DEBUG: geometry_msgs encontrada")
find_package(rosidl_default_generators REQUIRED)
message(STATUS "DEBUG: rosidl_default_generators encontrada")
find_package(rosidl_generator_dds_idl REQUIRED)
message(STATUS "DEBUG: rosidl_generator_dds_idl encontrada")

# --- Generación de interfaces ROSIDL ---
message(STATUS "DEBUG: Iniciando rosidl_generate_interfaces para ${PROJECT_NAME}...")
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/AudioData.msg"
  "msg/BmsCmd.msg"
  "msg/BmsState.msg"
  "msg/Error.msg"
  "msg/Go2FrontVideoData.msg"
  "msg/HeightMap.msg"
  "msg/IMUState.msg"
  "msg/InterfaceConfig.msg"
  "msg/LidarState.msg"
  "msg/LowCmd.msg"
  "msg/LowState.msg"
  "msg/MotorCmd.msg"
  "msg/MotorCmds.msg"
  "msg/MotorState.msg"
  "msg/MotorStates.msg"
  "msg/PathPoint.msg"
  "msg/Req.msg"
  "msg/Res.msg"
  "msg/SportModeCmd.msg"
  "msg/SportModeState.msg"
  "msg/TimeSpec.msg"
  "msg/UwbState.msg"
  "msg/UwbSwitch.msg"
  "msg/WirelessController.msg"
  DEPENDENCIES geometry_msgs
)
message(STATUS "DEBUG: rosidl_generate_interfaces completada.")

rosidl_generate_dds_interfaces(
  ${rosidl_generate_interfaces_TARGET}__dds_connext_idl
  IDL_TUPLES ${rosidl_generate_interfaces_IDL_TUPLES}
  OUTPUT_SUBFOLDERS "dds_connext"
)
message(STATUS "DEBUG: rosidl_generate_dds_interfaces completada.")

add_dependencies(${PROJECT_NAME} ${PROJECT_NAME}__dds_connext_idl)
message(STATUS "DEBUG: Dependencias agregadas para DDS Connext.")

# --- Configurar testing (opcional) ---
if(BUILD_TESTING)
  message(STATUS "DEBUG: Configurando testing...")
  find_package(ament_lint_auto REQUIRED)
  # Si es necesario, habilita los chequeos adicionales:
  # set(ament_cmake_copyright_FOUND TRUE)
  # set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
  message(STATUS "DEBUG: Testing configurado.")
endif()

# --- Finalizar paquete ament ---
ament_package()
message(STATUS "DEBUG: ament_package completado para ${PROJECT_NAME}.")

# --- Instalar los archivos generados por rosidl ---
set(ROSIDL_GEN_DIR "${CMAKE_CURRENT_BINARY_DIR}/rosidl_generator_cpp/unitree_go/msg")
message(STATUS "DEBUG: Verificando existencia de directorio generado: ${ROSIDL_GEN_DIR}")
if(EXISTS "${ROSIDL_GEN_DIR}")
  message(STATUS "DEBUG: Directorio rosidl generado encontrado.")
else()
  message(WARNING "DEBUG: Directorio rosidl generado NO encontrado. Revisa rosidl_generate_interfaces.")
endif()

install(
  DIRECTORY ${ROSIDL_GEN_DIR}/ 
  DESTINATION include/unitree_go/msg
  FILES_MATCHING PATTERN "*.hpp"
)
message(STATUS "DEBUG: Archivos de interfaces ROSIDL instalados en include/unitree_go/msg.")
