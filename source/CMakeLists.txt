# Specify .cpp files
set(TARGET_NAME ${PROJECT_NAME})
set(SRC
  debug/backtrace.cpp)

# Build a library from all specified source files
# This is required for using Google Test
add_library(${PROJECT_NAME}-lib ${SRC})
target_link_libraries(${PROJECT_NAME}-lib ${LIBRARIES})
set_target_properties(${PROJECT_NAME}-lib PROPERTIES OUTPUT_NAME ${PROJECT_NAME})

# Build an executable from main.cpp and all the specified source files
add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${TARGET_NAME} ${LIBRARIES} ${PROJECT_NAME}-lib)
