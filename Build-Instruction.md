

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
```
asm / slvs /
   
    /src/swig/python/slvs.py
asm / slvs /
    
     /src/swig/python/Release/_slvs.pyd 
``` 
     
Если вы хотите создать отладочную версию, либо загрузите библиотеки отладки Python, либо поместите каталог FreeCAD libpack в переменную среды PATH перед настройкой CMake, чтобы CMake мог найти библиотеку Python отладочной версии. После сборки вы должны переименовать _slvs.pyd в _slvs_d.pyd перед копированием в asm / py_slvs.

## Сборка для MacOS

Бинарный файл предварительной сборки для MacOS находится в [другом] (../ tree / master / py_slvs_mac) субмодуле, поскольку расширение Python для MacOS имеет то же имя, что и расширение Linux. Чтобы собрать его самостоятельно для использования в пакете приложений FreeCAD, сначала вам нужно настроить `Homebrew` в соответствии с этой [вики] (https://www.freecadweb.org/wiki/CompileOnMac) и собрать пакет приложений FreeCAD.

Предполагая, что вы установили пакет FreeCAD в `~ / some / place / FreeCAD.app`, затем клонируйте репозиторий Assembly3 в` ~ / some / place / FreeCAD.app / Contents / Ext / freecad / `. И, что очень важно, убедитесь, что вы назвали каталог клонов __asm3__. После этого проверьте субмодуль `slvs` и все его собственные субмодули.

```
cd ~ / some / place / FreeCAD.app / Conntents / Ext / freecad / asm3
git обновление подмодуля --init slvs
cd slvs
git обновление подмодуля --init --recursive
```

Используйте следующую команду для настройки и сборки

```
mkdir build
cd build
cmake \
-DCMAKE_BUILD_TYPE = Выпуск \
-DBUILD_PYTHON = 1 \
-DPYTHON_EXECUTABLE: FILEPATH=/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/bin/python2.7 \
-DPYTHON_INCLUDE_DIR=/usr/local/opt/python@2/Frameworks/Python.framework/Headers/ \
-DPYTHON_LIBRARY=/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/lib/libpython2.7.dylib \
-DPython_FRAMEWORKS=/usr/local/opt/python@2/Frameworks/Python.framework/ ..
make _slvs
```

После этого создайте каталог с именем `py_slvs_mac` в` asm3` и скопируйте результаты.

```
cd ~ / some / place / FreeCAD.app / Conntents / Ext / freecad / asm3
mkdir py_slvs_mac
коснитесь py_slvs_mac / __ init__.py
cp slvs / build / src / swig / python / _slvs.so py_slvs_mac /
cp slvs / build / src / swig / python / slvs.py py_slvs_mac /
```

Наконец, вы должны сделать `_slvs.so` перемещаемым, чтобы иметь возможность загружать его в пакет FreeCAD, с помощью следующей команды

```
cd py_slvs_mac
install_name_tool -id "_slvs.so" _slvs.so
install_name_tool -add_rpath "@loader_path /../../../../ lib /" _slvs.so
install_name_tool -change \
    "/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/Python" "@ rpath / Python" _slvs.so
```

Последняя команда изменяет путь к связанной библиотеке относительно динамического загрузчика библиотеки пакета. Если вы использовали другую конфигурацию CMake, вы можете узнать путь к вашей связанной библиотеке, используя следующую команду

```
otool -L _slvs.so
```

Готово, и вы можете запустить FreeCAD.app и попробовать Assembly3.

# SymPy + SciPy

Другой бэкэнд решателя ограничений использует [SymPy] (http://www.sympy.org/) и [SciPy] (https://www.scipy.org/). В основном они основаны на Python, с некоторыми
ускорение в определенных критических частях. Модели реализации бэкэнда соответствуют дизайну решателя SolveSpace, то есть символьная алгебраическая + нелинейная минимизация по методу наименьших квадратов. Его можно рассматривать как реализацию на Python решателя SolveSpace.

SciPy предлагает десяток различных [минимизация] (https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.minimize.html)
алгоритмы, но большинство из которых не могут конкурировать с SolveSpace по производительности. В следующем списке показаны некоторые результаты неформального тестирования с использованием параметров по умолчанию в примере [assembly] (# create-a-super-assembly-with-external-link-array), описанном позже.

| Алгоритм | Время |
| --------- | ---- |
| SolveSpace (как ссылка) | 0,006 с |
| Нелдер-Мид | Не сходятся |
| Пауэлл | 7,8 с |
| CG | 10 + 37s <sup> [1] (# f1) </sup> |
| BFGS | 10 + 0.8 <sup> [1] (# f1) </sup> |
| Ньютон-CG | 10 + 61 + 0.5s <sup>[2ght(#f2),</sup> <sup> [3] (# f3) </sup> |
| L-BFGS-B | 10 + 1.5s <sup>[1ght(#f1),</sup> <sup> [3] (# f3) </sup> |
| TNC | 10 + 0.8s <sup> [1] (# f1) </sup> |
| COBYLA | 0,2 с <sup> [3] (# f3) </sup> |
| SLSQP | 10 + 0,3 <sup>[1 ](#f1),</sup> <sup> [3] (# f3) </sup> |
| изогнутый | 10 + 61 +? S <sup> [2] (# f2) </sup> Не удалось решить, ошибка linalg |
| траст-нкг | 10 + 61 + 1.5s <sup> [2] (# f2) </sup> |

<b name = "f1"> [1] </b> Включая вычисление матрицы Якоби (10 в этом тестовом примере), которое реализовано с использованием sympy lambdify с numpy.

<b name = "f2"> [2] </b> Включая вычисление матрицы Гессе (61 сек в данном тестовом примере) в дополнение к матрице Якоби.

<b name = "f3"> [3] </b> Полученное решение содержит небольшие зазоры в некоторых точках ограничения совпадений. Неправильное использование алгоритма?

Причины написания этого бэкэнда:

* SolveSpace находится под лицензией GPL, что несовместимо с FreeCAD LGPL,
* Чтобы получить больше информации о системе решателя и легко экспериментировать с новыми идеями благодаря природе, основанной на Python,
* Возможно, для будущего расширения, моделирования на основе физики?

Вам нужно будет установить SymPy и SciPy для вашей платформы.

```
pip install --upgrade sympy scipy
```
