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

SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins")
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
SET(CMAKE_MACOSX_RPATH TRUE)

set(CORIANDER_DIR ${CMAKE_INSTALL_PREFIX})
include(${CORIANDER_DIR}/share/cocl/cocl_vars.cmake)
include(${CORIANDER_DIR}/share/cocl/cocl_impl.cmake)
include(${COCL_LIB}/cocl/cocl-targets.cmake)

if(WIN32)
    set(CMAKE_CXX_FLAGS "/EHsc ${PLATFORM_OPTIONS}")
else()
    set(CMAKE_CXX_FLAGS "-std=c++11 -fPIC -g ${PLATFORM_OPTIONS}")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-rpath,$ORIGIN")
endif()

add_definitions(-DUSE_CLEW)

# ==================================================================================================

# next ~20 lines or so are copied from CLBlast CMakeLists.txt (seems easier than figuring out the whole cmake import/export
# bit...)
# I prefixed the vars with CLBLAST_ , and removed all the level 1/2 routines, which we dont use

# Sets the supported routines and the used kernels. New routines and kernels should be added here.
set(CLBLAST_KERNELS copy_fast copy_pad transpose_fast transpose_pad xaxpy xdot xger xgemm xgemv)
set(CLBLAST_LEVEL1_ROUTINES xswap xscal xcopy xaxpy xdot xdotu xdotc xnrm2 xasum xamax)
set(CLBLAST_LEVEL2_ROUTINES xgemv xgbmv xhemv xhbmv xhpmv xsymv xsbmv xspmv xtrmv xtbmv xtpmv
                    xger xgeru xgerc xher xhpr xher2 xhpr2 xsyr xspr xsyr2 xspr2 xtrsv)
set(CLBLAST_LEVEL3_ROUTINES xgemm xsymm xhemm xsyrk xherk xsyr2k xher2k xtrmm xtrsm)
set(CLBLAST_LEVELX_ROUTINES xomatcopy xaxpybatched xgemmbatched xinvert)
set(CLBLAST_ROUTINES ${CLBLAST_LEVEL1_ROUTINES} ${CLBLAST_LEVEL2_ROUTINES} ${CLBLAST_LEVEL3_ROUTINES} ${CLBLAST_LEVELX_ROUTINES})
set(CLBLAST_PRECISIONS 32 64 3232 6464 16)

# Gathers all source-files
set(CLBLAST_SOURCES
    src/CLBlast/src/database/database.cpp
    src/CLBlast/src/routines/common.cpp
    src/CLBlast/src/cache.cpp
    src/CLBlast/src/clblast.cpp
    src/CLBlast/src/clblast_c.cpp
    src/CLBlast/src/routine.cpp
    src/CLBlast/src/utilities/clblast_exceptions.cpp
    src/CLBlast/src/utilities/utilities.cpp
)
foreach(ROUTINE ${CLBLAST_LEVEL1_ROUTINES})
    set(CLBLAST_SOURCES ${CLBLAST_SOURCES} src/CLBlast/src/routines/level1/${ROUTINE}.cpp)
endforeach()
foreach(ROUTINE ${CLBLAST_LEVEL2_ROUTINES})
    set(CLBLAST_SOURCES ${CLBLAST_SOURCES} src/CLBlast/src/routines/level2/${ROUTINE}.cpp)
endforeach()
foreach(ROUTINE ${CLBLAST_LEVEL3_ROUTINES})
    set(CLBLAST_SOURCES ${CLBLAST_SOURCES} src/CLBlast/src/routines/level3/${ROUTINE}.cpp)
endforeach()
foreach(ROUTINE ${CLBLAST_LEVELX_ROUTINES})
    set(CLBLAST_SOURCES ${CLBLAST_SOURCES} src/CLBlast/src/routines/levelx/${ROUTINE}.cpp)
endforeach()


include_directories(src/CLBlast/src)
add_library(clblast SHARED ${CLBLAST_SOURCES})
if(APPLE)
    # remove covered switch build warnings
    target_compile_options(clblast PRIVATE -Wno-switch -DUSE_CLEW)
elseif(WIN32)
    target_compile_options(clblast PRIVATE /DCLBLAST_DLL /DCOMPILING_DLL=1 /DUSE_CLEW)
else()
endif()
target_include_directories(clblast PRIVATE src/CLBlast/include)
target_include_directories(clblast PRIVATE ${CORIANDER_DIR}/include)
target_include_directories(clblast SYSTEM BEFORE PRIVATE ${CORIANDER_DIR}/include/clew/proxy-headers)
target_link_libraries(clblast clew)

add_library(corianderclblast SHARED src/cocl_blas.cpp)
target_include_directories(corianderclblast PRIVATE "${CORIANDER_DIR}/include")
target_include_directories(corianderclblast PRIVATE "include")
target_include_directories(corianderclblast PRIVATE ${CLANG_HOME}/include)
target_include_directories(corianderclblast PRIVATE src/CLBlast/include BEFORE)
target_include_directories(corianderclblast SYSTEM BEFORE PRIVATE "${CORIANDER_DIR}/include/clew/proxy-headers")
target_link_libraries(corianderclblast clblast cocl easycl clew)

# for testing
set(CORIANDERCLBLAST_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/include)
add_subdirectory(test)

CONFIGURE_FILE(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/corianderclblast.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake/corianderclblast.cmake" @ONLY)
INSTALL(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/cmake/corianderclblast.cmake DESTINATION share/cocl/coriander_plugins)

INSTALL(FILES include/cublas_v2.h DESTINATION ${CORIANDER_DIR}/include/coriander_plugins/corianderclblast)
FILE(GLOB DNN_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/include/coriander-clblast/*.h)
INSTALL(FILES ${DNN_HEADERS} DESTINATION ${CORIANDER_DIR}/include/coriander_plugins/corianderclblast/coriander-clblast)

install(TARGETS clblast corianderclblast EXPORT corianderclblast-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins
    ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins
)
install(EXPORT corianderclblast-targets DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/coriander_plugins)
