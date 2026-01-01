Установить расширения:
C++ TestMate - для запуска тестов(отдельная панель в VScode)
clangd - основное приложения для работы с С++ (альтернатива intellisence). включает форматтер clang-format и линтер clang-tidy (должны быть установлены в системе все 3)
CMake Tools - для настройки запуска cmake проекта
CodeLLDB - дебаггер для С++

Структура проекта:
-src		 - исходники с кодом *.cpp
-include	 - исходники с кодом *.h
-tests		 - тесты
-build		 - сборочные файлы для cmake
-.vscode	 - настройки vscode: launch.json(для запуска дебаггера) settings.json(настройки проекта)
-.git
.clang-format	 - настройки форматера
.clang-tidy	 - настройки линтера
CMakeLists.txt
CMakePresets.json - описывает Debug/Release и build‑папки

Запускать:
ctrl+shift+p
CMake: Select Configure Preset → debug или release.
Build в статус‑баре(с шестеренкой-сконфигурирует и сбилдит)
Run стрелочка рядом с шестеренкой

Для запуска целей format и tidy:
ctrl+shift+p -> Cmake: Build Target -> format/tidy
После этого tidy должен заработать лучше.