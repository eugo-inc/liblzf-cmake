# MARK: - 1. Preface
cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
project(liblzf VERSION 3.6 DESCRIPTION "A very small data compression library" LANGUAGES C)


# MARK: - 2. CMake Includes
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)


# MARK: - 3. Libraries
set(LIBRARY_NAME lzf)

# Extract API version from lzf.h
file(READ src/include/lzf.h lzf_content)
if(lzf_content MATCHES ".*#define LZF_VERSION (0x[0-9A-Fa-f]+).*")
    # CMake 3.13+ is needed to evaluate 0x hexadecimal input values
    math(EXPR API_VERSION_MAJOR "${CMAKE_MATCH_1} >> 8")
    math(EXPR API_VERSION_MINOR "${CMAKE_MATCH_1} & 255")
    set(API_VERSION "${API_VERSION_MAJOR}.${API_VERSION_MINOR}")
    message(STATUS "liblzf API ${API_VERSION}")
else()
    message(FATAL_ERROR "cannot extract API version from lzf.h")
endif()

add_library(
    ${PROJECT_NAME}
    # Public headers
    src/include/lzf.h
    # Private headers
    src/lib/lzfP.h
    # Sources
    src/lib/lzf_c.c
    src/lib/lzf_d.c
)
target_include_directories(
    ${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
set_target_properties(
    ${PROJECT_NAME} PROPERTIES
    VERSION ${API_VERSION}
    SOVERSION ${API_VERSION_MAJOR}
    OUTPUT_NAME ${LIBRARY_NAME}
)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

if(CMAKE_C_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "Clang")
    target_compile_options(${PROJECT_NAME} PRIVATE "-funroll-all-loops")
endif()

if(MINGW)
    target_compile_definitions(${PROJECT_NAME} PRIVATE "_int64=long long")
endif()


# MARK: - 4. Package Installation
install(
    TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}-targets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(FILES src/include/lzf.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})


# MARK: - 5. Configuration Files Creation
# MARK: - 5.1. CMake config files
install(EXPORT ${PROJECT_NAME}-targets NAMESPACE ${PROJECT_NAME}:: DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
export(EXPORT ${PROJECT_NAME}-targets NAMESPACE ${PROJECT_NAME}::)

configure_package_config_file(
    ${PROJECT_NAME}-config.cmake.in ${PROJECT_NAME}-config.cmake
    PATH_VARS CMAKE_INSTALL_INCLUDEDIR
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

write_basic_package_version_file(${PROJECT_NAME}-config-version.cmake COMPATIBILITY SameMajorVersion)

install(
    FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

# MARK: - 5.3. pkg_config files
configure_file(${PROJECT_NAME}.pc.in ${PROJECT_NAME}.pc @ONLY)
install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)
