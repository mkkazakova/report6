Создаём репозиторий.
В локальном репозитории. Создаём папку lab-06. Связываем её с нашим репозиторием. И заходим в эту папку.
Долее скачиваем необходимые библиотеки и файлы.

##### formatter_lib

```
$ mkdir formatter_lib
$ cd formatter_lib
$ wget https://raw.githubusercontent.com/tp-labs/lab03/master/formatter_lib/formatter.cpp
$ wget https://raw.githubusercontent.com/tp-labs/lab03/master/formatter_lib/formatter.h
$ cat >> CMakeLists.txt << EOF
>EOF
$ nano CMakeLists.txt
```

Содержимое файла CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.4)

project(formatter_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter_lib STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter.cpp ${CMAKE_CURRENT_SOURCE_DIR}/formatter.h)
```

##### formatter_ex_lib

```
$ cd ..
$ mkdir formatter_ex_lib
$ cd formatter_ex_lib
$ wget https://raw.githubusercontent.com/tp-labs/lab03/master/formatter_ex_lib/formatter_ex.cpp
$ wget https://raw.githubusercontent.com/tp-labs/lab03/master/formatter_ex_lib/formatter_ex.h
$ nano formatter_ex.cpp
```

Новое содержимое файла formatter_ex.cpp:

```
#include "formatter.h"

std::ostream& formatter(std::ostream& out, const std::string& message)
{
    return out << formatter(message);
}
```

```
$ cat >> CMakeLists.txt << EOF
>EOF
$ nano CMakeLists.txt
```

Содержимое файла CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.4)

project(formatter_ex_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib formatter_lib_dir)

add_library(formatter_ex_lib STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex.cpp)

target_include_directories(formatter_ex_lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib )

target_link_libraries(formatter_ex_lib formatter_lib)
```

##### solver_lib

```
$ cd ..
$ mkdir solver_lib
$ cd solver_lib
$ wget https://raw.githubusercontent.com/tp-labs/lab03/master/solver_lib/solver.cpp
$ wget https://raw.githubusercontent.com/tp-labs/lab03/master/solver_lib/solver.h
$ nano solver.cpp
```

Меняем ошибку в файле solver.cpp:

```
#include "solver.h"

#include <stdexcept>
#include <math.h>

void solve(float a, float b, float c, float& x1, float& x2)
{
    float d = (b * b) - (4 * a * c);

    if (d < 0)
    {
        throw std::logic_error{"error: discriminant < 0"};
    }

    x1 = (-b - std::sqrt(d)) / (2 * a);
    x2 = (-b + std::sqrt(d)) / (2 * a);
}
```

##### solver_application

```
$ cd ..
$ mkdir solver_application
$ cd solver_application
$ wget https://raw.githubusercontent.com/tp-labs/lab03/master/solver_application/equation.cpp
$ cat >> CMakeLists.txt << EOF
> EOF
$ nano CMakeLists.txt
```

Содержимое файла CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.4)

project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(/../formatter_ex_lib formatter_ex_lib_dir)

add_library(solver_lib /../solver_lib/solver.cpp /../solver_lib/solver.h)
add_executable(solver /equation.cpp)

target_include_directories(formatter_ex_lib PUBLIC /../formatter_lib /../formatter_ex_lib /../solver_lib)

target_link_libraries(solver formatter_ex_lib formatter_lib solver_lib)
```

##### Всё коммитим и пушим

```
$ cd ..
$ git add formatter_lib
$ git add formatter_ex_lib
$ git add solver_lib
$ git add solver_application
$ git commit -m "add lib"
$ git push origin master
```

##### CMakeLists.txt (общий)

CMakeLists.txt для общей работы программы. Он связывает все директории.

```
$ cat >> CMakeLists.txt << EOF
>EOF
$ nano CMakeLists.txt 
```

Содержимое файла CMakeLists.txt:

```
cmake_minimum_required(VERSION 3.4)

project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex_lib formatter_ex_lib_dir)

add_library(solver_lib ${CMAKE_CURRENT_SOURCE_DIR}/solver_lib/solver.cpp)
add_executable(solver ${CMAKE_CURRENT_SOURCE_DIR}/solver_application/equation.cpp)

target_include_directories(formatter_ex_lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter_lib ${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex_lib ${CMAKE_CURRENT_SOURCE_DIR}/solver_lib)

target_link_libraries(solver formatter_ex_lib formatter_lib solver_lib)

install(TARGETS solver
    RUNTIME DESTINATION bin
)

#Передает управление средству пакетирования, которое мы создадим позже
include(CPackConfig.cmake)
```

Коммитим и пушим CMakeLists.txt

```
$ git add CMakeLists.txt 
$ git commit -m "CMakeLists.txt for all libs"
$ git push origin master
```

##### CPackConfig.cmake

CPackConfig.cmake - средство пакетирования

```
$ cat >> CPackConfig.cmake << EOF
>EOF
$ nano CPackConfig.cmake
```

Содержимое файла CMakeLists.txt:

```
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT mkkazakova@yandex.ru)
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "C++ app for solving quadratic equations")
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_SOURCE_IGNORE_FILES  "\\\\.cmake;/build/;/.git/;/.github/")

