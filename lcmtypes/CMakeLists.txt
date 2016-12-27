if(WIN32)
  set(CONFIG_INSTALL_DIR CMake)
else()
  set(CONFIG_INSTALL_DIR lib${LIB_SUFFIX}/${PROJECT_NAME}/cmake)
endif()

if(PYTHONINTERP_FOUND)
  set(python_args PYTHON_SOURCES python_install_sources)
endif()
if(JAVA_FOUND AND TARGET lcm-java)
  set(java_args JAVA_SOURCES java_sources)
endif()

lcm_wrap_types(
  C_EXPORT bot_core_lcmtypes
  C_SOURCES c_sources
  C_HEADERS c_install_headers
  CPP_HEADERS cpp_install_headers
  CREATE_C_AGGREGATE_HEADER
  CREATE_CPP_AGGREGATE_HEADER
  ${python_args}
  ${java_args}
  atlas_command_t.lcm
  force_torque_t.lcm
  gps_data_t.lcm
  gps_satellite_info_list_t.lcm
  gps_satellite_info_t.lcm
  image_metadata_t.lcm
  images_t.lcm
  image_sync_t.lcm
  image_t.lcm
  ins_t.lcm
  joint_angles_t.lcm
  joint_state_t.lcm
  kvh_raw_imu_batch_t.lcm
  kvh_raw_imu_t.lcm
  planar_lidar_t.lcm
  pointcloud2_t.lcm
  pointcloud_t.lcm
  pointfield_t.lcm
  pose_t.lcm
  position_3d_t.lcm
  quaternion_t.lcm
  raw_t.lcm
  rigid_transform_t.lcm
  robot_state_t.lcm
  robot_urdf_t.lcm
  sensor_status_t.lcm
  six_axis_force_torque_array_t.lcm
  six_axis_force_torque_t.lcm
  system_status_t.lcm
  twist_t.lcm
  utime_t.lcm
  vector_3d_t.lcm
  viewer_command_t.lcm
  viewer_draw_t.lcm
  viewer_geometry_data_t.lcm
  viewer_link_data_t.lcm
  viewer_load_robot_t.lcm
)

lcm_add_library(lcmtypes_bot2-core C ${c_sources} ${c_install_headers})
generate_export_header(lcmtypes_bot2-core
  BASE_NAME bot_core_lcmtypes)
target_include_directories(lcmtypes_bot2-core INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

lcm_add_library(lcmtypes_bot2-core-cpp CPP ${cpp_install_headers})
target_include_directories(lcmtypes_bot2-core-cpp INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

lcm_install_headers(
  ${CMAKE_CURRENT_BINARY_DIR}/bot_core_lcmtypes_export.h
  ${c_install_headers}
  ${cpp_install_headers}
  DESTINATION include/lcmtypes
)

install(TARGETS lcmtypes_bot2-core lcmtypes_bot2-core-cpp
  EXPORT ${PROJECT_NAME}Targets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib${LIB_SUFFIX}
  ARCHIVE DESTINATION lib${LIB_SUFFIX}
  INCLUDES DESTINATION include/lcmtypes
)

if(PYTHONINTERP_FOUND)
  lcm_install_python(${python_install_sources})
endif()

if(JAVA_FOUND)
  if(NOT TARGET lcm-java)
    message(STATUS "lcm-java not found: not building Java LCM type bindings")
  else()
    add_jar(lcmtypes_bot2-core-jar
      OUTPUT_NAME lcmtypes_bot2-core
      INCLUDE_JARS lcm-java
      SOURCES ${java_sources}
    )

    install_jar(lcmtypes_bot2-core-jar share/java)

    set(BOT2-CORE_INCLUDE_JAVA
      "include(\${CMAKE_CURRENT_LIST_DIR}/bot2-coreJavaTargets.cmake)"
    )
    export_jars(
      FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}JavaTargets.cmake
      TARGETS lcmtypes_bot2-core-jar
    )
    install_jar_exports(
      FILE ${PROJECT_NAME}JavaTargets.cmake
      DESTINATION ${CONFIG_INSTALL_DIR}
      TARGETS lcmtypes_bot2-core-jar
    )
  endif()
endif()

configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${CONFIG_INSTALL_DIR}
)

# Exported targets for build directory
export(
  TARGETS lcmtypes_bot2-core
  FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake
)

# Exported targets for installation
install(EXPORT ${PROJECT_NAME}Targets
  DESTINATION ${CONFIG_INSTALL_DIR}
  FILE ${PROJECT_NAME}Targets.cmake
)

# Install package configuration files
install(FILES
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  DESTINATION ${CONFIG_INSTALL_DIR}
)
