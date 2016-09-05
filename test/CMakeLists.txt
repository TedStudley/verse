# //==========================\\
# || Google Testing Framework ||
# \\==========================//
include(gtest)

# Use the proper library suffix for the current environment
if(MSVC)
  set(LIB_SUFFIX ".lib")
else()
  set(LIB_SUFFIX ".a")
  set(PTHREAD_FLAG "-pthread")
endif()

if(APPLE)
  # Ensure that GTest can compile properly on OSX
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -U__STRING_ANSI__")
endif()

# Include the GTest framework files
ExternalProject_Get_Property(googletest source_dir)
include_directories(${source_dir}/include)

# Glob recurse all of the test source files
file(GLOB_RECURSE TEST_FILES
    RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
    *.cpp)

set(GTEST_TARGET ${PROJECT_NAME}-test)
add_executable(${GTEST_TARGET}
    EXCLUDE_FROM_ALL ${TEST_FILES})
add_dependencies(${GTEST_TARGET}
    ${GTEST_LIBRARY}
    ${PROJECT_NAME}-lib)
target_link_libraries(${GTEST_TARGET}
    ${PROJECT_NAME}-lib
    ${LIBRARIES}
    ${binary_dir}/${CMAKE_FIND_LIBRARY_PREFIXES}gtest${LIB_SUFFIX}
    ${binary_dir}/${CMAKE_FIND_LIBRARY_PREFIXES}gtest_main${LIB_SUFFIX})

if(CMAKE_COMPILER_IS_GNUC OR CMAKE_COMPILER_IS_GNUCXX)
  # Include pthread if the compiler is GNU
  target_link_libraries(${GTEST_TARGET} pthread)
endif()

# Add a custom target to automatically run all tests
SET(ENV{GTEST_OUTPUT} "xml:${CMAKE_BINARY_DIR}/test.xml")
add_custom_target(check
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMAND ./${GTEST_TARGET} || true)
add_dependencies(check ${GTEST_TARGET})

if(COVERAGE_ENABLED)
  # Add a custom target to automatically check test code coverage
  find_package(gcovr REQUIRED)
  add_custom_target(coverage
      WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
      COMMAND ${GCOVR_EXECUTABLE} -r ${CMAKE_BINARY_DIR} -f".*mc-mini/\(source|include\).*" --xml-pretty -o ${CMAKE_BINARY_DIR}/coverage.xml)
  add_dependencies(coverage check)
endif()
