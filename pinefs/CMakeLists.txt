cmake_minimum_required(VERSION 3.10)

find_package(OpenSSL REQUIRED)
set(pinefs_LIBS ${OPENSSL_CRYPTO_LIBRARY} PARENT_SCOPE)
set(pinefs_INCLUDE_PATHS ${OPENSSL_INCLUDE_DIR} PARENT_SCOPE)

set(pinefs_SOURCES "pinefs.cc" PARENT_SCOPE)
set(pinefs_HEADERS "pinefs.h" PARENT_SCOPE)
set(pinefs_CMAKE_EXE_LINKER_FLAGS "-u pinefs_reg" PARENT_SCOPE)
