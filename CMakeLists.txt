cmake_minimum_required(VERSION 3.8)
project(serial)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)


if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

# if(UNIX AND NOT APPLE)
#     # If Linux, add rt and pthread
#     set(rt_LIBRARIES rt)
#     set(pthread_LIBRARIES pthread)
#     # catkin_package(
#     #     LIBRARIES ${PROJECT_NAME}
#     #     INCLUDE_DIRS include
#     #     DEPENDS rt pthread
#     # )
#     ament_target_dependencies(${PROJECT_NAME}
#         rt
#         pthread
#     )
# else()
#     # Otherwise normal call
#     # catkin_package(
#     #     LIBRARIES ${PROJECT_NAME}
#     #     INCLUDE_DIRS include
#     # )
# endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)
if(APPLE)
    # If OSX
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})
if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    # target_link_libraries(${PROJECT_NAME} rt pthread)
    # find_package(rt REQUIRED)
    find_library(LIB_RT  rt)
    set(THREADS_PREFER_PTHREAD_FLAG ON)
    find_package(Threads REQUIRED)
    set(rt_LIBRARIES rt)
    target_link_libraries(${PROJECT_NAME} rt Threads::Threads)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()

## Uncomment for example
# add_executable(serial_example examples/serial_example.cc)
# add_dependencies(serial_example ${PROJECT_NAME})
# target_link_libraries(serial_example ${PROJECT_NAME})

## Include headers
include_directories(include)

# ## Install executable
# install(TARGETS ${PROJECT_NAME}
#     ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
# )

# ## Install headers
# install(FILES include/serial/serial.h include/serial/v8stdint.h
#   DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/serial)

target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

install(TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

install(DIRECTORY include/ DESTINATION include)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})


# Too lazy to rewrite tests cmake stuff lmao
## Tests
# if(BUILD_TESTING)
#     add_subdirectory(tests)
# endif()

ament_package()
