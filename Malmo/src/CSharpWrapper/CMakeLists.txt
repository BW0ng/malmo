# ------------------------------------------------------------------------------------------------
# Copyright (c) 2016 Microsoft Corporation
# 
# Permission is hereby granted, free of charge, to any person obtaining a copy of this software and
# associated documentation files (the "Software"), to deal in the Software without restriction,
# including without limitation the rights to use, copy, modify, merge, publish, distribute,
# sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
# 
# The above copyright notice and this permission notice shall be included in all copies or
# substantial portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT
# NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
# NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
# DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
# ------------------------------------------------------------------------------------------------

include( ${SWIG_USE_FILE} )
include( ${CSHARP_USE_FILE} )

execute_process(
    COMMAND ${GIT_EXECUTABLE} rev-parse --abbrev-ref HEAD
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_BRANCH
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Get the latest abbreviated commit hash of the working branch
execute_process(
    COMMAND ${GIT_EXECUTABLE} log -1 --format=%h
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_COMMIT_HASH
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

set_source_files_properties ( MalmoNETNative.i PROPERTIES CPLUSPLUS ON )

# CSharp version requirements: http://msdn.microsoft.com/en-us/library/system.reflection.assemblyversionattribute.aspx
# major.minor[.build[.revision]] where all components are 16-bit unsigned integers

set(MALMONET_VERSION_CSHARP_AssemblyVersion "${MALMO_VERSION_MAJOR}.${MALMO_VERSION_MINOR}.${GIT_BRANCH}.${GIT_COMMIT_HASH}")

  # Make sure the nested directory structure exists
set(CSHARP_SOURCE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/swig CACHE INTERNAL "")
file(MAKE_DIRECTORY ${CSHARP_SOURCE_DIRECTORY})

  # Create swig target
set(CMAKE_SWIG_OUTDIR ${CSHARP_SOURCE_DIRECTORY})

set(CMAKE_SWIG_FLAGS -I${CMAKE_CURRENT_SOURCE_DIR} -namespace \"Microsoft.Research.Malmo\" ${CMAKE_SWIG_GLOBAL_FLAGS} ${CMAKE_SWIG_FLAGS})
if( ALE_FOUND )
  set (CMAKE_SWIG_FLAGS ${CMAKE_SWIG_FLAGS} "-DWRAP_ALE" )
endif()

SET_SOURCE_FILES_PROPERTIES(MalmoNETNative.i PROPERTIES SWIG_FLAGS "-includeall")
SWIG_ADD_LIBRARY(MalmoNETNative LANGUAGE csharp SOURCES MalmoNETNative.i )
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})
SWIG_LINK_LIBRARIES(${SWIG_MODULE_MalmoNETNative_REAL_NAME} Malmo )

if( UNIX )
  set_target_properties(${SWIG_MODULE_MalmoNETNative_REAL_NAME} PROPERTIES SUFFIX ".so")
  set( MALMO_NET_NATIVE libMalmoNETNative.so CACHE INTERNAL "The MalmoNETNative built library" )
else()
  set_target_properties(${SWIG_MODULE_MalmoNETNative_REAL_NAME} PROPERTIES SUFFIX ".dll")
  set( MALMO_NET_NATIVE MalmoNETNative.dll CACHE INTERNAL "The MalmoNETNative built library" )
endif()

# Configure AssemblyInfo.cs
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/AssemblyInfo.cs.in
    ${CSHARP_SOURCE_DIRECTORY}/AssemblyInfo.cs
    @ONLY
)

# Add managed wrapper
csharp_add_library(
    MalmoNET
    ${CSHARP_SOURCE_DIRECTORY}/*.cs
)
add_dependencies(MalmoNET ${SWIG_MODULE_MalmoNETNative_REAL_NAME})

install( TARGETS ${SWIG_MODULE_MalmoNETNative_REAL_NAME} DESTINATION CSharp_Examples )
install( TARGETS ${SWIG_MODULE_MalmoNETNative_REAL_NAME} DESTINATION HumanAction )
IF( WIN32 )
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/$<CONFIG>/MalmoNET.dll DESTINATION CSharp_Examples )
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/$<CONFIG>/MalmoNET.dll DESTINATION HumanAction )
ELSE( UNIX )
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/MalmoNET.dll DESTINATION CSharp_Examples )
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/MalmoNET.dll DESTINATION HumanAction )
ENDIF( WIN32 )
