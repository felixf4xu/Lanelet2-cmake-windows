# Modify only if you know what you are doing!
cmake_minimum_required(VERSION 3.5.1)
project(lanelet2_projection)

add_compile_definitions(_USE_MATH_DEFINES)

###################
## Find packages ##
###################
find_package(ament_cmake REQUIRED)

# https://bugs.launchpad.net/ubuntu/+source/geographiclib/+bug/1805173
if(CMAKE_HOST_SYSTEM_NAME STREQUAL "Linux")
  set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};/usr/share/cmake/geographiclib")
endif()
find_package(GeographicLib REQUIRED)

############################
## Read source code files ##
############################
file(GLOB_RECURSE PROJECT_SOURCE_FILES_SRC
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
)

###########
## Build ##
###########
# Declare a cpp library
add_library(${PROJECT_NAME}    
    ${PROJECT_SOURCE_FILES_SRC}
    )
target_include_directories(${PROJECT_NAME}  
    PUBLIC
      "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
      "$<INSTALL_INTERFACE:include>")
target_include_directories(${PROJECT_NAME} 
    PRIVATE ${GeographicLib_INCLUDE_DIRS}
  )      
target_link_libraries(${PROJECT_NAME} PUBLIC ${GeographicLib_LIBRARIES})

find_package(lanelet2_io  REQUIRED)      
ament_target_dependencies(${PROJECT_NAME} PUBLIC lanelet2_io)   

#############
## Install ##
#############
install(
  DIRECTORY include/lanelet2_projection
  DESTINATION include/
)
install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)
ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(GeographicLib)
ament_export_dependencies(lanelet2_io)

#############
## Testing ##
#############
if(BUILD_TESTING)
  find_package(ament_cmake_gtest REQUIRED)
  file(GLOB TEST_SOURCES "test/*.cpp")
  # Add ament_gtest for your test file
  foreach(TEST_SOURCE ${TEST_SOURCES})
    get_filename_component(TEST_NAME ${TEST_SOURCE} NAME_WE)
    ament_add_gtest(${TEST_NAME} ${TEST_SOURCE})
    target_link_libraries(${TEST_NAME}
      ${PROJECT_NAME}
      ${ament_LIBRARIES}
    )

  endforeach()
endif()

ament_package(CONFIG_EXTRAS "cmake/ConfigExtras.cmake")
