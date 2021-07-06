# Copyright (c) 2017-2021, Ruslan Baratov, Rahul Sheth
# All rights reserved.

cmake_minimum_required(VERSION 3.2)
project(vurtun VERSION 0.0.1)

add_library(lib STATIC)

# Installation (https://github.com/forexample/package-example) {

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use:
#   * 'TARGETS_EXPORT_NAME'
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

function(GenerateHeader header_name defines)
    file(READ "${header_name}" HEADER_STRING)
    get_filename_component(header_prefix ${header_name} NAME_WE)
    set(GENERATED_HEADER "${generated_dir}/${header_prefix}.h")
    set(HEADER_DEFS "")
    set(HEADER_UNDEFS "")
    foreach(define IN LISTS defines)
        string(APPEND HEADER_DEFS "#define ${define}\n")
        string(REGEX REPLACE "(.*)\ .*" "\\1" undefine "${define}")
        string(APPEND HEADER_UNDEFS "#undef ${undefine}\n")
    endforeach()
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/header.template.in" "${GENERATED_HEADER}")
    install(FILES "${GENERATED_HEADER}" DESTINATION "${include_install_dir}/${PROJECT_NAME}")

    list(LENGTH defines define_count)
    if(define_count GREATER 0)
        string(PREPEND HEADER_STRING "${HEADER_DEFS}")
        file(WRITE "${generated_dir}/${header_prefix}.c" "${HEADER_STRING}")
        target_sources(lib PRIVATE "${generated_dir}/${header_prefix}.c")
    endif()
endfunction()

GenerateHeader(json.h "JSON_IMPLEMENTATION;JSON_STATIC")
GenerateHeader(sched.h "SCHED_IMPLEMENTATION;SCHED_STATIC")
GenerateHeader(sdefl.h "SDEFL_IMPLEMENTATION")
GenerateHeader(sinfl.h "SINFL_IMPLEMENTATION")
GenerateHeader(web.h "WBY_IMPLEMENTATION;WBY_STATIC")

target_compile_definitions(lib PUBLIC VURTUN_LIB_USE_HUNTER)
set_target_properties(lib PROPERTIES OUTPUT_NAME vurtun-lib DEBUG_POSTFIX d)

install(
    TARGETS lib
    EXPORT "${TARGETS_EXPORT_NAME}"
    INCLUDES DESTINATION "${include_install_dir}/${PROJECT_NAME}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

# }
