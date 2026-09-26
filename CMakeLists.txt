cmake_minimum_required(VERSION 3.28.3)
if(NOT CMAKE_C_COMPILER)
  set(CMAKE_C_COMPILER clang CACHE STRING "C compiler: clang" FORCE)
endif()
project(llvm-playground VERSION 0.1.0
                        DESCRIPTION "Learning about compilers"
                        LANGUAGES C)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type: Release" FORCE)
endif()
message(STATUS "CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
file(GLOB MY_SOURCES CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/*.c)
message(STATUS "SOURCES: ${MY_SOURCES}")
add_executable(${CMAKE_PROJECT_NAME}_${CMAKE_PROJECT_VERSION} ${MY_SOURCES})
target_include_directories(${CMAKE_PROJECT_NAME}_${CMAKE_PROJECT_VERSION}
                            PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
#cmake --build build
# tests
#
add_executable(lexer_test_runner
    tests/lexer-main.c
    lexer.c
)
target_include_directories(lexer_test_runner
                            PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
enable_testing()

file(GLOB TEST_INPUTS CONFIGURE_DEPENDS
          "${CMAKE_CURRENT_SOURCE_DIR}/tests/*.ie")
set(TEST_FILES "")
foreach(INPUT_FILE ${TEST_INPUTS})
  get_filename_component(TEST_NAME ${INPUT_FILE} NAME_WE)

  set(EXPECTED_FILE
      "${CMAKE_CURRENT_SOURCE_DIR}/tests/lexer-expected-outs/${TEST_NAME}.out")
  set(ACTUAL_FILE "${CMAKE_CURRENT_BINARY_DIR}/${TEST_NAME}.actual")

  # 1. Run the lexer test executable using lexer-main.c
  add_test(NAME RunLexer_${TEST_NAME}
  COMMAND $<TARGET_FILE:lexer_test_runner> ${INPUT_FILE} ${ACTUAL_FILE})

  list(APPEND ALL_ACTUAL_FILES ${ACTUAL_FILE})
  # add these files for cleanup in the main clean too
  set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_CLEAN_FILES "${ACTUAL_FILE}")

  # 2. Compare the actual output file with the expected output file
  add_test(NAME Compare_${TEST_NAME}
    COMMAND ${CMAKE_COMMAND} -E compare_files ${EXPECTED_FILE} ${ACTUAL_FILE})

  set_tests_properties(Compare_${TEST_NAME}
    PROPERTIES DEPENDS RunLexer_${TEST_NAME})
endforeach() # ctest --test-dir build --output-on-failure

# custom clean-tests
add_custom_target(clean-tests
    COMMAND ${CMAKE_COMMAND} -E remove ${ALL_ACTUAL_FILES}
    COMMENT "Cleaning test .actual files..."
)
