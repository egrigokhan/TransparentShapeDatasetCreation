#
# Copyright (c) 2016, NVIDIA CORPORATION. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#

set(sources
  ${SAMPLES_INCLUDE_DIR}/commonStructs.h
  ${SAMPLES_INCLUDE_DIR}/helpers.h
  ${SAMPLES_INCLUDE_DIR}/intersection_refinement.h
  ${SAMPLES_INCLUDE_DIR}/random.h
  tinyplyloader/tinyply.cpp
  tinyplyloader/plyLoader.cpp

  tinyobjloader/objLoader.cpp

  tinyxml/tinystr.cpp
  tinyxml/tinyxml.cpp
  tinyxml/tinyxmlerror.cpp
  tinyxml/tinyxmlparser.cpp

  Camera.cpp
  Camera.h
  OptiXMesh.cpp
  OptiXMesh.h
  ${CMAKE_CURRENT_BINARY_DIR}/../sampleConfig.h
  sutil.cpp
  sutil.h
  sutilapi.h
  )




include_directories(${CMAKE_CURRENT_SOURCE_DIR})
# For commonStructs.h, etc
include_directories(${SAMPLES_INCLUDE_DIR})

# Compile the cuda files to ptx.  Note that this will ignore all of the non CUDA
# files.
CUDA_COMPILE_PTX(ptx_files ${sources})

# Make the library.
set(sutil_target "sutil_sdk")
add_library(${sutil_target} ${sources} ${ptx_files})

# Use gcc rather than g++ to link if we are linking statically against libgcc_s
# and libstdc++
if(USING_GNU_C OR USING_GNU_CXX)
  if(GCC_LIBSTDCPP_HACK)
    set_target_properties(${sutil_target} PROPERTIES LINKER_LANGUAGE "C")
    target_link_libraries(${sutil_target} LINK_PRIVATE ${STATIC_LIBSTDCPP})
  endif()
endif()

# variable will be empty.
if(WIN32)
  target_link_libraries(${sutil_target} winmm.lib)
endif()


if(RELEASE_INSTALL_BINARY_SAMPLES AND NOT RELEASE_STATIC_BUILD)
  # If performing a release install, we want to use rpath for our install name.
  # The executables' rpaths will then be set to @executable_path so we can invoke
  # the samples from an arbitrary location and it will still find this library.
  set_target_properties(${sutil_target} PROPERTIES
    INSTALL_NAME_DIR "@rpath"
    BUILD_WITH_INSTALL_RPATH ON
    )
  install(TARGETS ${sutil_target}
    RUNTIME DESTINATION ${SDK_BINARY_INSTALL_DIR}
    LIBRARY DESTINATION ${SDK_BINARY_INSTALL_DIR}
    )
endif()

# Make the list of sources available to the parent directory for installation needs.
set(sutil_sources "${sources}" PARENT_SCOPE)
