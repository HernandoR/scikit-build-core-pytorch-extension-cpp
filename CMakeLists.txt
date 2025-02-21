# Set the minimum required version of CMake
cmake_minimum_required(VERSION 3.10)

# Set the project name, which should match the library name in the Python setup
project(extension_cpp LANGUAGES CXX)



# Set the C++ standard to C++17
if (NOT DEFINED CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()


# Find Python development libraries and headers
find_package(
  Python
  COMPONENTS Interpreter Development.Module
  REQUIRED
  )

# Find the Torch library and its includes
find_package(Torch REQUIRED)

# Get the value of the DEBUG environment variable
if(DEFINED ENV{DEBUG})
    set(DEBUG_MODE $ENV{DEBUG})
else()
    set(DEBUG_MODE 0)
endif()
message(STATUS "CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
# check CMAKE_CUDA_COMPILER is set
if(DEFINED CMAKE_CUDA_COMPILER)
    enable_language(CUDA)
    if(NOT DEFINED CMAKE_CUDA_STANDARD)
        set(CMAKE_CUDA_STANDARD 11)
        set(CMAKE_CUDA_STANDARD_REQUIRED ON)
    endif()
    set(USE_CUDA 1)
    set(CUDA_FOUND 1)
    define_property(TARGET PROPERTY CUDA_SEPARABLE_COMPILATION BRIEF_DOCS "Enable or disable separable compilation for CUDA sources" FULL_DOCS "Enable or disable separable compilation for CUDA sources" )
    message(STATUS "CUDA found: Compiler: ${CMAKE_CUDA_COMPILER}")
else()
    set(USE_CUDA 0)
endif()
# Check if CUDA is available
set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS})
# if(${USE_CUDA} EQUAL 1)
#     find_package(CUDA REQUIRED)
#     set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} -O3 --allow-unsupported-compiler)
#     # set architecture
#     if(NOT DEFINED CUDA_ARCH)
#         set(CUDA_ARCH "native")
#     endif()
#     if(${DEBUG_MODE} EQUAL 1)
#         set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} -O0 -g)
#     endif()
#     set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -O3 -fdiagnostics-color=always -DPy_LIMITED_API=0x03090000)
#     if(${DEBUG_MODE} EQUAL 1)
#         set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -O0 -g)
#     endif()
#     message(STATUS "CUDA found: Headers: ${CUDA_INCLUDE_DIRS}")
#     message(STATUS "CUDA found: Libraries: ${CUDA_LIBRARIES}")
# else()
#     set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -O3 -fdiagnostics-color=always -DPy_LIMITED_API=0x03090000)
#     if(${DEBUG_MODE} EQUAL 1)
#         set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -O0 -g)
#     endif()
#     message(STATUS "CUDA not found")
# endif()


# Set the directory where the source files are located
set(EXTENSION_DIR ${PROJECT_SOURCE_DIR}/extension_cpp/csrc)

# Find all C++ source files
file(GLOB_RECURSE SOURCES ${EXTENSION_DIR}/*.cpp)
message(STATUS "EXTENSION_DIR: ${EXTENSION_DIR}")
# If CUDA is enabled, find CUDA source files
if(${USE_CUDA} EQUAL 1)
    set(CUDA_EXTENSION_DIR ${EXTENSION_DIR}/cuda)
    file(GLOB_RECURSE CUDA_SOURCES ${CUDA_EXTENSION_DIR}/*.cu)
    list(APPEND SOURCES ${CUDA_SOURCES})
endif()

message(STATUS "Sources: ${SOURCES}")
# Set the name of the Python extension module
set(EXTENSION_MODULE_NAME _C)


# python_add_library(example MODULE example.c WITH_SOABI)
# # Create the Python extension module
python_add_library(${EXTENSION_MODULE_NAME} MODULE ${SOURCES} WITH_SOABI)
# add_library(${EXTENSION_MODULE_NAME} MODULE ${SOURCES})
# target_compile_options(${EXTENSION_MODULE_NAME} PRIVATE -Wall -Wextra -Wpedantic -Werror)

# Link against the Torch library
target_link_libraries(${EXTENSION_MODULE_NAME} PRIVATE ${TORCH_LIBRARIES})

# Set the appropriate Python include directories
target_include_directories(${EXTENSION_MODULE_NAME} PRIVATE ${Python_INCLUDE_DIRS})
target_include_directories(${EXTENSION_MODULE_NAME} PRIVATE ${TORCH_INCLUDE_DIRS})

# If CUDA is enabled, set the CUDA include directories
if(${USE_CUDA} EQUAL 1)
    target_include_directories(${EXTENSION_MODULE_NAME} PRIVATE ${CUDA_INCLUDE_DIRS})
endif()
# set build directory
set_target_properties(${EXTENSION_MODULE_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/extension_cpp)

# Set the output directory for the shared library to be consistent with Python's expectations
set_target_properties(${EXTENSION_MODULE_NAME} PROPERTIES PREFIX "")
set_target_properties(${EXTENSION_MODULE_NAME} PROPERTIES SUFFIX ".so")
set_target_properties(${EXTENSION_MODULE_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR})


# install the library
install(TARGETS ${EXTENSION_MODULE_NAME} DESTINATION ${PROJECT_SOURCE_DIR}/extension_cpp)
