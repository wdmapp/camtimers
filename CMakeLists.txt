add_library(timers
  f_wrappers.c
  GPTLget_memusage.c
  GPTLprint_memusage.c
  gptl.c
  gptl_papi.c
  GPTLutil.c
  perf_mod.F90
  perf_utils.F90)

target_compile_definitions(timers PRIVATE FORTRANUNDERSCORE)
target_link_libraries(timers PUBLIC MPI::MPI_C global_settings)
target_include_directories(timers PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

# provide under CAMTIMERS::CAMTIMERS name
add_library(CAMTIMERS::CAMTIMERS INTERFACE IMPORTED GLOBAL)
target_link_libraries(CAMTIMERS::CAMTIMERS INTERFACE timers)
