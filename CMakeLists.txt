cmake_minimum_required(VERSION 3.16)

find_package(ros_industrial_cmake_boilerplate REQUIRED)
find_package(Python3 REQUIRED COMPONENTS Interpreter)

extract_package_metadata(pkg)
project(opentelemetry_vendor VERSION ${pkg_extracted_version} LANGUAGES CXX)

# Enable Python components
enable_language(NONE)

set(OPENTELEMETRY_CPP_VERSION "1.27.0")
set(OPENTELEMETRY_PYTHON_SITE_PACKAGES "lib/python${Python3_VERSION_MAJOR}.${Python3_VERSION_MINOR}/site-packages")
set(OPENTELEMETRY_PYTHON_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/${OPENTELEMETRY_PYTHON_SITE_PACKAGES}")
# Remove patch version for find_package
string(REGEX REPLACE "\\.[0-9]+$" "" OPENTELEMETRY_CPP_VERSION_MAJORMINOR ${OPENTELEMETRY_CPP_VERSION})

find_package(opentelemetry-cpp ${OPENTELEMETRY_CPP_VERSION_MAJORMINOR} QUIET CONFIG)

if(NOT opentelemetry-cpp_FOUND)
  message(STATUS "opentelemetry-cpp not found. Building from source.")

  include(ExternalProject)

  ExternalProject_Add(
    opentelemetry-cpp
    GIT_REPOSITORY https://github.com/open-telemetry/opentelemetry-cpp.git
    GIT_TAG v${OPENTELEMETRY_CPP_VERSION}
    SOURCE_DIR ${CMAKE_BINARY_DIR}/opentelemetry-cpp-src
    BINARY_DIR ${CMAKE_BINARY_DIR}/opentelemetry-cpp-build
    CMAKE_CACHE_ARGS
      -DCMAKE_BUILD_TYPE:STRING=Release
      -DCMAKE_INSTALL_PREFIX:STRING=${CMAKE_INSTALL_PREFIX}
      -DCMAKE_TOOLCHAIN_FILE:STRING=${CMAKE_TOOLCHAIN_FILE}
      -DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=ON
      -DBUILD_SHARED_LIBS:BOOL=ON
      -DWITH_STL:STRING=CXX17
      -DCMAKE_CXX_STANDARD:STRING=17
      -DCMAKE_CXX_STANDARD_REQUIRED:BOOL=ON
      -DCMAKE_CXX_EXTENSIONS:BOOL=OFF
      -DWITH_ABI_VERSION_2:BOOL=ON
      -DWITH_ABI_VERSION_1:BOOL=OFF
      -DWITH_OTLP_GRPC:BOOL=ON
      -DWITH_OTLP_RETRY_PREVIEW:BOOL=ON
      -DWITH_OTLP_GRPC_SSL_MTLS_PREVIEW:BOOL=ON
      -DWITH_OTLP_GRPC_CREDENTIAL_PREVIEW:BOOL=ON
      -DWITH_ASYNC_EXPORT_PREVIEW:BOOL=ON
      -DWITH_METRICS_EXEMPLAR_PREVIEW:BOOL=ON
      -DWITH_THREAD_INSTRUMENTATION_PREVIEW:BOOL=ON
      -DWITH_OTLP_FILE:BOOL=OFF
      -DWITH_OTLP_HTTP:BOOL=OFF
      -DWITH_EXAMPLES:BOOL=OFF
      -DWITH_BENCHMARK:BOOL=OFF
      -DBUILD_TESTING:BOOL=OFF
  )
endif()

include(ExternalProject)

ExternalProject_Add(
  opentelemetry-python
  DOWNLOAD_COMMAND ""
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND
    ${Python3_EXECUTABLE} -m pip install
      --upgrade
      --upgrade-strategy eager
      --disable-pip-version-check
      --no-warn-script-location
      --target ${OPENTELEMETRY_PYTHON_INSTALL_DIR}
      -r ${CMAKE_CURRENT_SOURCE_DIR}/python_requirements.txt
)

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/pythonpath.dsv.in
  ${CMAKE_CURRENT_BINARY_DIR}/pythonpath.dsv
  @ONLY
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/pythonpath.dsv DESTINATION share/${PROJECT_NAME}/hook)

install(FILES package.xml DESTINATION share/${PROJECT_NAME})