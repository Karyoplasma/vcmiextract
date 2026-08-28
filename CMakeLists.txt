cmake_minimum_required(VERSION 3.15)

project(vcmiextract)

option(VCMIEXTRACT_WIN7 "Build with Windows 7 API compatibility" OFF)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(ZLIB REQUIRED)
find_package(PNG REQUIRED)
find_package(LZMA QUIET)

if (NOT LZMA_FOUND)
    find_library(LZMA_LIBRARY NAMES lzma liblzma)
    find_path(LZMA_INCLUDE_DIR NAMES lzma.h)

    if (LZMA_LIBRARY AND LZMA_INCLUDE_DIR)
		set(LZMA_FOUND TRUE)
        set(LZMA_LIBRARIES ${LZMA_LIBRARY})
        set(LZMA_INCLUDE_DIRS ${LZMA_INCLUDE_DIR})
    else()
        set(LZMA_FOUND FALSE)
    endif()
endif()

if (MSVC)
    add_compile_options(/W4)
else()
    add_compile_options(-Wall -Wextra -pedantic -Werror -Wno-error=unused-variable)
endif()

set(extract_SRCS
    src/hota_file_names.h
    src/file_format_png.cpp
    src/file_format_png.h
    src/file_format_dds.cpp
    src/file_format_dds.h
    src/memory_file.h
    src/vcmiextract.cpp
    src/vcmiextract.h
    src/vcmiextract_archive.cpp
    src/vcmiextract_image.cpp
    src/vcmiextract_hd.cpp
    src/vcmiextract_zlib.cpp
    src/vcmiextract_lzma.cpp
)

add_executable(vcmiextract ${extract_SRCS})

# Static MSVC runtime.
set_property(
    TARGET vcmiextract
    PROPERTY MSVC_RUNTIME_LIBRARY
    "MultiThreaded$<$<CONFIG:Debug>:Debug>"
)

if (WIN32 AND VCMIEXTRACT_WIN7)
    # Windows 7 / Server 2008 R2.
    add_compile_definitions(
        WINVER=0x0601
        _WIN32_WINNT=0x0601
    )
endif()

target_include_directories(
    vcmiextract
    PRIVATE
        ${LZMA_INCLUDE_DIRS}
)

target_link_libraries(
    vcmiextract
    PRIVATE
        ${LZMA_LIBRARIES}
        ZLIB::ZLIB
        PNG::PNG
)

install(TARGETS vcmiextract RUNTIME DESTINATION .)

set(CPACK_PACKAGE_NAME "vcmiextract")
set(CPACK_PACKAGE_VERSION "1.0")

include(CPack)