# Designed to be run from the main CMakeLists.txt, in the root of the Coriander repo
# via include(...)

set(TESTS_EXCLUDE EXCLUDE_FROM_ALL)
if(BUILD_TESTS)
  set(TESTS_EXCLUDE)
endif(BUILD_TESTS)

# add_subdirectory(test/gtest)
add_subdirectory(endtoend)

add_custom_target(tests
    DEPENDS clblast clew easycl cocl patch_hostside ${BUILD_TEST_TARGETS})
add_custom_target(run-tests
    DEPENDS tests ${RUN_TEST_TARGETS})
