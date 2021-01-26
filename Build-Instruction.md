

На момент написания этой статьи Assembly3 работает только с разветвленной [веткой] FreeCAD (https://github.com/realthunder/FreeCAD/tree/LinkStage3). Вам необходимо сначала проверить эту ветку и [построить] (https://github.com/realthunder/FreeCAD/tree/LinkStage3#compiling) самостоятельно.

После этого проверьте этот репозиторий прямо в каталоге `Ext / freecad /` вашей установки FreeCAD или каталога сборки. Обязательно назовите каталог ** asm3 **. Инструментальные средства Assembly3 поддерживают несколько бэкэндов решателя ограничений. В настоящее время доступны два бэкэнда, SolveSpace и SymPy + SciPy, оба из которых имеют внешнюю зависимость. В настоящее время основное внимание уделяется тому, чтобы бэкэнд SolveSpace сначала полностью заработал, а SymPy + SciPy служит эталонной реализацией для будущих исследований. Все бэкенды необязательны. Но вам понадобится по крайней мере один установленный, чтобы иметь возможность выполнять сборку на основе ограничений, если вы не в порядке с перемещением вручную, что на самом деле выполнимо, потому что Assembly3 предоставляет мощный инструмент для перетаскивания мышью.

# SolveSpace

[SolveSpace] (http://solvespace.com/) само по себе является автономным программным обеспечением САПР с отличной поддержкой сборки. ИМО, он имеет противоположный принцип дизайна FreeCAD: большой, модульный и полностью расширяемый. SolveSpace, с другой стороны, прост и компактен и отлично справляется с тем, что предлагает. Но, скорее всего, вы обнаружите, что чего-то не хватает, и вам придется обратиться за помощью к другому программному обеспечению. Решатель ограничений SolveSpace доступен в виде небольшой библиотеки для интеграции сторонним программным обеспечением, что дает нам возможность использовать лучшее из обоих миров.

Официальной привязки SolveSpace к Python на данный момент нет. Кроме того, требуется небольшая модификация, чтобы включить функциональность сборки SolveSpace в библиотеку решателя. Вы можете найти мою вилку в подкаталоге asm3 / slvs. Проверить,

```
cd asm3
git submodule update --init slvs
```

Если вы используете Ubuntu 16.04 или 64-разрядную версию Windows, вы можете проверить предварительно созданную привязку python в подкаталоге asm3 / py_slvs.

```
cd asm3
git submodule update --init py_slvs
```

## Сборка для Ubuntu

Чтобы собрать для Ubuntu, запустите


```
apt-get install libpng12-dev libjson-c-dev libfreetype6-dev \
                libfontconfig1-dev libgtkmm-2.4-dev libpangomm-1.4-dev \
                libgl-dev libglu-dev libglew-dev libspnav-dev cmake
```

Обязательно проверьте один из необходимых субмодулей перед сборкой.

```
cd asm3/slvs
git submodule update --init extlib/libdxfrw 
```

Только для создания привязки python

```
cd asm3/slvs
mkdir build
cd build
cmake -DBUILD_PYTHON=1 ..
make _slvs
```

После завершения компиляции скопируйте slvs.py и _slvs.so из asm3 / slvs / build / src / swig / python / в asm3 / py_slvs. Перезаписать существующие файлы, если вы проверили подмодуль `py_slvs`. Если нет, то обязательно создайте пустой файл с именем `__init __. Py` в` asm3 / py_slvs`.

## Кросс-компиляция для Windows

Для сборки для 64-разрядной версии Windows у вас есть два варианта. В этом разделе показано, как выполнить кросс-компиляцию для Windows в Ubuntu.

```
apt-get install cmake mingw-w64
cd asm3/slvs
git submodule update --init --recursive
mkdir build_mingw
cd build_mingw
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_PYTHON=1 -DCMAKE_TOOLCHAIN_FILE=../cmake/Toolchain-mingw64.cmake ..
make _slvs
```
После завершения скопируйте slvs.py и _slvs.pyd из asm3 / slvs / build / src / swig / python / в asm3 / py_slvs. Перезаписать существующие файлы, если вы проверили подмодуль `py_slvs`. Если нет, то обязательно создайте пустой файл с именем `__init __. Py` в` asm3 / py_slvs`.

## Сборка в Windows

Для сборки в Windows вы должны использовать Visual Studio 2013, ту же, что и во FreeCAD. Установите CMake и Python. Если вы создаете 64-битную версию, убедитесь, что вы установили 64-битную версию Python. Я тестировал сборку только с 64-разрядной версией Python 2.7.14. Вы, вероятно, можете использовать python lib, включенную в FreeCAD libpack, добавив путь libpack в переменную окружения PATH. Но у меня как-то не получается. CMake нашел только отладочную версию python lib в libpack.

Загрузите и извлеките последний [swig] (http://www.swig.org/download.html) куда-нибудь и добавьте путь к переменной окружения PATH. Я не тестировал сборку со старой версией swig, которая идет в комплекте с FreeCAD libpack.

Обязательно проверьте все подмодули slv перед сборкой. Ни один из них фактически не используется, но по-прежнему необходим для проверки зависимостей CMake,

```
cd asm3/slvs
git submodule update --init --recursive
```

Запускаем CMake-gui, выбираем директорию сборки. Добавьте запись типа BOOL с именем BUILD_PYTHON и установите для нее значение true. Затем нажмите `configure` и выберите Visual Studio 2013 Win64, что и использовал FreeCAD. Если все прошло без ошибок, нажмите «сгенерировать».

Наконец, откройте файл `solvespace.sln` в каталоге сборки. Вам нужно создать только два проекта, сначала `slvs_static_excp`, а затем` _slvs`. После завершения скопируйте вывод в следующее место в `asm / py_slvs`

`` ''
asm / slvs /
   
    /src/swig/python/slvs.py
asm / slvs /
    
     /src/swig/python/Release/_slvs.pyd ``` 
     
If you want to build the Debug version, either download Python debug libraries, or put FreeCAD libpack directory in `PATH` environment variable before configuring CMake, so that CMake can find the debug version Python library. Once built, you must rename `_slvs.pyd` to `_slvs_d.pyd` before copying to `asm/py_slvs` ## Build for MacOS The pre-build binary for MacOS is located at a [different](../tree/master/py_slvs_mac) sub-module, because MacOS python extension has the same name as Linux one. To build it yourself for use in FreeCAD App bundle, first you need to setup `Homebrew` according to this [wiki](https://www.freecadweb.org/wiki/CompileOnMac), and build FreeCAD App bundle. Assuming you installed FreeCAD bundle at `~/some/place/FreeCAD.app`, then clone Assembly3 repository at `~/some/place/FreeCAD.app/Contents/Ext/freecad/`. And very importantly, make sure you name the clone directory as __asm3__. After that checkout `slvs` sub-module, and all of its own sub-modules. ``` cd ~/some/place/FreeCAD.app/Conntents/Ext/freecad/asm3 git submodule update --init slvs cd slvs git submodule update --init --recursive ``` Use the following command to configure and build ``` mkdir build cd build cmake \ -DCMAKE_BUILD_TYPE=Release \ -DBUILD_PYTHON=1 \ -DPYTHON_EXECUTABLE:FILEPATH=/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/bin/python2.7 \ -DPYTHON_INCLUDE_DIR=/usr/local/opt/python@2/Frameworks/Python.framework/Headers/ \ -DPYTHON_LIBRARY=/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/lib/libpython2.7.dylib \ -DPython_FRAMEWORKS=/usr/local/opt/python@2/Frameworks/Python.framework/ .. make _slvs ``` After done, create a directory named `py_slvs_mac` under `asm3`, and copy out the results ``` cd ~/some/place/FreeCAD.app/Conntents/Ext/freecad/asm3 mkdir py_slvs_mac touch py_slvs_mac/__init__.py cp slvs/build/src/swig/python/_slvs.so py_slvs_mac/ cp slvs/build/src/swig/python/slvs.py py_slvs_mac/ ``` Finally, you must make `_slvs.so` relocatable in order to be able to load it in FreeCAD bundle, with the following command ``` cd py_slvs_mac install_name_tool -id "_slvs.so" _slvs.so install_name_tool -add_rpath "@loader_path/../../../../lib/" _slvs.so install_name_tool -change \ "/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/Python" "@rpath/Python" _slvs.so ``` The last command changes the linked library path to be relative to the bundle's dynamic library loader. In case you used a different `CMake` configuration, you can find out your linked library path using the following command ``` otool -L _slvs.so ``` Done, and you can fire up FreeCAD.app and try out Assembly3. # SymPy + SciPy The other constraint solver backend uses [SymPy](http://www.sympy.org/) and [SciPy](https://www.scipy.org/). They are mostly Python based, with some native acceleration in certain critical parts. The backend implementation models after SolveSpace's solver design, that is, symbolic algebraic + non-linear least square minimization. It can be considered as a python implementation of the SolveSpace's solver. SciPy offers a dozen of different [minimization](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.minimize.html) algorithms, but most of which cannot compete with SolveSpace performance wise. The following list shows some non-formal testing result using default parameters with the sample [assembly](#create-a-super-assembly-with-external-link-array) described later | Algorithm | Time | | --------- | ---- | | SolveSpace (as reference) | 0.006s | | Nelder-Mead | Not converge | | Powell | 7.8s | | CG | 10+37s 
     [1](#f1) | | BFGS | 10+0.8 
     [1](#f1) | | Newton-CG | 10+61+0.5s 
     [2](#f2),
     [3](#f3) | | L-BFGS-B | 10+1.5s 
     [1](#f1),
     [3](#f3) | | TNC | 10+0.8s 
     [1](#f1) | | COBYLA | 0.2s 
     [3](#f3) | | SLSQP | 10+0.3 
     [1](#f1),
     [3](#f3) | | dogleg | 10+61+?s 
     [2](#f2) Failed to solve, linalg error | | trust-ncg | 10+61+1.5s 
     [2](#f2) | 
     [1] Including Jacobian matrix calculation (10s in this test case), which is implemented using sympy lambdify with numpy. 
     [2] Including Hessian matrix calculation (61s in this test case), in addition to Jacobian matrix. 
     [3] The obtained solution contains small gaps in some of the coincidence constrained points. Incorrect use of the algorithm? The reasons for writing this backend are, * SolveSpace is under GPL, which is incompatible with FreeCAD's LGPL, * To gain more insight of the solver system, and easy experimentation with new ideas due to its python based nature, * For future extension, physics based simulation, maybe? You'll need to install SymPy and SciPy for your platform. ``` pip install --upgrade sympy scipy ``` 
    
   

