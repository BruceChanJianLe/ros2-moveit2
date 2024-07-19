cmake_minimum_required(VERSION 3.22)
project(moveit_planners_ompl)

# Common cmake code applied to all moveit packages
find_package(moveit_common REQUIRED)
moveit_package()

# # FetchContent module is included by default in CMake >= 3.14
# include(FetchContent)
#
# # Fetch OMPL
# function(fetch_project)
#   cmake_parse_arguments(FETCH_SOURCE "" "NAME;URL;DESTINATION" "" ${ARGN})
#
#   FetchContent_Declare(${FETCH_SOURCE_NAME}
#     URL ${FETCH_SOURCE_URL}
#     SOURCE_DIR ${CMAKE_BINARY_DIR}/${FETCH_SOURCE_DESTINATION}
#     CMAKE_ARGS     -DOMPL_BUILD_SHARED_LIBS=ON
#     )
#
#   FetchContent_Populate(${FETCH_SOURCE_NAME})
#   #   URL ${FETCH_SOURCE_URL}
#   #   SOURCE_DIR ${CMAKE_BINARY_DIR}/${FETCH_SOURCE_DESTINATION}
#   # )
#
#   set(
#     "${FETCH_SOURCE_NAME}_LOCATION"
#     ${CMAKE_BINARY_DIR}/${FETCH_SOURCE_DESTINATION}
#     PARENT_SCOPE
#   )
# endfunction()
#
# fetch_project(
#   NAME ompl_src
#   URL https://github.com/ompl/ompl/archive/refs/tags/1.5.2.zip
#   DESTINATION ompl
# )
#
# message("LOCAL_OMPL_SOURCE_DIRS: ${ompl_src_LOCATION}")
# message("LOCAL_OMPL_BUILD_DIRS: ${ompl_BINARY_DIR}")

set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/ompl/install"
  CACHE STRING "Final install location.")
set(CMAKE_BUILD_TYPE_INIT "Release")

# Define OMPL ExternalProject
include(ExternalProject)
ExternalProject_Add(ompl
    URL https://github.com/ompl/ompl/archive/refs/tags/1.5.2.zip
    CMAKE_ARGS
      -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
      -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
      -DOMPL_BUILD_SHARED_LIBS=ON
      -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    # Optional: Define additional CMake arguments for OMPL
    # CMAKE_ARGS     -DCMAKE_INSTALL_PREFIX=<install-dir> -DOMPL_BUILD_EXAMPLES=OFF
    PREFIX ${CMAKE_BINARY_DIR}/ompl
    # Define where to build OMPL
    BINARY_DIR ${CMAKE_BINARY_DIR}/ompl/build
    # Define where to install OMPL
    INSTALL_DIR ${CMAKE_BINARY_DIR}/ompl/install
    BUILD_COMMAND ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR}/ompl/build
)

message("OMPL_SOURCE_DIRS: ${ompl_SOURCE_DIRS}")
message("OMPL_INCLUDE_DIRS: ${ompl_INCLUDE_DIRS}")
message("OMPL_LIBRARIES: ${ompl_LIBRARIES}")
message("OMPL_VERSION: ${ompl_VERSION}")
include_directories(${CMAKE_BINARY_DIR}/ompl/build/src ${CMAKE_BINARY_DIR}/ompl/src/ompl/src)

find_package(Boost REQUIRED system filesystem date_time thread serialization)
find_package(moveit_core REQUIRED)
find_package(moveit_msgs REQUIRED)
find_package(moveit_ros_planning REQUIRED)
find_package(rclcpp REQUIRED)
find_package(pluginlib REQUIRED)
find_package(tf2_eigen REQUIRED)
find_package(tf2_ros REQUIRED)
# find_package(ompl REQUIRED)

if(CMAKE_CXX_COMPILER_ID MATCHES "(GNU|Clang)")
    add_compile_options(-Wno-overloaded-virtual)
endif()

include_directories(ompl_interface/include)
include_directories(SYSTEM
                    ${Boost_INCLUDE_DIRS}
                    # ${OMPL_INCLUDE_DIRS}
                    )

add_subdirectory(ompl_interface)

install(TARGETS moveit_ompl_interface moveit_ompl_planner_plugin
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)
ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(moveit_core ompl)
# target_link_libraries(moveit_core ompl)

pluginlib_export_plugin_description_file(moveit_core ompl_interface_plugin_description.xml)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)

  # These don't pass yet, disable them for now
  set(ament_cmake_copyright_FOUND TRUE)
  set(ament_cmake_cpplint_FOUND TRUE)
  set(ament_cmake_uncrustify_FOUND TRUE)

  # Run all lint tests in package.xml except those listed above
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
