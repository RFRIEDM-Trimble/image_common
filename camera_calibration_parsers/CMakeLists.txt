cmake_minimum_required(VERSION 3.5)

project(camera_calibration_parsers)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(ament_cmake_auto REQUIRED)

# set(CMAKE_FIND_DEBUG_MODE TRUE)
find_package(rclcpp REQUIRED)
# set(CMAKE_FIND_DEBUG_MODE FALSE)
find_package(sensor_msgs REQUIRED)
find_package(yaml_cpp_vendor REQUIRED)
message("TARGETS: ${yaml_cpp_vendor_TARGETS}")

# define the library
add_library(${PROJECT_NAME}
  src/parse.cpp
  src/parse_ini.cpp
  src/parse_yml.cpp
)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

target_compile_definitions(${PROJECT_NAME} PRIVATE "CAMERA_CALIBRATION_PARSERS_BUILDING_DLL")

if(CMAKE_COMPILER_IS_GNUCXX)
  target_link_libraries(${PROJECT_NAME} PUBLIC stdc++fs)
endif()

message("Include Dirs for RCLCPP: ${rclcpp_INCLUDE_DIRS}")
target_link_libraries(${PROJECT_NAME} PUBLIC
  rclcpp::rclcpp  # rclcpp::rclcpp generates linker errors if used
  sensor_msgs::sensor_msgs_library # https://github.com/ros2/common_interfaces/pull/178
  yaml-cpp # https://github.com/jbeder/yaml-cpp/pull/741
)

# ament_target_dependencies(
#   ${PROJECT_NAME}
#   rclcpp
#   rcpputils # If uncommented, add a call to find_package and ament_export_dependencies
#   sensor_msgs
#   yaml_cpp_vendor
# )

# TODO: Reenable Python Wrapper with new serialization technique.
#find_package(PythonLibs REQUIRED)
#if(PYTHONLIBS_VERSION_STRING VERSION_LESS 3)
#  find_package(Boost REQUIRED COMPONENTS python)
#else()
#  find_package(Boost REQUIRED COMPONENTS python3)
#endif()
#add_library(${PROJECT_NAME}_wrapper
#  src/parse_wrapper.cpp)
#ament_target_dependencies(${PROJECT_NAME}_wrapper
#  "rclcpp"
#  "sensor_msgs"
#)
#target_include_directories(${PROJECT_NAME}_wrapper PUBLIC ${Boost_INCLUDE_DIRS} ${PYTHON_INCLUDE_DIRS})
#target_link_libraries(${PROJECT_NAME}_wrapper ${PROJECT_NAME} ${Boost_LIBRARIES} ${PYTHON_LIBRARIES} ${YAML_CPP_LIBRARIES})

# define the exe to convert
add_executable(convert src/convert.cpp)
target_link_libraries(convert PRIVATE ${PROJECT_NAME}::${PROJECT_NAME})

install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  EXPORT export_${PROJECT_NAME}
  NAMESPACE ${PROJECT_NAME}::
  DESTINATION lib/cmake/
)

install(
  TARGETS convert
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

# ament_export_include_directories(include)
if(NOT TARGET ${PROJECT_NAME}::${PROJECT_NAME})
  message(ERROR "Unable to export not found targets")
endif()
ament_export_libraries(${PROJECT_NAME})
ament_export_dependencies(rclcpp sensor_msgs yaml_cpp_vendor)


if(BUILD_TESTING)
  #TODO(mjcarroll) switch back to this once I can fix copyright check
  #find_package(ament_lint_auto REQUIRED)
  #ament_lint_auto_find_test_dependencies()

  find_package(ament_cmake_cppcheck REQUIRED)
  find_package(ament_cmake_cpplint REQUIRED)
  find_package(ament_cmake_lint_cmake REQUIRED)
  find_package(ament_cmake_uncrustify REQUIRED)
  #TODO(mjcarroll) Headers need to be .hpp for this to work properly.
  #ament_cppcheck()
  ament_cpplint()
  ament_lint_cmake()
  ament_uncrustify()
  include(GoogleTest)

  gtest_add_tests(TARGET ${PROJECT_NAME}-parse_ini SOURCES test/test_parse_ini.cpp)
  if(TARGET ${PROJECT_NAME}-parse_ini)
    target_link_libraries(${PROJECT_NAME}-parse_ini PRIVATE ${PROJECT_NAME})
  endif()

  gtest_add_tests(TARGET ${PROJECT_NAME}-parse_yml SOURCES test/test_parse_yml.cpp)
  if(TARGET ${PROJECT_NAME}-parse_yml)
    target_link_libraries(${PROJECT_NAME}-parse_yml PRIVATE ${PROJECT_NAME})
  endif()
endif()

ament_auto_package()
