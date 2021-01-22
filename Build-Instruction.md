На момент написания этой статьи Assembly3 работает только с разветвленным FreeCAD.
[ветка] (https://github.com/realthunder/FreeCAD/tree/LinkStage3). Вам нужно
сначала проверьте эту ветку и [сборку] (https://github.com/realthunder/FreeCAD/tree/LinkStage3#compiling)
это сами.

После этого проверьте этот репозиторий прямо внутри `Ext / freecad /`
каталог вашей установки FreeCAD или каталог сборки. Обязательно назовите
каталог как **asm3**. Верстак Assembly3 поддерживает множественные ограничения
бэкэнды решателя. В настоящее время доступны два бэкенда: `SolveSpace` и
SymPy + SciPy, оба из которых имеют внешнюю зависимость. Текущее внимание уделяется
чтобы сначала полностью запустить бэкэнд SolveSpace, с SymPy + SciPy, выступающим в качестве
эталонная реализация для будущих исследований. Все бэкенды необязательны.
Но вам понадобится хотя бы один установленный, чтобы иметь возможность делать на основе ограничений
сборка, если вас не устраивает движение вручную, что на самом деле
выполнимо, потому что Assembly3 предоставляет мощный инструмент для перетаскивания мышью.

# SolveSpace

[SolveSpace] (http://solvespace.com/) сам по себе является автономным программным обеспечением САПР.
с отличной монтажной опорой. ИМО, он имеет противоположный принцип конструкции
FreeCAD - большой, модульный и полностью расширяемый. SolveSpace, с другой стороны
Рука тонкая и компактная, и она очень хорошо справляется с тем, что предлагает. Но ты
скорее всего, вы найдете то, что вам нужно, чего не хватает, и вам придется искать
другой софт для помощи. Решатель ограничений SolveSpace доступен как
небольшая библиотека для интеграции стороннего программного обеспечения, которая дает нам
возможность принести лучшее из обоих миров.

Официальной привязки SolveSpace к Python на данный момент нет. Кроме того, некоторые
требуется небольшая модификация для вывода сборки SolveSpace
функциональность в библиотеке решателя. Вы можете найти мою вилку в `asm3 / slvs`
подкаталог. Проверить,

```
cd asm3
git submodule update --init slvs
```

Если вы используете Ubuntu 16.04 или 64-разрядную версию Windows, вы можете проверить
предварительно созданная привязка python в подкаталоге asm3 / py_slvs.

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

После завершения компиляции скопируйте slvs.py и _slvs.so из
`asm3 / slvs / build / src / swig / python /` в `asm3 / py_slvs`. Перезаписать
существующие файлы, если вы проверили подмодуль `py_slvs`. Если нет, то будь
обязательно создайте пустой файл с именем `__init __. py` в` asm3 / py_slvs`.

## Кросс-компиляция для Windows

Для сборки для 64-разрядной версии Windows у вас есть два варианта. В этом разделе показано, как
кросс-компиляция для Windows на Ubuntu

```
apt-get install cmake mingw-w64
cd asm3/slvs
git submodule update --init --recursive
mkdir build_mingw
cd build_mingw
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_PYTHON=1 -DCMAKE_TOOLCHAIN_FILE=../cmake/Toolchain-mingw64.cmake ..
make _slvs
```
После завершения скопируйте slvs.py и _slvs.pyd из
`asm3 / slvs / build / src / swig / python /` в `asm3 / py_slvs`. Перезаписать
существующие файлы, если вы проверили подмодуль `py_slvs`. Если нет, то будь
обязательно создайте пустой файл с именем `__init __. py` в` asm3 / py_slvs`.

## Сборка в Windows

Для сборки под Windows вы должны использовать Visual Studio 2013, ту же FreeCAD.
использует. Установите CMake и Python. Если вы собираете 64-битную версию, сделайте
обязательно установите 64-разрядную версию Python. Я только тестировал сборку с
Python 2.7.14 64-бит. Вероятно, вы можете использовать python lib, включенный в FreeCAD
libpack, добавив путь libpack к переменной окружения PATH. Но это
у меня как-то не работает. CMake нашел только отладочную версию python lib в
пакет libpack.

Загрузите и извлеките последний [swig] (http://www.swig.org/download.html) в
some where, и добавьте путь к переменной окружения PATH. Я не тестировал
для сборки со старой версией swig, которая идет в комплекте с FreeCAD libpack.

Обязательно проверьте все подмодули slv перед сборкой. Никто из них
фактически используется, но по-прежнему необходим для проверки зависимостей CMake,

```
cd asm3/slvs
git submodule update --init --recursive
```

Запускаем CMake-gui, выбираем директорию сборки. Добавьте запись типа BOOL с именем
BUILD_PYTHON и установите для него значение true. Затем нажмите `configure` и выберите Visual
Studio 2013 Win64, который использовала FreeCAD. Если все прошло без ошибок, нажмите
`генерировать`.

Наконец, откройте файл `solvespace.sln` в каталоге сборки. Вам нужно только
построить два проекта, сначала `slvs_static_excp`, а затем` _slvs`. После завершения
скопируйте вывод из следующего места в `asm / py_slvs`

```
asm/slvs/<your_build_directory>/src/swig/python/slvs.py
asm/slvs/<your_build_directory>/src/swig/python/Release/_slvs.pyd
```

Если вы хотите создать версию Debug, либо загрузите библиотеки отладки Python,
или поместите каталог libpack FreeCAD в переменную окружения PATH перед
настройка CMake, чтобы CMake мог найти библиотеку Python отладочной версии.
После сборки вы должны переименовать _slvs.pyd в _slvs_d.pyd перед копированием в
`asm / py_slvs`

## Сборка для MacOS

Бинарный файл предварительной сборки для MacOS находится в [другом] (../ tree / master / py_slvs_mac)
субмодуль, потому что расширение Python для MacOS имеет то же имя, что и расширение Linux. Чтобы
Соберите его самостоятельно для использования в пакете приложений FreeCAD, сначала вам нужно настроить `Homebrew`
согласно этой [вики] (https://www.freecadweb.org/wiki/CompileOnMac), и
собрать пакет приложений FreeCAD.

Предполагая, что вы установили пакет FreeCAD в `~ / some / place / FreeCAD.app`, затем клонируйте
Репозиторий Assembly3 в `~ / some / place / FreeCAD.app / Contents / Ext / freecad /`. И
очень важно, убедитесь, что вы назвали каталог клона как __asm3__. После этого
подмодуль checkout `slvs` и все его собственные подмодули.
```
cd ~/some/place/FreeCAD.app/Conntents/Ext/freecad/asm3
git submodule update --init slvs
cd slvs
git submodule update --init --recursive
```

Используйте следующую команду для настройки и сборки

```
mkdir build
cd build
cmake \
-DCMAKE_BUILD_TYPE=Release \
-DBUILD_PYTHON=1 \
-DPYTHON_EXECUTABLE:FILEPATH=/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/bin/python2.7 \
-DPYTHON_INCLUDE_DIR=/usr/local/opt/python@2/Frameworks/Python.framework/Headers/ \
-DPYTHON_LIBRARY=/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/lib/libpython2.7.dylib  \
-DPython_FRAMEWORKS=/usr/local/opt/python@2/Frameworks/Python.framework/ ..

make _slvs
```

После этого создайте каталог с именем py_slvs_mac в asm3 и скопируйте его.
результаты

```
cd ~/some/place/FreeCAD.app/Conntents/Ext/freecad/asm3
mkdir py_slvs_mac
touch py_slvs_mac/__init__.py
cp slvs/build/src/swig/python/_slvs.so py_slvs_mac/
cp slvs/build/src/swig/python/slvs.py py_slvs_mac/
```

Наконец, вы должны сделать `_slvs.so` перемещаемым, чтобы иметь возможность загружать его в пакет FreeCAD, с помощью следующей команды

```
cd py_slvs_mac
install_name_tool -id "_slvs.so" _slvs.so
install_name_tool -add_rpath "@loader_path/../../../../lib/" _slvs.so
install_name_tool -change \
    "/usr/local/opt/python@2/Frameworks/Python.framework/Versions/2.7/Python" "@rpath/Python" _slvs.so
```
Последняя команда изменяет путь к связанной библиотеке относительно динамического загрузчика библиотеки пакета. Если вы использовали другую конфигурацию CMake, вы можете узнать путь к вашей связанной библиотеке, используя следующую команду

```
otool -L _slvs.so
```

Готово, и вы можете запустить FreeCAD.app и попробовать Assembly3.

# SymPy + SciPy

Другой бэкэнд решателя ограничений использует [SymPy] (http://www.sympy.org/) и [SciPy] (https://www.scipy.org/). В основном они основаны на Python с некоторым собственным ускорением в определенных критических частях. Модели реализации бэкэнда соответствуют дизайну решателя SolveSpace, то есть символьная алгебраическая + нелинейная минимизация по методу наименьших квадратов. Его можно рассматривать как реализацию на Python решателя SolveSpace.

SciPy предлагает десяток различных [минимизация] (https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.minimize.html)
алгоритмы, но большинство из них не могут конкурировать с SolveSpace по производительности.
В следующем списке показаны некоторые результаты неформального тестирования с использованием параметров по умолчанию в примере [assembly] (# create-a-super-assembly-with-external-link-array), описанном позже.

| Алгоритм | Время |
| --------- | ---- |
| SolveSpace (как ссылка) | 0,006 с |
| Нелдер-Мид | Не сходятся |
| Пауэлл | 7,8 с |
| CG | 10 + 37s <sup> [1] (# f1) </sup> |
| BFGS | 10 + 0.8 <sup> [1] (# f1) </sup> |
| Ньютон-CG | 10 + 61 + 0.5s <sup> [2 ](#f2),</sup> <sup> [3] (# f3) </sup> |
| L-BFGS-B | 10 + 1.5s <sup>[1ght(#f1),</sup> <sup> [3] (# f3) </sup> |
| TNC | 10 + 0.8s <sup> [1] (# f1) </sup> |
| COBYLA | 0,2 с <sup> [3] (# f3) </sup> |
| SLSQP | 10 + 0.3 <sup> [1 ](#f1),</sup> <sup> [3] (# f3) </sup> |
| изогнутый | 10 + 61 +? S <sup> [2] (# f2) </sup> Не удалось решить, ошибка linalg |
| траст-нкг | 10 + 61 + 1.5s <sup> [2] (# f2) </sup> |

<b name = 'f1'> [1] </b> Включая вычисление матрицы Якоби (10 в этом тестовом примере), которое реализовано с использованием sympy lambdify с numpy.

<b name = 'f2'> [2] </b> Включая вычисление матрицы Гессе (61 сек в данном тестовом примере) в дополнение к матрице Якоби.

<b name = 'f3'> [3] </b> Полученное решение содержит небольшие зазоры в некоторых точках ограничения совпадений. Неправильное использование алгоритма?

Причины написания этого бэкенда:

* SolveSpace находится под лицензией GPL, что несовместимо с FreeCAD LGPL,
* Чтобы получить больше информации о системе решателя и легко экспериментировать с новыми идеями из-за его природы, основанной на Python,
* Возможно, для будущего расширения, моделирования на основе физики?

Вам нужно будет установить SymPy и SciPy для вашей платформы.

```
pip install --upgrade sympy scipy
```

