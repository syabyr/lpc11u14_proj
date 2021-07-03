# Base LPC11U14 CMake file
#
# This can be used as is as a project base, or by adding the efm32-base
# repository as a submodule to another project, copying this CMakeLists file
# to the top level directory, and updating the BASE_LOCATION variable to reflect this
# change
#
## Copyright (c) 2016 Ryan Kurte
# This file is covered under the MIT license available at: https://opensource.org/licenses/MIT

###### Project Environment #####

# Set minimum CMake version
cmake_minimum_required(VERSION 3.1.0)

#set(COMPILER_PREFIX /Users/mybays/develop/efr32/toolchains/gnu_arm/7.2_2017q4/bin/)

if(NOT COMPILER_PREFIX)
    set(CMAKE_C_COMPILER arm-none-eabi-gcc)
    set(CMAKE_AR arm-none-eabi-ar)
else ()
    set(CMAKE_C_${COMPILER_PREFIX}/COMPILER arm-none-eabi-gcc)
    set(CMAKE_AR ${COMPILER_PREFIX}/arm-none-eabi-ar)
endif ()


# Optional verbose mode, uncomment or pass in -DCMAKE_VERBOSE_MAKEFILE=ON
set(CMAKE_VERBOSE_MAKEFILE ON)

set(BASE_LOCATION ../)
message ( STATUS "BASE_LOCATION = ${BASE_LOCATION}.")


set(DEVICE "lpc11u14")

# Set the compiler (must be prior to project setup)
include(../toolchain/arm-gcc.cmake)

##### Project Setup #####

# Configure project and languages
project(blink C CXX ASM)


set(CMAKE_BUILD_TYPE RELEASE)


# Set build
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE DEBUG)
endif ()



include(../toolchain/lpc-base.cmake)


if(CMAKE_BUILD_TYPE STREQUAL "RELEASE")
    set(LINKER_SCRIPT "${BASE_LOCATION}/../toolchain/nxp_lpcxpresso_11u14_Release.ld")
else()
    set(LINKER_SCRIPT "${BASE_LOCATION}/../toolchain/nxp_lpcxpresso_11u14_Debug.ld")
endif ()

set(LINKER_SCRIPT "${BASE_LOCATION}/../toolchain/nxp_lpcxpresso_11u14_Debug.ld")

set(CMAKE_EXE_LINKER_FLAGS "-nostdlib -T${LINKER_SCRIPT} -Wl,-Map=${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.map -Xlinker --gc-sections -mcpu=cortex-m0 -mthumb -Wl,-v -u _printf_float")


if(CMAKE_BUILD_TYPE STREQUAL "RELEASE")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L../../nxp_lpcxpresso_11u14_board_lib/Release -L../../lpc_chip_11uxx_lib/Release")
else()
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L../../nxp_lpcxpresso_11u14_board_lib/Debug -L../../lpc_chip_11uxx_lib/Debug")
endif ()


# Add project headers
include_directories(./example/inc)
include_directories(${BASE_LOCATION}/lpc_chip_11uxx_lib/inc)
include_directories(${BASE_LOCATION}/nxp_lpcxpresso_11u14_board_lib/inc)

AUX_SOURCE_DIRECTORY(./ SOURCE_FILES)
AUX_SOURCE_DIRECTORY(./example/src SOURCE_FILES)


# Generate executable and link
add_executable(${PROJECT_NAME}.axf
	${SOURCE_FILES})


target_link_libraries(${PROJECT_NAME}.axf ${LIBS} nxp_lpcxpresso_11u14_board_lib lpc_chip_11uxx_lib)


include(../toolchain/post-build.cmake)


include(../toolchain/isp.cmake)