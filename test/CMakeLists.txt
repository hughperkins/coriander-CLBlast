cmake_minimum_required(VERSION 4.0.0 FATAL_ERROR)

# Originally designed to be run from the main CMakeLists.txt, in the root of the Coriander repo
# via include(...)

if(TARGET clblast)
    # runnign embedded
else()
    # running standalone
    include($ENV{COCL_CMAKE}/cocl_vars.cmake)
    include($ENV{COCL_CMAKE}/cocl_impl.cmake)
    include(${COCL_LIB}/cocl/cocl-targets.cmake)
    include(${COCL_LIB}/coriander_plugins/corianderclblast-targets.cmake)
    if(WIN32)
        set(CMAKE_CXX_FLAGS "/EHsc")
    else()
        set(CMAKE_CXX_FLAGS "-std=c++11 -fPIC -g")
        set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-rpath,$ORIGIN")
    endif()
endif()

# Now designed to run standalone too

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
