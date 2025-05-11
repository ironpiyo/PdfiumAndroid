# PDFiumAndroidのネイティブライブラリビルド設定
# 
# このファイルの目的：
# 1. PDFiumのJNIラッパーライブラリ（libjniPdfium.so）のビルド設定
# 2. プリビルドされたPDFiumライブラリ（libmodpdfium.so）の統合
# 3. 必要なAndroid NDKライブラリとのリンク設定
#
# 主な機能：
# - JNIインターフェースの実装（JavaとC++の橋渡し）
# - PDFiumのネイティブ機能へのアクセス
#
# ビルド成果物：
# - libjniPdfium.so: JNIラッパーライブラリ
# - libmodpdfium.so: PDFiumのコアライブラリ

# CMakeの最小バージョン要求
# Android NDKの要件に合わせて設定
# 3.4.1は、Android NDKがサポートする最小バージョン
cmake_minimum_required(VERSION 3.4.1)

# プロジェクト名の設定
# この名前は、ビルドシステムで使用される識別子として機能
# プロジェクト内で一意である必要がある
project(PdfiumAndroid)

# Main JNI library
add_library(jniPdfium SHARED
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/src/mainJNILib.cpp)

# インクルードディレクトリの設定
# PRIVATE: このターゲットのみが使用するインクルードパス
#  - 他のターゲットには公開されない
#  - ヘッダーファイルの検索パスを指定
# ${CMAKE_CURRENT_SOURCE_DIR}: 現在のCMakeLists.txtがあるディレクトリ
# ../jni/include: JNIのヘッダーファイルが格納されているディレクトリ
target_include_directories(jniPdfium PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/include)

# コンパイラフラグの設定
# HAVE_PTHREADS: スレッドサポートの有効化
#  - POSIXスレッド（pthread）のサポートを有効にする
#  - マルチスレッド処理に必要
# PRIVATE: このターゲットのみに適用される定義
target_compile_definitions(jniPdfium PRIVATE HAVE_PTHREADS)

# リンカーフラグの設定
# -Wl: リンカーに渡すオプション
# -z,max-page-size=16384: メモリページサイズの最大値を設定
#  - PDFiumの要件に基づく設定
#  - メモリ管理の最適化に影響
# -z,common-page-size=16384: 共通ページサイズを設定
#  - メモリアラインメントの最適化
set_target_properties(jniPdfium PROPERTIES
    LINK_FLAGS "-Wl,-z,max-page-size=16384 -Wl,-z,common-page-size=16384")

# リンクするライブラリの設定
# libmodpdfium.so: PDFiumのネイティブライブラリ
#  - PDFの解析と表示のためのコア機能を提供
#  - プリビルドされた共有ライブラリ
# log: Androidのログ機能
#  - __android_log_print()などのログ出力関数を提供
#  - デバッグやエラー追跡に使用
# android: AndroidのネイティブAPI
#  - ANativeWindowなどのAndroid固有の機能にアクセス
# jnigraphics: AndroidのグラフィックスAPI
#  - AndroidBitmapなどのグラフィックス処理機能を提供
#  - 画像処理や表示に使用
target_link_libraries(jniPdfium
    ${CMAKE_CURRENT_SOURCE_DIR}/../jni/lib/${ANDROID_ABI}/libmodpdfium.so
    log
    android
    jnigraphics) 