set(CPACK_SOURCE_INSTALLED_DIRECTORIES "${CMAKE_SOURCE_DIR}; /")

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")

set(CPACK_DEBIAN_PACKAGE_NAME "solverapp-dev")
set(CPACK_DEBIAN_FILE_NAME "solver-${PRINT_VERSION}.deb")
set(CPACK_DEBIAN_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "mkkazakova")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

set(CPACK_GENERATOR "DEB")

include(CPack)
```

Коммитим и пушим CPackConfig.cmake

```
$ git add CPackConfig.cmake 
$ git commit -m "CPackConfig"
$ git push origin master
```

##### DESCRIPTION и LICENSE

```
$ cat >> DESCRIPTION << EOF
>Enter 3 coefficients for x
>EOF
$ wget https://raw.githubusercontent.com/tp-labs/lab03/master/LICENSE
$ git add DESCRIPTION
$ git add LICENSE
$ git commit -m "DESCRIPTION and LICENSE"
$ git push origin master
```

##### workflows (сценарии)

```
$ mkdir .github
$ cd .github
$ mkdir workflows
$ cd workflows
```

Создаём Action.yml (для работы Solver)

```
$ cat >> Action.yml << EOF
>EOF
$ nano Action.yml
```

Содержимое Action.yml:

```
name: CMake

on:
 push:
  branches: [master]
 pull_request:
  branches: [master]

jobs: 
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Configure Solver
    run: cmake ${{github.workspace}} -B ${{github.workspace}}/build

  - name: Build Solver
    run: cmake --build ${{github.workspace}}/build
```

Создаём Release.yml (для создания пакетов)

```
$ cat >> Release.yml << EOF
>EOF
$ nano Release.yml
```

Содержимое Release.yml:

```
name: CMake

on:
 push:
   tags:
     - v**

permissions:
  contents: write

jobs: 

  build_packages_Linux:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Configure Solver
      run: cmake ${{github.workspace}} -B ${{github.workspace}}/build -D PRINT_VERSION=${GITHUB_REF_NAME#v}

    - name: Build Solver
      run: cmake --build ${{github.workspace}}/build

    - name: Build package
      run: cmake --build ${{github.workspace}}/build --target package

    - name: Build source package
      run: cmake --build ${{github.workspace}}/build --target package_source

    - name: Make a release
      uses: ncipollo/release-action@v1.10.0
      with:
        artifacts: "build/*.deb,build/*.tar.gz,build/*.zip"
        replacesArtifacts: false
        GITHUB_TOKEN: ${{ secrets.GH_PAT }}
        allowUpdates: true
```

Коммитим и пушим Action.yml и Release.yml

```
$ git add Action.yml
$ git add Release.yml
$ git commit -m "Action.yml and Release.yml"
$ git push origin master
```

##### releases

Заходим на сайт github. Справа от файлов 

![image](https://user-images.githubusercontent.com/125077130/230469212-d8cf0c48-d477-46de-be72-897f0eefef18.png)

Нажимаем. Далее

![image](https://user-images.githubusercontent.com/125077130/230469352-c8b77793-ec3c-4346-825f-2d19b7cbe2a3.png)

![image](https://user-images.githubusercontent.com/125077130/230469389-83ac98e8-59a5-4898-85aa-551fd1c51b9d.png)

![image](https://user-images.githubusercontent.com/125077130/230469431-b48166f6-3ce0-418d-8747-836c38b63947.png)

![image](https://user-images.githubusercontent.com/125077130/230469475-a10640d6-32fc-45bd-b08e-d057abbf4f5a.png)

![image](https://user-images.githubusercontent.com/125077130/230469527-d848ef6d-16b5-475e-a80a-711dc7b5dab7.png)


Далее проверяем, чтобы было 2 галочки в Actions

![image](https://user-images.githubusercontent.com/125077130/230469901-ac6cd972-190e-46b4-94b7-b00f2d4f8407.png)


### Скрины

![image](https://user-images.githubusercontent.com/125077130/230470042-705f0344-e04e-4ef8-8676-e765165012a5.png)

![image](https://user-images.githubusercontent.com/125077130/230470111-316fe848-da6e-42a1-8042-1cddbb5c3789.png)

![image](https://user-images.githubusercontent.com/125077130/230470146-20ab7c9f-8863-462a-a44a-4beed94765c4.png)

![image](https://user-images.githubusercontent.com/125077130/230470169-19e4647c-0b01-491f-a139-3df8db9435c7.png)

![image](https://user-images.githubusercontent.com/125077130/230470209-84256207-9eec-4248-937a-a2299b274bb6.png)

![image](https://user-images.githubusercontent.com/125077130/230470234-35cf8d56-edbc-4924-8aa7-caa481e84b5a.png)

![image](https://user-images.githubusercontent.com/125077130/230470270-764e2350-76ea-4668-b73d-c808c942daf3.png)









