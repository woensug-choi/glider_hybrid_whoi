cmake_minimum_required(VERSION 2.8.3)
project(hybrid_glider_gazebo_plugins)

# Specify C++11 standard
add_definitions(-std=c++11)

find_package(catkin REQUIRED COMPONENTS
    gazebo_dev)
find_package(Boost REQUIRED COMPONENTS system)
find_package(Eigen3 REQUIRED)
find_package(Protobuf REQUIRED)

file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/msgs)

set(hybrid_glider_GAZEBO_PLUGINS_LIST "")

# add_subdirectory(msgs)
set(PROTOBUF_IMPORT_DIRS "")
foreach(ITR ${GAZEBO_INCLUDE_DIRS})
  if(ITR MATCHES ".*gazebo-[0-9.]+$")
    set(PROTOBUF_IMPORT_DIRS "${ITR}/gazebo/msgs/proto")
  endif()
endforeach()

message(STATUS "PROTOBUF_IMPORT_DIRS = " ${PROTOBUF_IMPORT_DIRS})

set(GAZEBO_MSG_INCLUDE_DIRS)
foreach(ITR ${GAZEBO_INCLUDE_DIRS})
  if(ITR MATCHES ".*gazebo-[0-9.]+$")
    set(GAZEBO_MSG_INCLUDE_DIRS "${ITR}/gazebo/msgs")
  endif()
endforeach()

message(STATUS "GAZEBO_MSG_INCLUDE_DIRS = " ${GAZEBO_MSG_INCLUDE_DIRS})

