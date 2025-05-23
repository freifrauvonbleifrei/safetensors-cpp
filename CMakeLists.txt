cmake_minimum_required(VERSION 3.16)

project(SAFETENSORS_CPP CXX C)

set(BUILD_TARGET_C "c-safetensors")

option(SAFETENSORS_CPP_BUILD_C_API "Build C API?" ON)
option(SAFETENSORS_CPP_BUILD_EXAMPLES "Build examples" ON)

# Disable C++ exception by default.
option(SAFETENSORS_CPP_CXX_EXCEPTIONS "Enable C++ exception(disable by default)" OFF)

set(SAFETENSORS_CPP_SOURCES
  safetensors.cc)

# compile with c++ for libsafetensors-c.a
set(SAFETENSORS_C_SOURCES
  safetensors-c.cc)

add_library(safetensors_cpp ${SAFETENSORS_CPP_SOURCES})

if (SAFETENSORS_CPP_BUILD_C_API)
  add_library(safetensors_c ${SAFETENSORS_C_SOURCES})
endif()

if(NOT SAFETENSORS_CPP_CXX_EXCEPTIONS)
  if(MSVC)
    # TODO: disable exception reliably
    #target_compile_options(safetensors_cpp PUBLIC /EHs-c-)
  else()
    target_compile_options(safetensors_cpp PUBLIC -fno-exceptions)
  endif()
endif()


if (SAFETENSORS_CPP_BUILD_EXAMPLES)
  add_executable(example example.cc)
  target_compile_definitions(example PRIVATE "SAFETENSORS_CPP_NO_IMPLEMENTATION")
  target_link_libraries(example safetensors_cpp)

  add_executable(serialize_example serialize-example.cc)
  target_compile_definitions(serialize_example PRIVATE "SAFETENSORS_CPP_NO_IMPLEMENTATION")
  target_link_libraries(serialize_example safetensors_cpp)

  if (SAFETENSORS_CPP_BUILD_C_API)
    add_executable(example-c example-c.c)
    target_compile_definitions(example-c PRIVATE "SAFETENSORS_C_NO_IMPLEMENTATION")
    target_link_libraries(example-c safetensors_c)
  endif ()
endif ()

# one library to collect all potential targets
add_library(safetensors-cpp INTERFACE)

# add dependecy to sources 
# so that everything that links to safetensors-cpp without install
# will also get the headers
target_include_directories(safetensors_cpp INTERFACE  
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

install(
  TARGETS safetensors_cpp EXPORT safetensors-cpp
  LIBRARY DESTINATION lib
  INCLUDES DESTINATION include
)
set(SAFETENSORS_INSTALL_HEADERS
  safetensors.hh
)
if (SAFETENSORS_CPP_BUILD_C_API)
  list(APPEND SAFETENSORS_INSTALL_HEADERS
    safetensors-c.h
  )
endif (SAFETENSORS_CPP_BUILD_C_API)

install(
  FILES $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>/${SAFETENSORS_INSTALL_HEADERS}
  DESTINATION include
)
# export full library interface
install(
  EXPORT safetensors-cpp
  FILE safetensors-cpp-config.cmake
  NAMESPACE safetensors-cpp::
  DESTINATION lib/cmake/
)
