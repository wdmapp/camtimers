
cmake_minimum_required(VERSION 3.12)

project(Camtimers)
ENABLE_LANGUAGE(C Fortran)
set(Fortran_FORMAT FREE)

find_package(MPI REQUIRED)
find_package(OpenMP)

# CLone the perfstubs repo if it doesn't exist

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/perfstubs/CMakeLists.txt")
include(cmake/GitUtils.cmake)
git_clone(
        PROJECT_NAME    perfstubs
        GIT_URL         https://github.com/khuck/perfstubs.git
        GIT_BRANCH      master
        DIRECTORY       ${PROJECT_SOURCE_DIR}
)
endif(NOT EXISTS "${PROJECT_SOURCE_DIR}/perfstubs/CMakeLists.txt")

# configure a header file to pass some of the CMake settings
# to the source code
if(NOT EXISTS "${PROJECT_SOURCE_DIR}/perfstubs/perfstubs_api/config.h")
set (PerfStubs_VERSION_MAJOR 0)
set (PerfStubs_VERSION_MINOR 1)
configure_file (
    "${PROJECT_SOURCE_DIR}/perfstubs/perfstubs_api/config.h.in"
    "${PROJECT_SOURCE_DIR}/perfstubs/perfstubs_api/config.h"
)
endif(NOT EXISTS "${PROJECT_SOURCE_DIR}/perfstubs/perfstubs_api/config.h")

# The easiest way to include perfstubs support is to include the one source file in
# the camtimers library.

add_library(timers
  perfstubs/perfstubs_api/timer.c
  f_wrappers.c
  GPTLget_memusage.c
  GPTLprint_memusage.c
  gptl.c
  gptl_papi.c
  GPTLutil.c
  perf_mod.F90
  perf_utils.F90)

target_compile_definitions(timers PRIVATE FORTRANUNDERSCORE HAVE_MPI LINUX PERFSTUBS_USE_TIMERS)
target_link_libraries(timers PUBLIC MPI::MPI_C global_settings)
if(OpenMP_FOUND)
    target_link_libraries(timers PRIVATE OpenMP::OpenMP_C)
    target_link_libraries(timers PRIVATE OpenMP::OpenMP_Fortran)
endif(OpenMP_FOUND)
target_include_directories(timers PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/perfstubs)

install(TARGETS timers)
install (FILES gptl.h DESTINATION include)
install (FILES ${CMAKE_BINARY_DIR}/perf_mod.mod DESTINATION include)
install (FILES ${CMAKE_BINARY_DIR}/perf_utils.mod DESTINATION include)

# provide under CAMTIMERS::CAMTIMERS name
add_library(CAMTIMERS::CAMTIMERS INTERFACE IMPORTED GLOBAL)
target_link_libraries(CAMTIMERS::CAMTIMERS INTERFACE timers)

