cmake_minimum_required(VERSION 2.4)

set(CMAKE_CXX_FLAGS "-g -O0 -Wall -Wno-deprecated -DDEBUG -DUNORDERED -fPIC")
include_directories(inc/LASlib inc/LASzip)

set(RUNTIME_OUTPUT_DIRECTORY bin)
set(EXECUTABLE_OUTPUT_PATH bin)

set(LIBRARY_OUTPUT_DIRECTORY lib)
set(LIBRARY_OUTPUT_PATH lib)

set(LAS_BIN_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

set(LASLIB_DIR ${CMAKE_CURRENT_SOURCE_DIR}/LASlib)
set(LASLIB_INCLUDE_DIR ${LASLIB_DIR}/inc)
set(LASLIB_SRC_DIR ${LASLIB_DIR}/src)

set(LASZIP_DIR ${CMAKE_CURRENT_SOURCE_DIR}/LASzip)
set(LASZIP_INCLUDE_DIR ${LASZIP_DIR}/src)
set(LASZIP_SRC_DIR ${LASZIP_DIR}/src)

set(LASLIB_SOURCE         ${LASLIB_SRC_DIR}/lasreader.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreader_las.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreader_bin.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreader_qfit.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreader_shp.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreader_asc.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreader_bil.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreader_dtm.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreader_txt.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreadermerged.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreaderbuffered.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasreaderpipeon.cpp)

LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswriter.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswriter_las.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswriter_bin.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswriter_qfit.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswriter_wrl.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswriter_txt.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswritercompatible.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswaveform13reader.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/laswaveform13writer.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasutility.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lasfilter.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/lastransform.cpp)
LIST(APPEND LASLIB_SOURCE ${LASLIB_SRC_DIR}/fopen_compressed.cpp)

set(LASZIP_SOURCE         ${LASZIP_SRC_DIR}/laszip.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/lasreadpoint.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/lasreaditemcompressed_v1.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/lasreaditemcompressed_v2.cpp)

LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/laswritepoint.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/laswriteitemcompressed_v1.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/laswriteitemcompressed_v2.cpp)

LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/integercompressor.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/arithmeticdecoder.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/arithmeticencoder.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/arithmeticmodel.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/lasindex.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/lasquadtree.cpp)
LIST(APPEND LASZIP_SOURCE ${LASZIP_SRC_DIR}/lasinterval.cpp)


add_library(laslib SHARED ${LASLIB_SOURCE} ${LASZIP_SOURCE})
target_include_directories(laslib PUBLIC
  $<BUILD_INTERFACE:${LASLIB_INCLUDE_DIR}>
  )
target_include_directories(laslib PUBLIC
  $<BUILD_INTERFACE:${LASZIP_INCLUDE_DIR}>
)

#add_library(laszip SHARED ${LASZIP_SOURCE})
#target_include_directories(laszip PUBLIC
  #$<BUILD_INTERFACE:${LASZIP_INCLUDE_DIR}>
#)
#target_include_directories(laszip PRIVATE
  #$<BUILD_INTERFACE:${LASLIB_INCLUDE_DIR}>
#)

#add_library(laszip-static STATIC ${LASZIP_SOURCE})
#target_include_directories(laszip-static PUBLIC
  #$<BUILD_INTERFACE:${LASZIP_INCLUDE_DIR}>
#)
#target_include_directories(laszip-static PRIVATE
  #$<BUILD_INTERFACE:${LASLIB_INCLUDE_DIR}>
#)

#set_target_properties(laszip-static PROPERTIES OUTPUT_NAME laszip)

install(TARGETS laslib
        LIBRARY DESTINATION lib)
install(DIRECTORY ${LASLIB_INCLUDE_DIR}/ ${LASZIP_INCLUDE_DIR}/
        DESTINATION include/laslib
        FILES_MATCHING PATTERN "*.hpp")

#install(TARGETS laszip
        #LIBRARY DESTINATION lib)
#install(TARGETS laszip-static
        #ARCHIVE DESTINATION lib)
#install(DIRECTORY ${LASZIP_INCLUDE_DIR}/
        #DESTINATION include/laszip
        #FILES_MATCHING PATTERN "*.hpp")

add_library(geoprojectionconverter ${LAS_BIN_DIR}/geoprojectionconverter.cpp)
target_link_libraries(geoprojectionconverter laslib)

set(BIN_TARGETS laszip lasinfo lasprecision
    txt2las las2las lasmerge las2txt lasdiff lasindex)

set(GEOPROJ_TARGETS laszip lasinfo lasprecision txt2las las2las lasmerge)

foreach(target_name ${BIN_TARGETS})
  add_executable(lastools-${target_name} ${LAS_BIN_DIR}/${target_name}.cpp)
endforeach(target_name)

foreach(target_name ${GEOPROJ_TARGETS})
  target_link_libraries(lastools-${target_name} geoprojectionconverter)
endforeach(target_name)

foreach(target_name ${BIN_TARGETS})
  target_include_directories(lastools-${target_name} PRIVATE
    $<BUILD_INTERFACE:${LAS_BIN_DIR}>
  )
  target_include_directories(lastools-${target_name} PRIVATE
    $<BUILD_INTERFACE:${LASLIB_INCLUDE_DIR}>
  )
  target_include_directories(lastools-${target_name} PRIVATE
    $<BUILD_INTERFACE:${LASZIP_INCLUDE_DIR}>
  )
  target_link_libraries(lastools-${target_name} laslib laslib)
  install(TARGETS lastools-${target_name}
          RUNTIME DESTINATION bin)
endforeach(target_name)


