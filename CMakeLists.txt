cmake_minimum_required(VERSION 3.20)

#set(CMAKE_C_COMPILER_WORKS TRUE)
#set(CMAKE_C_STANDARD 99)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR GD32VF103)
#设置工程名字,PROJECTZ指令必须在版本之后，其它指令之前

#设置RISCV-GCC工具链
set(TOOLCHAIN_PREFIX riscv-none-elf-)
set(TOOLCHAIN_DIR /home/fab/sharefile/tools/riscv-none-elf-gcc/xpack-riscv-none-elf-gcc-12.2.0-1/bin/) 
#set(TOOLCHAIN_PREFIX riscv64-unknown-elf-)
#set(TOOLCHAIN_DIR /usr/bin)
set(CMAKE_C_COMPILER ${TOOLCHAIN_DIR}${TOOLCHAIN_PREFIX}gcc)
set(CMAKE_ASM_COMPILER ${CMAKE_C_COMPILER}) 
set(CMAKE_AR ${TOOLCHAIN_DIR}${TOOLCHAIN_PREFIX}ar) 
set(CMAKE_OBJCOPY ${TOOLCHAIN_DIR}${TOOLCHAIN_PREFIX}objcopy) 
set(CMAKE_OBJDUMP ${TOOLCHAIN_DIR}${TOOLCHAIN_PREFIX}objdump)
set(CMAKE_SIZE ${TOOLCHAIN_DIR}${TOOLCHAIN_PREFIX}size)

#set(CMAKE_CXX_COMPILER ${TOOLCHAIN_PREFIX}g++)
project(LED_GD32VF103 C ASM)
message(STATUS "Start to deal with the project: ${PROJECT_NAME}")

#设置编译参数
set(MCU -march=rv32imac_zicsr -mabi=ilp32 -mcmodel=medlow -msmall-data-limit=8 -fmessage-length=0 -fsigned-char)
add_compile_options(${MCU} -std=gnu99 -Wall -Werror -Wall -fdata-sections -ffunction-sections)
#add_compile_options(-march=rv32imac -mabi=ilp32 -mcmodel=medlow -msmall-data-limit=8 -fmessage-length=0 -fsigned-char)
#GCC编译flags
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_C_FLAGS_DEBUG "-g -O2")
#set(CMAKE_C_FLAGS_RELEASE "-O3")
SET(ASM_OPTIONS "-x assembler-with-cpp")
#SET(CMAKE_ASM_FLAGS "${MCU} ${ASM_OPTIONS}" -Og -Wall -fdata-sections -ffunction-sections )
SET(CMAKE_ASM_FLAGS "${ASM_OPTIONS}")

#target_include_directories(${PROJECT_NAME} PRIVATE "${PROJECT_SOURCE_DIR}/startup")
include_directories(
    ${PROJECT_SOURCE_DIR}/devices
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_SOURCE_DIR}/RISCV/drivers
    ${PROJECT_SOURCE_DIR}/GD32VF103_SDK_V1.0.0/Include
    ${PROJECT_SOURCE_DIR}/LCD_common
)


#creat library for other block
file(GLOB_RECURSE Driver_SDK 
    #driver部分
    # ${PROJECT_SOURCE_DIR}/drivers/common/*.c
     ${PROJECT_SOURCE_DIR}/GD32VF103_SDK_V1.0.0/Source/*.c

)

#Build a static library

add_library(drvlib_gd32vf103 STATIC
   ${Driver_SDK}
   #{其它目录}
)

target_include_directories(drvlib_gd32vf103 PRIVATE
    ${PROJECT_SOURCE_DIR}/devices
    #${PROJECT_SOURCE_DIR}/src
    ${PROJECT_SOURCE_DIR}/GD32VF103_SDK_V1.0.0/Include
    #${PROJECT_SOURCE_DIR}/RISCV/drivers
    #${PROJECT_SOURCE_DIR}/RISCV/stubs
    #${PROJECT_SOURCE_DIR}/drivers
    #${PROJECT_SOURCE_DIR}/CMSIS
    #${PROJECT_SOURCE_DIR}/Utilities
)


set(LINKER_SCRIPT ${PROJECT_SOURCE_DIR}/devices/GD32VF103_lds/GD32VF103xB.lds)
add_link_options(${MCU} -nostartfiles -specs=nano.specs -Xlinker -T${LINKER_SCRIPT})
#add_link_options(-Wl,-Map, GD32VF103_LED_LIGHT.map --data-sections) 


#AUX_SOURCE_DIRECTORY(. DIR_SRCS)
# 源文件：寻找目录下所有的源文件，加到执行目标
file(GLOB_RECURSE frdm_src 
    ${PROJECT_SOURCE_DIR}/src/*.c
    ${PROJECT_SOURCE_DIR}/devices/startup/entry.S
    ${PROJECT_SOURCE_DIR}/devices/startup/start.S
    ${PROJECT_SOURCE_DIR}/devices/startup/handlers.c
    ${PROJECT_SOURCE_DIR}/devices/startup/init.c
    ${PROJECT_SOURCE_DIR}/devices/system_gd32vf103.c
    ${PROJECT_SOURCE_DIR}/RISCV/drivers/*.c
    ${PROJECT_SOURCE_DIR}/LCD_common/*.c
)

#main file
set(EXECTABLE_NAME ${PROJECT_NAME})
add_executable(${EXECTABLE_NAME}.elf  
    ${frdm_src}
    #${其它}
)

#link extra libraries
target_link_libraries(${EXECTABLE_NAME}.elf
   drvlib_gd32vf103
)

target_include_directories(${EXECTABLE_NAME}.elf PRIVATE
    ${PROJECT_SOURCE_DIR}/devices
    ${PROJECT_SOURCE_DIR}/src
    #${PROJECT_SOURCE_DIR}/Utilities
)


set(ELF_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.elf)
set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)

# display the size of the output binary after a build is finished
add_custom_command(TARGET ${EXECTABLE_NAME}.elf POST_BUILD COMMAND ${CMAKE_SIZE} --format=berkeley ${EXECTABLE_NAME}.elf)
add_custom_target(${EXECTABLE_NAME}.hex ALL DEPENDS ${EXECTABLE_NAME}.elf COMMAND ${CMAKE_OBJCOPY} -Oihex ${EXECTABLE_NAME}.elf ${EXECTABLE_NAME}.hex)
add_custom_target(${EXECTABLE_NAME}.bin ALL DEPENDS ${EXECTABLE_NAME}.elf COMMAND ${CMAKE_OBJCOPY} -Obinary ${EXECTABLE_NAME}.elf ${EXECTABLE_NAME}.bin)
#add_custom_command(TARGET "${PROJECT_NAME}.elf" POST_BUILD
#    COMMAND ${CMAKE_OBJCOPY} -Obinary ${ELF_FILE} $drvlib_gd32f450
#    COMMAND ${CMAKE_OBJCOPY} -Oihex  ${ELF_FILE} ${HEX_FILE}
#    COMMENT "Building ${PROJECT_NAME}.bin and ${PROJECT_NAME}.hex"

#    COMMAND ${CMAKE_COMMAND} -E copy ${HEX_FILE} "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.hex"
#    COMMAND ${CMAKE_COMMAND} -E copy ${BIN_FILE} "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.bin"

#    COMMAND ${CMAKE_SIZE} --format=berkeley ${PROJECT_NAME}.elf ${PROJECT_NAME}.hex
#    COMMENT "Invoking: Cross ARM GNU Print Size"
#)