file(GLOB msgs msgs/*.proto)

catkin_package(
  INCLUDE_DIRS
    include
    ${CMAKE_CURRENT_BINARY_DIR} # for generated messages
    ${GAZEBO_MSG_INCLUDE_DIRS}
  LIBRARIES
    hybrid_glider_buoyancypump_plugin
    hybrid_glider_slidingmass_plugin
    hybrid_glider_underwater_object_plugin
    hybrid_glider_thruster_plugin
    hybrid_glider_fin_plugin
    hybrid_glider_dynamics
)

include_directories(${PROJECT_SOURCE_DIR}/include
                    ${CMAKE_CURRENT_BINARY_DIR} # for generated messages
                    ${Boost_INCLUDE_DIR}
                    ${catkin_INCLUDE_DIRS}
                    ${GAZEBO_MSG_INCLUDE_DIRS}
                    ${EIGEN3_INCLUDE_DIRS})

link_directories(
  ${catkin_LIBRARY_DIRS}
  ${EIGEN3_LIBRARY_DIRS})

###########
## Build ##
###########

protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS ${msgs})
add_library(hybrid_glider_gazebo_plugins_msgs SHARED ${PROTO_SRCS})
target_link_libraries(hybrid_glider_gazebo_plugins_msgs ${PROTOBUF_LIBRARY})
list(APPEND hybrid_glider_GAZEBO_PLUGINS_LIST hybrid_glider_gazebo_plugins_msgs)

add_library(hybrid_glider_dynamics
    SHARED
    src/Dynamics.cc
)
list(APPEND hybrid_glider_GAZEBO_PLUGINS_LIST hybrid_glider_dynamics)

add_library(hybrid_glider_fin_plugin
    src/LiftDragModel.cc
    src/FinPlugin.cc
)
target_link_libraries(hybrid_glider_fin_plugin
    hybrid_glider_dynamics
    hybrid_glider_gazebo_plugins_msgs
    ${catkin_LIBRARIES}
    ${Boost_LIBRARIES})
add_dependencies(hybrid_glider_fin_plugin
  hybrid_glider_gazebo_plugins_msgs
  ${catkin_EXPORTED_TARGETS})
list(APPEND hybrid_glider_GAZEBO_PLUGINS_LIST hybrid_glider_fin_plugin)

add_library(hybrid_glider_underwater_object_plugin
    src/HydrodynamicModel.cc
    src/BuoyantObject.cc
    src/UnderwaterObjectPlugin.cc
)
target_link_libraries(hybrid_glider_underwater_object_plugin
    ${catkin_LIBRARIES}
    ${EIGEN3_LIBRARIES}
)
list(APPEND hybrid_glider_GAZEBO_PLUGINS_LIST hybrid_glider_underwater_object_plugin)

add_library(hybrid_glider_thruster_plugin
    src/ThrusterConversionFcn.cc
    src/ThrusterPlugin.cc
)
target_link_libraries(hybrid_glider_thruster_plugin
    hybrid_glider_dynamics
    hybrid_glider_gazebo_plugins_msgs
    ${catkin_LIBRARIES}
    ${Boost_LIBRARIES})
add_dependencies(hybrid_glider_thruster_plugin
    hybrid_glider_gazebo_plugins_msgs
    ${catkin_EXPORTED_TARGETS})
list(APPEND hybrid_glider_GAZEBO_PLUGINS_LIST hybrid_glider_thruster_plugin)

add_library(hybrid_glider_umbilical_plugin
  src/UmbilicalModel.cc
  src/UmbilicalPlugin.cc)
target_link_libraries(hybrid_glider_umbilical_plugin
  ${catkin_LIBRARIES} ${Boost_LIBRARIES})
list(APPEND hybrid_glider_GAZEBO_PLUGINS_LIST hybrid_glider_umbilical_plugin)

add_library(hybrid_glider_slidingmass_plugin
    src/SlidingMassPlugin.cc
)
target_link_libraries(hybrid_glider_slidingmass_plugin
    hybrid_glider_dynamics
    hybrid_glider_gazebo_plugins_msgs
    ${catkin_LIBRARIES}
    ${Boost_LIBRARIES})
add_dependencies(hybrid_glider_slidingmass_plugin
    hybrid_glider_gazebo_plugins_msgs
    ${catkin_EXPORTED_TARGETS})
list(APPEND hybrid_glider_GAZEBO_PLUGINS_LIST hybrid_glider_slidingmass_plugin)

add_library(hybrid_glider_buoyancypump_plugin
    src/BuoyancyPumpPlugin.cc
)
target_link_libraries(hybrid_glider_buoyancypump_plugin
    hybrid_glider_dynamics
    hybrid_glider_gazebo_plugins_msgs
    ${catkin_LIBRARIES}
    ${Boost_LIBRARIES})
add_dependencies(hybrid_glider_buoyancypump_plugin
    hybrid_glider_gazebo_plugins_msgs
    ${catkin_EXPORTED_TARGETS})
list(APPEND hybrid_glider_GAZEBO_PLUGINS_LIST hybrid_glider_buoyancypump_plugin)

#############
## Install ##
#############

install(TARGETS ${hybrid_glider_GAZEBO_PLUGINS_LIST}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.hh"
  PATTERN "*~" EXCLUDE
)

install(DIRECTORY include/
   DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
   FILES_MATCHING PATTERN ".hh"
)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.pb.*"
  PATTERN "*~" EXCLUDE
)

#############
## Testing ##
#############

if(CATKIN_ENABLE_TESTING)
  catkin_add_gtest(UNIT_ThrusterConversionFcn_TEST_HYBRID_GLIDER
    src/ThrusterConversionFcn_TEST.cc)
  target_link_libraries(UNIT_ThrusterConversionFcn_TEST_HYBRID_GLIDER
    hybrid_glider_thruster_plugin
    ${catkin_LIBRARIES})

  catkin_add_gtest(UNIT_ThrusterDynamics_TEST_HYBRID_GLIDER
    src/ThrusterDynamics_TEST.cc)
  target_link_libraries(UNIT_ThrusterDynamics_TEST_HYBRID_GLIDER
    hybrid_glider_dynamics
    ${catkin_LIBRARIES})
endif()
