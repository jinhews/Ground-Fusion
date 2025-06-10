cmake_minimum_required(VERSION 2.8.3)
project(vins)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++17")

# -DEIGEN_USE_MKL_ALL")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g")

# NEW
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  geometry_msgs
  nav_msgs
  tf
  cv_bridge
  camera_models
  image_transport

  rospy

  actionlib
  nodelet
  message_generation
)

list(APPEND LIBRARIES "m")

# Find X11
message(STATUS "Searching for X11...")
find_package(X11 REQUIRED)

if(X11_FOUND)
  include_directories(${X11_INCLUDE_DIR})
  link_libraries(${X11_LIBRARIES})
  message(STATUS " X11_INCLUDE_DIR: " ${X11_INCLUDE_DIR})
  message(STATUS " X11_LIBRARIES: " ${X11_LIBRARIES})
endif(X11_FOUND)

# Find rquired packeges
find_package(Boost REQUIRED COMPONENTS thread)
find_package(OpenCV 4 REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

add_definitions(-DOPENCV)
add_definitions(-O4 -g)

include_directories(
  include
  ${Boost_INCLUDE_DIRS}
)


message(WARNING "OpenCV_VERSION: ${OpenCV_VERSION}")

find_package(Ceres REQUIRED)

include_directories(${catkin_INCLUDE_DIRS} ${CERES_INCLUDE_DIRS})

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
find_package(Eigen3)
include_directories(
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}

  # line
  ${CMAKE_CURRENT_SOURCE_DIR}/src/linefeatureTracker/line_descriptor/include
)

add_library(vins_lib
  src/estimator/parameters.cpp
  src/estimator/estimator.cpp
  src/estimator/feature_manager.cpp
  src/factor/pose_local_parameterization.cpp
  src/factor/pose_subset_parameterization.cpp # new
  src/factor/orientation_local_parameterization.cpp
  src/factor/orientation_subset_parameterization.cpp
  src/factor/projectionTwoFrameOneCamFactor.cpp
  src/factor/projectionTwoFrameTwoCamFactor.cpp
  src/factor/projectionOneFrameTwoCamFactor.cpp
  src/factor/marginalization_factor.cpp
  src/utility/utility.cpp
  src/utility/visualization.cpp
  src/utility/CameraPoseVisualization.cpp
  src/initial/solve_5pts.cpp
  src/initial/initial_aligment.cpp
  src/initial/initial_sfm.cpp
  src/initial/initial_ex_rotation.cpp
  src/featureTracker/feature_tracker.cpp

  # line
  src/linefeatureTracker/linefeature_tracker.cpp
  src/linefeatureTracker/line_descriptor/src/binary_descriptor_custom.cpp
  src/linefeatureTracker/line_descriptor/src/binary_descriptor_matcher.cpp
  src/linefeatureTracker/line_descriptor/src/draw_custom.cpp
  src/linefeatureTracker/line_descriptor/src/LSDDetector_custom.cpp
  src/utility/line_geometry.cpp
  src/factor/line_projection_factor.cpp
  src/factor/line_parameterization.cpp
  src/factor/projection_factor.cpp # new

  src/factor/pos_vel_factor.cpp
  src/factor/pose_anchor_factor.cpp
)
target_link_libraries(vins_lib ${catkin_LIBRARIES} ${OpenCV_LIBS} ${CERES_LIBRARIES} /usr/local/lib/libSophus.so fmt)

add_executable(vins_node src/rosNodeTest.cpp)
target_link_libraries(vins_node vins_lib /usr/local/lib/libSophus.so
  ${CMAKE_CURRENT_SOURCE_DIR}/src/linefeatureTracker/line_descriptor/lib/liblinedesc.so
)
