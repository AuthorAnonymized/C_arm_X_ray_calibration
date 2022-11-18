cmake_minimum_required(VERSION 3.15)
project(x_ray_calibration)

set(OpenCV_DIR /opt/opencv-3.2.0/share/OpenCV/)
find_package(OpenCV REQUIRED)
# Print some message showing some of them
message(STATUS "OpenCV library status:")
message(STATUS "    version: ${OpenCV_VERSION}")
message(STATUS "    libraries: ${OpenCV_LIBS}")
message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")


set(CMAKE_CXX_STANDARD 14)

add_executable(x_ray_calibration calibration.cpp)
target_link_libraries(x_ray_calibration ${OpenCV_LIBS})

add_executable(imagelist_creator imagelist_creator.cpp)
target_link_libraries(imagelist_creator ${OpenCV_LIBS})