cmake_minimum_required(VERSION 3.1)

project(http-parser VERSION 2.8.0)

add_library(http_parser "http_parser.c" "http_parser.h")
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
  target_compile_definitions(http_parser PRIVATE HTTP_PARSER_STRICT=1)
else ()
  target_compile_definitions(http_parser PRIVATE HTTP_PARSER_STRICT=0)
endif ()

if (ENABLE_TESTING)
  enable_testing()
  message("building http-parser tests")
  add_executable(
    http_parser_tests

    "test.c"
  )

target_link_libraries(http_parser_tests PRIVATE http_parser)

endif()

# Installation (https://github.com/forexample/package-example)

# Layout. This works for all platforms:
#   * <prefix>/lib/cmake/<PROJECT-NAME>
#   * <prefix>/lib/
#   * <prefix>/include/
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
  "cmake/Config.cmake.in"
  "${project_config}"
  INSTALL_DESTINATION "${config_install_dir}"
)

# Targets:
#   * <prefix>/lib/libhttp_parser.a
#   * header location after install: <prefix>/include/http_parser.h
#   * headers can be included by C++ code `#include <http_parser.h>`
install(
  TARGETS http_parser
  EXPORT "${TARGETS_EXPORT_NAME}"
  LIBRARY DESTINATION "lib"
  ARCHIVE DESTINATION "lib"
  RUNTIME DESTINATION "bin"
  INCLUDES DESTINATION "${include_install_dir}"
)

# Headers:
#   * http_parser.h -> <prefix>/include/http_parser.h
install(
  DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
  DESTINATION "${include_install_dir}"
  FILES_MATCHING PATTERN "*.h"
)

# Config
#   * <prefix>/lib/cmake/http-parser/http-parserConfig.cmake
#   * <prefix>/lib/cmake/http-parser/http-parserConfigVersion.cmake
install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
)

# Config
#   * <prefix>/lib/cmake/http-parser/http-parserTargets.cmake
install(
  EXPORT "${TARGETS_EXPORT_NAME}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
)
