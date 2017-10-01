cmake_minimum_required(VERSION 2.8.7)
project(test)

option(CLANG_ASAN_UBSAN "Enable Clang address & undefined behavior sanitizer for
binary." ON)
option(CLANG_MSAN "Enable Clang memory sanitizer for binary." OFF)
option(CLANG_TSAN "Enable Clang thread sanitizer for binary." OFF)

if((CLANG_ASAN_UBSAN AND CLANG_MSAN)
    OR (CLANG_ASAN_UBSAN AND CLANG_TSAN)
    OR (CLANG_MSAN AND CLANG_TSAN))
  message(FATAL_ERROR "Sanitizers cannot be enabled simultaneously.")
endif()

if((CLANG_ASAN_UBSAN OR CLANG_MSAN OR CLANG_TSAN) AND NOT CMAKE_C_COMPILER_ID MATCHES "Clang")
  message(FATAL_ERROR "Sanitizers are only supported for Clang.")
endif()


include(CheckCCompilerFlag)

set(execute "app")
add_executable(${execute} test.cpp)
target_link_libraries(${execute} stdc++)

if(CLANG_ASAN_UBSAN)
  message(STATUS "Enabling Clang address sanitizer and undefined behavior sanitizer.")
  check_c_compiler_flag(-fno-sanitize-recover=all SANITIZE_RECOVER_ALL)
  if(SANITIZE_RECOVER_ALL)
    set(SANITIZE_RECOVER -fno-sanitize-recover=all)     # Clang 3.6+
  else()
    set(SANITIZE_RECOVER -fno-sanitize-recover)         # Clang 3.5-
  endif()
  set_property(TARGET ${execute} APPEND_STRING PROPERTY COMPILE_FLAGS "-DEXITFREE ")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY COMPILE_FLAGS "${SANITIZE_RECOVER} -fno-omit-frame-pointer -fno-optimize-sibling-calls -fsanitize=address -fsanitize=undefined -fsanitize-blacklist=${PROJECT_SOURCE_DIR}/.asan-blacklist")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY LINK_FLAGS "-fsanitize=address -fsanitize=undefined ")
elseif(CLANG_MSAN)
  message(STATUS "Enabling Clang memory sanitizer.")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY COMPILE_FLAGS "-DEXITFREE ")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY COMPILE_FLAGS "-fsanitize=memory -fsanitize-memory-track-origins -fno-omit-frame-pointer -fno-optimize-sibling-calls ")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY LINK_FLAGS "-fsanitize=memory -fsanitize-memory-track-origins ")
elseif(CLANG_TSAN)
  message(STATUS "Enabling Clang thread sanitizer.")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY COMPILE_FLAGS "-DEXITFREE ")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY COMPILE_FLAGS "-fsanitize=thread ")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY COMPILE_FLAGS "-fPIE ")
  set_property(TARGET ${execute} APPEND_STRING PROPERTY LINK_FLAGS "-fsanitize=thread ")
endif()

