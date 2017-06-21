cmake_minimum_required(VERSION 3.0.0)

option(BUILD_TESTS OFF)

IF(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    if(WIN32)
        SET(CMAKE_INSTALL_PREFIX
            "$ENV{HOME}/coriander" CACHE PATH "Installation prefix, default '${HOME}/coriander'" FORCE
        )
    else()
        SET(CMAKE_INSTALL_PREFIX
            "$ENV{HOME}/coriander" CACHE PATH "Installation prefix, default '${HOME}/coriander'" FORCE
        )
    endif()
ENDIF(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

# if(WIN32)
#     # set(CORIANDER_DIR "C:\\Program Files\\Coriander" CACHE PATH "Installation directory of Coriander")
#     set(CLANG_HOME "C:\\Program Files\\LLVM" CACHE PATH "the downloaded clang-4.0.0 folder, containing lib, bin etc")
# else()
#     set(CLANG_HOME "/usr/local/opt/llvm-4.0" CACHE PATH "the downloaded clang-4.0.0 folder, containing lib, bin etc")
#     # set(CORIANDER_DIR "$ENV{HOME}/coriander" CACHE PATH "Installation directory of Coriander")
# endif()

SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins")
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
SET(CMAKE_MACOSX_RPATH TRUE)

set(CORIANDER_DIR ${CMAKE_INSTALL_PREFIX})
include($ENV{COCL_CMAKE}/cocl.cmake)

if(WIN32)
    set(CMAKE_CXX_FLAGS "/EHsc ${PLATFORM_OPTIONS}")
else()
    set(CMAKE_CXX_FLAGS "-std=c++11 -fPIC -g ${PLATFORM_OPTIONS}")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-rpath,$ORIGIN")
endif()
add_definitions(-DUSE_CLEW)

add_library(corianderclblast SHARED src/cocl_blas.cpp)
foreach(libname cocl clblast easycl clew)
    if(APPLE)
        target_link_libraries(corianderclblast "${CORIANDER_DIR}/lib/lib${libname}.dylib")
    elseif(WIN32)
        target_link_libraries(corianderclblast "${CORIANDER_DIR}\\lib\\lib${libname}.lib")
    else()
        target_link_libraries(corianderclblast "${CORIANDER_DIR}/lib/lib${libname}.so")
    endif()
endforeach()
target_include_directories(corianderclblast PRIVATE "${CORIANDER_DIR}/include")
target_include_directories(corianderclblast PRIVATE "include")
target_include_directories(corianderclblast SYSTEM BEFORE PRIVATE "${CORIANDER_DIR}/include/clew/proxy-headers")
target_include_directories(corianderclblast PRIVATE ${CLANG_HOME}/include)
target_include_directories(corianderclblast PRIVATE src/CLBlast/include BEFORE)

add_subdirectory(test)

CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/corianderclblast.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake/corianderclblast.cmake" @ONLY)
INSTALL(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/cmake/corianderclblast.cmake DESTINATION share/cocl/coriander_plugins)

INSTALL(FILES include/cudnn.h DESTINATION ${CORIANDER_DIR}/include/coriander_plugins/corianderclblast)
FILE(GLOB DNN_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/include/coriander-dnn/*.h)
INSTALL(FILES ${DNN_HEADERS} DESTINATION ${CORIANDER_DIR}/include/coriander_plugins/corianderclblast/coriander-dnn)

install(TARGETS corianderclblast EXPORT corianderclblast-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins
    ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins
)
install(EXPORT corianderclblast-targets DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins)
