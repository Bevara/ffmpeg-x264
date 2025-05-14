cmake_minimum_required(VERSION 3.7)
project(Filters)

include(filters.cmake)

add_definitions(-fpic)

SET(FF_SRC_x264
        ${CMAKE_CURRENT_SOURCE_DIR}/ff_rescale-hevc.c
        ${CMAKE_CURRENT_SOURCE_DIR}/ff_enc-x264.c
        ${CMAKE_CURRENT_SOURCE_DIR}/ff_mx-x264.c
        ${CMAKE_CURRENT_SOURCE_DIR}/ff_common.c
)

SET(FF_LIB_x264
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/libavcodec.a
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/libavutil.a
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/libavformat.a
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/libavfilter.a
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/libswscale.a
)

SET(FF_INCLUDE_x264
        ${CMAKE_CURRENT_SOURCE_DIR}/include/ffmpeg
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

add_filter(ffmpeg-x264
        "${FF_SRC_x264}"
        "${FF_LIB_x264}"
        []
        GPAC_HAS_FFMPEG
        "${FF_INCLUDE_x264}"
        ""
        "1")
