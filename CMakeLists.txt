cmake_minimum_required(VERSION 3.5)

# Setup project
# ------------------------------------------------------------
project(#[[project name]] VERSION 0.1.0 LANGUAGES C)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
add_definitions(-D__KERNEL__ -DMODULE)

set(SRCS #[[sources]])

# Workaround for clion intellisense
add_library(${PROJECT_NAME}_dummy EXCLUDE_FROM_ALL ${SRCS})

# Kernel headers: search and include
# ------------------------------------------------------------
execute_process(
        COMMAND uname -r
        OUTPUT_VARIABLE KERNEL_RELEASE
        OUTPUT_STRIP_TRAILING_WHITESPACE
)

find_path(
        KERNEL_HEADERS
        include/linux/user.h
        PATHS /usr/src/linux-headers-${KERNEL_RELEASE} /lib/modules/${KERNEL_RELEASE}/build
)

message(STATUS "Kernel release: ${KERNEL_RELEASE}")
message(STATUS "Kernel headers: ${KERNEL_HEADERS}")

include_directories(
        ${KERNEL_HEADERS}/include
)

# Create symlinks of sources in build directory
# ------------------------------------------------------------
set(SRCS_OBJ)

foreach (S IN LISTS SRCS)
    get_filename_component(S_NAME ${S} NAME_WLE)
    get_filename_component(S_DIR ${S} DIRECTORY)

    cmake_path(APPEND S_OBJ "${S_DIR}" "${S_NAME}.o")
    list(APPEND SRCS_OBJ "${S_OBJ}")
    unset(S_OBJ)

    if (S_DIR)
        execute_process(COMMAND mkdir -p ${S_DIR})
    endif ()

    execute_process(COMMAND ln -fs ${CMAKE_CURRENT_SOURCE_DIR}/${S} ${S})
endforeach ()

# Generate the Kbuild file
# ------------------------------------------------------------
string (REPLACE ";" " " SRCS_OBJ_STR "${SRCS_OBJ}")
FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/Kbuild
        "obj-m := ${PROJECT_NAME}.o\n"
        "${PROJECT_NAME}-objs := ${SRCS_OBJ_STR}")

# Kernel module build setup
# ------------------------------------------------------------
add_custom_command(OUTPUT ${PROJECT_NAME}.ko
        COMMAND make -C ${KERNEL_HEADERS} modules M=${CMAKE_CURRENT_BINARY_DIR} SRC=${CMAKE_CURRENT_SOURCE_DIR}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        DEPENDS ${SRCS} VERBATIM)

add_custom_target(${PROJECT_NAME} ALL DEPENDS ${PROJECT_NAME}.ko)
