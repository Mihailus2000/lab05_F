## Laboratory work V

Данная лабораторная работа посвещена изучению фреймворков для тестирования на примере **GTest**

```ShellSession
$ open https://github.com/google/googletest
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab05** на сервисе **GitHub**
- [x] 2. Выполнить инструкцию учебного материала
- [x] 3. Ознакомиться со ссылками учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial
Настройка окружения
```ShellSession           
$ export GITHUB_USERNAME=Mihailus2000       # создание переменной окружения
$ alias gsed=sed # for *-nix system         # Создание синонима gsed 
```
Подготовка окружения к ЛР
```ShellSession
$ cd ${GITHUB_USERNAME}/workspace       # Переход в директорию
$ pushd .                               # Сохранение текущей директории

~/Mihailus2000/workspace ~/Mihailus2000/workspace

$ source scripts/activate               # Активация скрипта подготовки окружения
``` 

Получение ЛР4 из репозитория
```ShellSession
$ git clone https://github.com/${GITHUB_USERNAME}/lab04 projects/lab05     # Скачивание из удалённого репозитория

Клонирование в «projects/lab05»…
remote: Enumerating objects: 36, done.
remote: Counting objects: 100% (36/36), done.
remote: Compressing objects: 100% (21/21), done.
remote: Total 36 (delta 10), reused 36 (delta 10), pack-reused 0
Распаковка объектов: 100% (36/36), готово.

$ cd projects/lab05                               # Переход в директорию
$ git remote remove origin                        # Удаление ссылки на репозиторий
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab05     # Указание ссылки на новый репозиторий
```

Добавление субмодуля тестирования
```ShellSession
$ mkdir third-party       # Создание директории
$ git submodule add https://github.com/google/googletest third-party/gtest  # Скачивание удалённого репозиотрия в указанную папку

Клонирование в «/home/mihailus/Mihailus2000/workspace/projects/lab05/third-party/gtest»…
remote: Enumerating objects: 48, done.
remote: Counting objects: 100% (48/48), done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 16857 (delta 15), reused 26 (delta 11), pack-reused 16809
Получение объектов: 100% (16857/16857), 5.86 MiB | 4.79 MiB/s, готово.
Определение изменений: 100% (12412/12412), готово.

$ cd third-party/gtest && git checkout release-1.8.1 && cd ../..  # Переход в нужную директорию, переход в указаннную ветку git, возврат на 2 уровня назад в файловой системе

Примечание: переход на «release-1.8.1».

Вы сейчас в состоянии «отделённого HEAD». Вы можете осмотреться, сделать
экспериментальные изменения и закоммитить их, также вы можете отменить
изменения любых коммитов в этом состоянии не затрагивая любые ветки и
не переходя на них.

Если вы хотите создать новую ветку и сохранить свои коммиты, то вы
можете сделать это (сейчас или позже) вызвав команду checkout снова,
но с параметром -b. Например:

  git checkout -b <имя-новой-ветки>

HEAD сейчас на 2fe3bd99 Merge pull request #1433 from dsacre/fix-clang-warnings

$ git add third-party/gtest     # Фиксация изменений
$ git commit -m"added gtest framework"      # Коммит зафиксированных изменений

[master 86b62aa] added gtest framework
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 third-party/gtest

```
Добавление в CMakeLists.txt сборку тестов
```ShellSession   
$ gsed -i '/option(BUILD_EXAMPLES "Build examples" OFF)/a\      # Вставка строки после указанной первой строки
option(BUILD_TESTS "Build tests" OFF)       
' CMakeLists.txt    
$ cat >> CMakeLists.txt <<EOF             # Добавление кода в CMakeLists.txt кода

