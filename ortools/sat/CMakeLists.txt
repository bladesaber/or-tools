# Copyright 2010-2022 Google LLC
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

file(GLOB _SRCS "*.h" "*.cc")
list(REMOVE_ITEM _SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/opb_reader.h
  ${CMAKE_CURRENT_SOURCE_DIR}/sat_cnf_reader.h
  ${CMAKE_CURRENT_SOURCE_DIR}/sat_runner.cc
)
set(NAME ${PROJECT_NAME}_sat)

# Will be merge in libortools.so
#add_library(${NAME} STATIC ${_SRCS})
add_library(${NAME} OBJECT ${_SRCS})
set_target_properties(${NAME} PROPERTIES
  POSITION_INDEPENDENT_CODE ON
  )
target_include_directories(${NAME} PRIVATE
  ${PROJECT_SOURCE_DIR}
  ${PROJECT_BINARY_DIR})
target_link_libraries(${NAME} PRIVATE
  absl::any
  absl::memory
  absl::synchronization
  absl::strings
  absl::str_format
  protobuf::libprotobuf
  $<$<BOOL:${USE_COINOR}>:Coin::Cbc>
  ${PROJECT_NAME}::proto)
#add_library(${PROJECT_NAME}::sat ALIAS ${NAME})

# Sat Runner
add_executable(sat_runner)
target_sources(sat_runner PRIVATE "sat_runner.cc")
target_include_directories(sat_runner PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_compile_features(sat_runner PRIVATE cxx_std_17)
target_link_libraries(sat_runner PRIVATE ${PROJECT_NAMESPACE}::ortools)

include(GNUInstallDirs)
if(APPLE)
  set_target_properties(sat_runner PROPERTIES INSTALL_RPATH
    "@loader_path/../${CMAKE_INSTALL_LIBDIR};@loader_path")
elseif(UNIX)
  cmake_path(RELATIVE_PATH CMAKE_INSTALL_FULL_LIBDIR
             BASE_DIRECTORY ${CMAKE_INSTALL_FULL_BINDIR}
             OUTPUT_VARIABLE libdir_relative_path)
  set_target_properties(sat_runner PROPERTIES
                        INSTALL_RPATH "$ORIGIN/${libdir_relative_path}")
endif()

install(TARGETS sat_runner)
