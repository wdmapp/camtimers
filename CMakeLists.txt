
cmake_minimum_required(VERSION 3.12)

project(Camtimers)
ENABLE_LANGUAGE(C CXX Fortran)

find_package(MPI REQUIRED)
find_package(OpenMP REQUIRED)

# Get the PerfStubs source, and add it as a subdirectory

find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
# Update submodules as needed
    option(GIT_SUBMODULE "Check submodules during build" ON)
    if(GIT_SUBMODULE)
        message(STATUS "Submodule update")
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                        RESULT_VARIABLE GIT_SUBMOD_RESULT)
        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
            message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
        endif()
    endif()
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/perfstubs/CMakeLists.txt")
    message(FATAL_ERROR "The submodules were not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()

add_subdirectory(perfstubs)

# The easiest way to include perfstubs support is to include the one source file in 
# the camtimers library.  However, we also have to set a dependency and build the
# perfstubs library so it can generate the perfstubs_api/config.h file.

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

add_dependencies(timers perfstubs)

target_compile_definitions(timers PRIVATE FORTRANUNDERSCORE HAVE_MPI LINUX PERFSTUBS_USE_TIMERS)
target_link_libraries(timers PUBLIC MPI::MPI_C global_settings perfstubs)
target_link_libraries(timers PRIVATE OpenMP::OpenMP_C)
target_link_libraries(timers PRIVATE OpenMP::OpenMP_Fortran)
target_include_directories(timers PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

install(TARGETS timers)
install (FILES gptl.h DESTINATION include)
install (FILES ${CMAKE_BINARY_DIR}/perf_mod.mod DESTINATION include)
install (FILES ${CMAKE_BINARY_DIR}/perf_utils.mod DESTINATION include)

# provide under CAMTIMERS::CAMTIMERS name
add_library(CAMTIMERS::CAMTIMERS INTERFACE IMPORTED GLOBAL)
target_link_libraries(CAMTIMERS::CAMTIMERS INTERFACE timers)