if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB \${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check \${\${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check \${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```
Создание кода с тестами
```ShellSession
$ mkdir tests                          # Создание директории
$ cat > tests/test1.cpp <<EOF          # Запись в файл
#include <print.hpp>

#include <gtest/gtest.h>                

TEST(Print, InFileStream)
{
  std::string filepath = "file.txt";
  std::string text = "hello";
  std::ofstream out{filepath};

  print(text, out);
  out.close();

  std::string result;
  std::ifstream in{filepath};
  in >> result;

  EXPECT_EQ(result, text);
}
EOF
```
Сборка проекта
```ShellSession
$ cmake -H. -B_build -DBUILD_TESTS=ON           # Конфигурирование

-- The C compiler identification is GNU 8.2.0
-- The CXX compiler identification is GNU 8.2.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found PythonInterp: /usr/bin/python (found version "2.7.15") 
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Looking for pthread_create
-- Looking for pthread_create - not found
-- Check if compiler accepts -pthread
-- Check if compiler accepts -pthread - yes
-- Found Threads: TRUE  
-- Configuring done
-- Generating done
-- Build files have been written to: /home/mihailus/Mihailus2000/workspace/projects/lab05/_build

$ cmake --build _build           # Компиляция

Scanning dependencies of target gtest
[  8%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest.dir/src/gtest-all.cc.o
[ 16%] Linking CXX static library libgtest.a
[ 16%] Built target gtest
Scanning dependencies of target gtest_main
[ 25%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest_main.dir/src/gtest_main.cc.o
[ 33%] Linking CXX static library libgtest_main.a
[ 33%] Built target gtest_main
Scanning dependencies of target print
[ 41%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 50%] Linking CXX static library libprint.a
[ 50%] Built target print
Scanning dependencies of target check
[ 58%] Building CXX object CMakeFiles/check.dir/tests/test1.cpp.o
[ 66%] Linking CXX executable check
[ 66%] Built target check
Scanning dependencies of target gmock
[ 75%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock.dir/src/gmock-all.cc.o
[ 83%] Linking CXX static library libgmock.a
[ 83%] Built target gmock
Scanning dependencies of target gmock_main
[ 91%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock_main.dir/src/gmock_main.cc.o
[100%] Linking CXX static library libgmock_main.a
[100%] Built target gmock_main

$ cmake --build _build --target test          # Компиляция цели

Running tests...
Test project /home/mihailus/Mihailus2000/workspace/projects/lab05/_build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.00 sec

```

Проверка тестов
```ShellSession
$ _build/check      # Запуск тестов

Running main() from /home/mihailus/Mihailus2000/workspace/projects/lab05/third-party/gtest/googletest/src/gtest_main.cc
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from Print
[ RUN      ] Print.InFileStream
[       OK ] Print.InFileStream (0 ms)
[----------] 1 test from Print (1 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (1 ms total)
[  PASSED  ] 1 test.

$ cmake --build _build --target test -- ARGS=--verbose        # Компиляция с выводом информации

Running tests...
UpdateCTestConfiguration  from :/home/mihailus/Mihailus2000/workspace/projects/lab05/_build/DartConfiguration.tcl
UpdateCTestConfiguration  from :/home/mihailus/Mihailus2000/workspace/projects/lab05/_build/DartConfiguration.tcl
Test project /home/mihailus/Mihailus2000/workspace/projects/lab05/_build
Constructing a list of tests
Done constructing a list of tests
Updating test list for fixtures
Added 0 tests to meet fixture requirements
Checking test dependency graph...
Checking test dependency graph end
test 1
    Start 1: check

1: Test command: /home/mihailus/Mihailus2000/workspace/projects/lab05/_build/check
1: Test timeout computed to be: 10000000
1: Running main() from /home/mihailus/Mihailus2000/workspace/projects/lab05/third-party/gtest/googletest/src/gtest_main.cc
1: [==========] Running 1 test from 1 test case.
1: [----------] Global test environment set-up.
1: [----------] 1 test from Print
1: [ RUN      ] Print.InFileStream
1: [       OK ] Print.InFileStream (0 ms)
1: [----------] 1 test from Print (0 ms total)
1: 
1: [----------] Global test environment tear-down
1: [==========] 1 test from 1 test case ran. (0 ms total)
1: [  PASSED  ] 1 test.
1/1 Test #1: check ............................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.00 sec

```
Обновление Travis CI конфига и бэйджа

```ShellSession
$ gsed -i 's/lab04/lab05/g' README.md       # Свап строк
$ gsed -i 's/\(DCMAKE_INSTALL_PREFIX=_install\)/\1 -DBUILD_TESTS=ON/' .travis.yml    # Дописывание к первой строке правой
$ gsed -i '/cmake --build _build --target install/a\      # Дописывание строк
- cmake --build _build --target test -- ARGS=--verbose
' .travis.yml
```
Проверка конфига
```ShellSession
$ travis lint    # Проверка 

Warnings for .travis.yml:
[x] value for addons section is empty, dropping
[x] in addons section: unexpected key apt, dropping
```
Отправка изменений на удалённый репозиторий
```ShellSession
$ git add .travis.yml   # Фиксация изменений файла    
$ git add tests         # Фиксация изменений файла
$ git add -p            # Фиксация изменений файла

diff --git a/CMakeLists.txt b/CMakeLists.txt
index 05cc72b..176b7ba 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -4,6 +4,7 @@ set(CMAKE_CXX_STANDARD 11)
 set(CMAKE_CXX_STANDARD_REQUIRED ON)
 
 option(BUILD_EXAMPLES "Build examples" OFF)
+option(BUILD_TESTS "Build tests" OFF)
 
 project(print)
 
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y         
@@ -34,3 +35,12 @@ install(TARGETS print
 
 install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
 install(EXPORT print-config DESTINATION cmake)
+
+if(BUILD_TESTS)
+  enable_testing()
+  add_subdirectory(third-party/gtest)
+  file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
+  add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
+  target_link_libraries(check ${PROJECT_NAME} gtest_main)
+  add_test(NAME check COMMAND check)
+endif()
Stage this hunk [y,n,q,a,d,K,g,/,e,?]? y

$ git commit -m"added tests"    # Коммит изменений
$ git push origin master        # Отправка изменений на репозиторий

Username for 'https://github.com': Mihailus2000
Password for 'https://Mihailus2000@github.com': 
Перечисление объектов: 40, готово.
Подсчет объектов: 100% (40/40), готово.
При сжатии изменений используется до 4 потоков
Сжатие объектов: 100% (34/34), готово.
Запись объектов: 100% (40/40), 11.83 KiB | 1.48 MiB/s, готово.
Всего 40 (изменения 11), повторно использовано 0 (изменения 0)
remote: Resolving deltas: 100% (11/11), done.
To https://github.com/Mihailus2000/lab05
 * [new branch]      master -> master

```
Авторизация и активация travis
```ShellSession
$ travis login --auto     # Авторизация

We need your GitHub login to identify you.
This information will not be sent to Travis CI, only to api.github.com.
The password will not be displayed.

Try running with --github-token or --auto if you don't want to enter your password anyway.

Username: Mihailus2000
Password for Mihailus2000: *********************
Successfully logged in as Mihailus2000!

$ travis enable       # Активация

Detected repository as Mihailus2000/lab05, is this correct? |yes| y
Mihailus2000/lab05: enabled :)

```
Сохранение 
```ShellSession
$ mkdir artifacts
$ sleep 20s && gnome-screenshot --file artifacts/screenshot.png            # Команда отсутствует
# Скриншот сделан вручную
# for macOS: $ screencapture -T 20 artifacts/screenshot.png
# open https://github.com/${GITHUB_USERNAME}/lab05
```

## Report

```ShellSession
$ popd
$ export LAB_NUMBER=05
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gistup -m "lab${LAB_NUMBER}"
```

## Homework

### Задание
1. Создайте `CMakeList.txt` для библиотеки *banking*.
2. Создайте модульные тесты на классы `Transaction` и `Account`.
    * Используйте mock-объекты.
    * Покрытие кода должно составлять 100%.
3. Настройте сборочную процедуру на **TravisCI**.
4. Настройте [Coveralls.io](https://coveralls.io/).

## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2019 The ISC Authors
```
