cmake_minimum_required(VERSION 3.16)

# set a default CXX standard for the tools and targets that do not specify them.
# If commented, the latest supported standard for your compiler is automatically set.
# set(CMAKE_CXX_STANDARD 20)

# Add project_options from https://github.com/aminya/project_options
# Change the version in the following URL to update the package (watch the releases of the repository for future updates)
include(FetchContent)
FetchContent_Declare(_project_options
	GIT_REPOSITORY "https://github.com/mous16/project_options.git"
	GIT_TAG        origin/main)
FetchContent_MakeAvailable(_project_options)
include(${_project_options_SOURCE_DIR}/Index.cmake)

# Define the features of the project
include("./Features.cmake")

# enable cross-compiling: - should be called before run_vcpkg()
if(ENABLE_CROSS_COMPILING)
  enable_cross_compiler()
endif()

# Set the project name and language
project(
  cpp_vcpkg_project
  VERSION 0.0.0
  DESCRIPTION ""
  HOMEPAGE_URL "https://github.com/cpp-best-practices/cpp_vcpkg_project"
  LANGUAGES CXX C)

setup_project_options("my")

if(FEATURE_TESTS)
  enable_testing()

  add_library(catch2_test_common INTERFACE)
  target_find_dependencies(catch2_test_common INTERFACE_CONFIG Catch2)
  target_link_libraries(catch2_test_common INTERFACE Catch2::Catch2)
  # generate a main function for the test executable
  target_compile_definitions(catch2_test_common
    INTERFACE CATCH_CONFIG_MAIN
              DO_NOT_USE_WMAIN)
  include(Catch)
endif()

add_subdirectory(./my_exe)
add_subdirectory(./my_lib)
add_subdirectory(./my_header_lib)

# Package the project
package_project(
  TARGETS
  my_exe
  my_lib
  my_header_lib
  my_project_options
  my_project_warnings)

if(FEATURE_FUZZ_TESTS)
  add_subdirectory(fuzz_test)
endif()

# set the startup project for the "play" button in MSVC
set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT my_exe)

# Experience shows that explicit package naming can help make it easier to sort
# out potential ABI related issues before they start, while helping you
# track a build to a specific GIT SHA
set(CPACK_PACKAGE_FILE_NAME
    "${CMAKE_PROJECT_NAME}-${CMAKE_PROJECT_VERSION}-${GIT_SHORT_SHA}-${CMAKE_SYSTEM_NAME}-${CMAKE_BUILD_TYPE}-${CMAKE_CXX_COMPILER_ID}-${CMAKE_CXX_COMPILER_VERSION}"
)

include(CPack)
