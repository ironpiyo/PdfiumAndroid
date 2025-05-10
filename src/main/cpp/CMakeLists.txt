cmake_minimum_required(VERSION 3.4.1)
project(PdfiumAndroid)

# Prebuilt library
add_library(aospPdfium SHARED IMPORTED)
set_target_properties(aospPdfium PROPERTIES IMPORTED_LOCATION
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/lib/${ANDROID_ABI}/libmodpdfium.so)

# Main JNI library
add_library(jniPdfium SHARED
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/src/mainJNILib.cpp)

target_include_directories(jniPdfium PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/include)

target_link_libraries(jniPdfium
    aospPdfium
    log
    android
    jnigraphics)

# Add compiler flags
target_compile_definitions(jniPdfium PRIVATE HAVE_PTHREADS)

# Add linker flags
set_target_properties(jniPdfium PROPERTIES
    LINK_FLAGS "-Wl,-z,max-page-size=16384 -Wl,-z,common-page-size=16384") 