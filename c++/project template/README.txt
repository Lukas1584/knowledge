Установить расширения:
C++ TestMate - для запуска тестов отдельная панель
clangd - основное приложения для работы с С++ (альтернатива intellisence). включает форматтер clang-format и линтер clang-tidy (должны быть установлены в системе все 3)
CMake Tools - для настройки проектов через cmake
CodeLLDB - дебаггер для С++

Структура проекта:
-src		 - исходники с кодом *.cpp
-include	 - исходники с кодом *.h
-tests		 - тесты
-build		 - сборочные файлы для cmake
-.vscode	 - настройки vscode: launch.json settings.json tasks.json
-.git
.clang-format	 - настройки форматера
.clang-tidy	 - настройки линтера
CMakeLists.txt
CMakePresets.json - описывает Debug/Release и build‑папки

Запускать:
ctrl+shift+p
CMake: Select Configure Preset → debug или release.
Build в статус‑баре(с шестеренкой)
Run/ctrl+shift+p -> Cmake: Debug (shift+F5)/ctrl+shift+p -> Cmake: Run Without Debbuging (ctrl+shift+F5)

Для запуска целей format и tidy:
ctrl+shift+p -> Cmake: Build Target -> format/tidy
Только после этого tidy заработает в коде.