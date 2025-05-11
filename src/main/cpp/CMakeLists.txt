# CMakeの最小バージョン要求（Android NDKの要件に合わせて設定）
cmake_minimum_required(VERSION 3.4.1)

# プロジェクト名の設定（ビルドシステムで使用される識別子）
project(PdfiumAndroid)

# Prebuilt library
add_library(aospPdfium SHARED IMPORTED)
set_target_properties(aospPdfium PROPERTIES IMPORTED_LOCATION
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/lib/${ANDROID_ABI}/libmodpdfium.so)

# Main JNI library
add_library(jniPdfium SHARED
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/src/mainJNILib.cpp)

# インクルードディレクトリの設定
# PRIVATE: このターゲットのみが使用するインクルードパス
target_include_directories(jniPdfium PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/include)

# コンパイラフラグの設定
# HAVE_PTHREADS: スレッドサポートの有効化
target_compile_definitions(jniPdfium PRIVATE HAVE_PTHREADS)

# リンカーフラグの設定
# -Wl: リンカーに渡すオプション
# -z,max-page-size=16384: メモリページサイズの設定（PDFiumの要件）
# -z,common-page-size=16384: 共通ページサイズの設定
set_target_properties(jniPdfium PROPERTIES
    LINK_FLAGS "-Wl,-z,max-page-size=16384 -Wl,-z,common-page-size=16384")

# リンクするライブラリの設定
# libmodpdfium.so: PDFiumのネイティブライブラリ
# log: Androidのログ機能
# android: AndroidのネイティブAPI
# jnigraphics: AndroidのグラフィックスAPI
target_link_libraries(jniPdfium
    aospPdfium
    log
    android
    jnigraphics)

# ビルド後の処理
# POST_BUILD: ライブラリのビルド完了後に実行
# ${CMAKE_COMMAND} -E copy: CMakeのコピーコマンド
# libmodpdfium.soを出力ディレクトリにコピー
add_custom_command(TARGET jniPdfium POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
        ${CMAKE_CURRENT_SOURCE_DIR}/../jni/lib/${ANDROID_ABI}/libmodpdfium.so
        $<TARGET_FILE_DIR:jniPdfium>/libmodpdfium.so) 