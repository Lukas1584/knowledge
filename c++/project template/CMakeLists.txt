cmake_minimum_required(VERSION 3.20)
project(smmr LANGUAGES CXX)

# Генерация compile_commands.json (для clangd/VS Code)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Стандарт C++
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ===== Источники =====
# Все .cpp из src/, кроме main.cpp — это "общие" файлы
file(GLOB_RECURSE SMMR_ALL_CPP CONFIGURE_DEPENDS
    "${CMAKE_SOURCE_DIR}/src/*.cpp"
)
# Исключаем main.cpp из общего набора чтобы не было в тестах дублирования
set(SMMR_COMMON_SOURCES "")
foreach(src ${SMMR_ALL_CPP})
    if (NOT src MATCHES ".*/main\\.cpp$")
        list(APPEND SMMR_COMMON_SOURCES "${src}")
    endif()
endforeach()

# Заголовки (для удобства целей format/tidy)
file(GLOB_RECURSE SMMR_HEADERS CONFIGURE_DEPENDS
    "${CMAKE_SOURCE_DIR}/include/**/*.h"
)

# Найти установленную библиотеку libcurl и импортируемую цель CURL::libcurl
find_package(CURL REQUIRED)

# ===== Исполняемый файл =====
add_executable(smmr
    src/main.cpp
    ${SMMR_COMMON_SOURCES}
)

# include директория (заголовки проекта)
target_include_directories(smmr
    PRIVATE
        ${CMAKE_SOURCE_DIR}/include
)

# Линковка с libcurl
target_link_libraries(smmr PRIVATE CURL::libcurl)

# Предупреждения: GCC/Clang vs MSVC
if (MSVC)
    target_compile_options(smmr PRIVATE /W4 /permissive-)
else()
    target_compile_options(smmr PRIVATE -Wall -Wextra -Wpedantic)
endif()

# ===== Санитайзеры (только GCC/Clang, только Debug) =====
option(SMMR_ENABLE_SANITIZERS "Enable ASan/UBSan in Debug for GCC/Clang" ON)
if (SMMR_ENABLE_SANITIZERS AND (CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU"))
    target_compile_options(smmr PRIVATE
        $<$<CONFIG:Debug>:-fsanitize=address,undefined>
    )
    target_link_options(smmr PRIVATE
        $<$<CONFIG:Debug>:-fsanitize=address,undefined>
    )
endif()

# ===== Тесты (GoogleTest + CTest) =====
include(CTest)  # добавляет опцию BUILD_TESTING (ON по умолчанию)

if (BUILD_TESTING)
    find_package(GTest QUIET)
    if (GTest_FOUND)
        file(GLOB_RECURSE SMMR_TEST_SOURCES CONFIGURE_DEPENDS
            "${CMAKE_SOURCE_DIR}/tests/*_test.cpp"
            "${CMAKE_SOURCE_DIR}/tests/*Test.cpp"
        )
        if (SMMR_TEST_SOURCES)
            # Тестовый исполняемый файл компилируем с тем же общим кодом
            add_executable(smmr_tests
                ${SMMR_TEST_SOURCES}
                ${SMMR_COMMON_SOURCES}
            )
            target_include_directories(smmr_tests PRIVATE ${CMAKE_SOURCE_DIR}/include)

            # Предупреждения
            if (MSVC)
                target_compile_options(smmr_tests PRIVATE /W4 /permissive-)
            else()
                target_compile_options(smmr_tests PRIVATE -Wall -Wextra -Wpedantic)
            endif()

            # Санитайзеры (те же условия)
            if (SMMR_ENABLE_SANITIZERS AND (CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU"))
                target_compile_options(smmr_tests PRIVATE
                    $<$<CONFIG:Debug>:-fsanitize=address,undefined>
                )
                target_link_options(smmr_tests PRIVATE
                    $<$<CONFIG:Debug>:-fsanitize=address,undefined>
                )
            endif()

            target_link_libraries(smmr_tests PRIVATE GTest::gtest GTest::gtest_main)

            include(GoogleTest)
            gtest_discover_tests(smmr_tests
                DISCOVERY_TIMEOUT 60
            )
        endif()
    else()
        message(STATUS "GTest not found: tests will not be built")
    endif()
endif()

# ===== Утилитарные цели: format/tidy =====
# Полный набор исходников (заголовки + исходники, включая main.cpp)
set(ALL_SOURCE_FILES ${SMMR_COMMON_SOURCES} ${SMMR_HEADERS} "${CMAKE_SOURCE_DIR}/src/main.cpp")

add_custom_target(format
    COMMAND clang-format -i ${ALL_SOURCE_FILES}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMENT "Running clang-format over sources"
)

add_custom_target(tidy
    COMMAND clang-tidy -p ${CMAKE_BINARY_DIR} ${ALL_SOURCE_FILES}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMENT "Running clang-tidy over sources"
)
