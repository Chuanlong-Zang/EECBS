cmake_minimum_required(VERSION 3.16)
project(eecbs LANGUAGES CXX)

# Build the CLI driver too?
option(EECBS_BUILD_DRIVER "Build the cbs command-line driver" ON)

# --- Sources ------------------------------------------------------------------
file(GLOB EECBS_ALL_SOURCES CONFIGURE_DEPENDS src/*.cpp)
set(EECBS_DRIVER_SRC src/driver.cpp)
list(REMOVE_ITEM EECBS_ALL_SOURCES ${EECBS_DRIVER_SRC})

# --- Core library -------------------------------------------------------------
add_library(eecbs_core STATIC ${EECBS_ALL_SOURCES})
add_library(eecbs::core ALIAS eecbs_core)

target_include_directories(eecbs_core
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc>
        $<INSTALL_INTERFACE:include>
)

# Prefer features instead of forcing global flags
target_compile_features(eecbs_core PUBLIC cxx_std_17)
set_target_properties(eecbs_core PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Most core code only needs header-only Boost (tokenizer, etc.).
find_package(Boost REQUIRED)  # header-only OK
target_link_libraries(eecbs_core PUBLIC Boost::boost)

# If your core sources use Boost.Filesystem (rare), uncomment:
# find_package(Boost REQUIRED COMPONENTS filesystem)
# target_link_libraries(eecbs_core PUBLIC Boost::filesystem)

# --- Optional CLI driver ------------------------------------------------------
if(EECBS_BUILD_DRIVER)
    find_package(Boost REQUIRED COMPONENTS program_options filesystem)
    add_executable(eecbs ${EECBS_DRIVER_SRC})
    target_link_libraries(eecbs PRIVATE eecbs_core Boost::program_options Boost::filesystem)
    target_compile_features(eecbs PRIVATE cxx_std_17)
endif()