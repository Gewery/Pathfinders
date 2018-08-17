To run it on your own computer you should install 2 modules:
- Flask (pip install flask)
- configparser (pip install configparser)

Also your .py file should be in the "players" and in the root of a game (folder "Pathfinders")



Programming the robot

To add a robot, just put the program file in the directory `players` (you can change the directory in the config file). Requirements for the program:

1. The file must have the extension `.py`
2. The file must contain a syntactically correct python code.
3. A function with two arguments `move (info, ctx)`

If all three conditions are met, then when the game starts, the player will appear whose name will be the same as the file name (without `.py`). Moving the robot at each move is specified using the `move (info, ctx)` function. The function must return a value from 0 to 3, which indicates the direction of the next step of the robot:
 
* 0 - up
* 1 - down
* 2 - left
* 3 - right

If the function returns any other value, the robot will remain in place on this move, but it can continue to move on subsequent moves. If an error occurs in the function (an exception is thrown), the robot will remain in place until the end of the game.

To calculate the next move, the function can use information about the current game situation. Information about the map of the labyrinth, the location of coins and rivals is contained in the variable info, which is the first argument of the function. info is a dictionary that can contain the following fields:
 
* `info [" x "]` - the current coordinate of the robot on the playing field horizontally (measured from the left starts at 0)
* `info [" y "]` - similarly vertically (counted from the top starts at 0)
* `info [" coins "]` - coordinates of coins. The list of tuples (x, y), for example `info [" coins "] = [(1, 3), (3, 0), (0, 4)]`
* `info [" players "]` - coordinates of other players. The list of tuples (x, y), for example `info [" playes "] = [(2, 2)]`
* `info [" map "]` - a map of the labyrinth. Presented as a two-dimensional array of 0 and 1. 1 - a wall, 0 - a free field.

All coordinates are counted from the upper left corner of the field starting from (0, 0). In subsequent editions of the game, the composition of the fields in the dictionary `info` can vary.

It is necessary to note the device of the card separately. A map is a two-dimensional array, that is, a list of lists. For example:

`` `python
info ["map"] = [
[0, 0, 0, 1, 0],
[0, 0, 0, 0, 0],
[0, 0, 0, 0, 0],
[1, 1, 0, 1, 0]
]
`` `
The map is a list of "strings", so to find out what is at the x, y coordinate, you need to refer to the element info ["map"] [y] [x] `, not to the info [" map "] [x] [y] `.

The second function argument is an empty dictionary `ctx` into which you can write arbitrary fields. The next time the function is called, these data will appear in the dictionary `ctx`. In this way, it can be used to store the results of calculations between subsequent calls to the `move ()` function.

## Examples of robot control programs

### "Always up"

`` `python
def move (info, ctx):
    return 0 # Move upwards always
`` `

### "Observer"

`` `python
def move (info, ctx):
    # Do nothing, we print a map and a list of treasures
    print(info ["map"])
    print(info ["coins"])
    # The robot does not move anywhere, will ignore the wrong value
    return None
`` `

### "The buggy robot"

`` `python
def move (info, ctx):
    return 10/0
`` `
After the start of the game, an error occurs.
`` `
ERROR. Process 'robot' on move function. integer division or modulo by zero.
`` `
The robot does not move anymore, but for the rest the game continues.

### "EGGOG"

Invalid Python code: no colon, tab.

`` `python
def move (info, ctx)
return 0
`` `
The game does not start with an error:
`` `
ERROR: Error loading player robot: invalid syntax (robot.py, line 3)
`` `
It is necessary either to correct the error, or to delete the file with an error from the directory with the programs of the robots.

### "Random walks"

In the `players` directory there are already two examples of programs for robots: chuck and rocky. In both cases, the direction of the next move is chosen randomly. But in the chuck algorithm, it is checked that the corresponding move is possible. Pay attention to the results of the competitions of the two algorithms.

## Bugs and features

* The game does not limit the number of players, but the web interface can not display more than 7 robots on the field. The restriction is associated with the patrol color for robots - there are now only 7 colors. Can be corrected if necessary.
* The game must be terminated by Ctrl-C in the launch console. In very rare cases, the flask does not end and the program hangs. The process must be killed with SIGTERM (kill)
* The full path to the pathfinders directory can not contain Russian letters. Otherwise, Flask can give an error when loading data from the static directory. `# - * - coding: utf-8 - * -` does not help. As a workaround, you need to move the game to a directory that does not contain Russian letters.
* On Windows, the current code for importing player modules is not working correctly. Python looks for modules in the standard path path and in the main game directory. As a workaround, you need to place the module with the player's program in the `players` directory and also a copy to the main directory next to the game. Let's investigate the problem, maybe fix.
* Robots always made a move in a certain order, which was determined when modules were loaded. The problem is fixed in the current version.



# pathfinders

Игра про поиск сокровищ в лабиринте. По лабиринту раскиданы золотые монеты, которые собирают полностью автоматические роботы. Задача игроков - написать программу управления роботом для сбора монет. Надо помнить, что другие роботы не стоят на месте и тоже ищут сокровища. Побеждает тот, чей робот собрёт больше монет.

Оригинальная идея игры заимствована из проекта [mipt-cs-labyrinth-challenge](https://github.com/avasyukov/mipt-cs-labyrinth-challenge)

## Как запустить игру?

Для работы игра требует python3 и [Flask](http://flask.pocoo.org/) для web интерфейса.

1. Склонировать репозитарий из Github
2. Запустить из командной строки 

```
$ ./pathfinders.py 
Process 'gosha' started
Process 'masha' started
...
```

3. Открыть в браузере [http://127.0.0.1:5000/](http://127.0.0.1:5000/) На странице должен отобразится состояние игрового поля.
4. Для завершения игры необходимо завершить процесс `pathfinder.py` в консоли при помощи Ctrl-C

## Принципы игры

При старте приложения игра загружает карту лабиринта (настраивается в конфигурационном файле) и случайным образом раскидывает сокровища по карте. Из директории с программами для роботов загружаются все файлы с расширением .py. Каждый файл - программа для отдельного робота. Путь к файлу карты и директории с программами игроков настраиваемся в конфигурационном файле.

Программа управления роботом (далее просто "программа") является обычным python файлом с расширение .py. Имя игрока (в web интерфейсе) будет таким же как и имя файла программы. В программе должна быть обязательно определена функция `move(info, ctx)`. Это основная функция управления роботом. В неё передается текущая информация о карте, расположении сокровищ и других игроков. Функция должна вернуть число от 0 до 3, которое обозначает направление следующего хода. Подробности о написании программ управления роботом можно узнать из соответсвующего раздела. Начальные положения роботов задаются случайно.

Игра вызывает программу управления роботом через определенные интервалы времени. Если функция `move()` не успеет выполнить нужные расчеты за указанное время, то робот остается не месте и никуда не двигается на текущем ходу. Если в программе робота случается ошибка, то он выходит из строя до конца игры и больше не может собирать сокровища.

В некоторых случаях некорректно написанная программа управления роботом не позволит игре запуститься. Например, если в файле программы не определена функция move(). В этом случае надо либо исправить ошибку в программе робота, либо удалить файл глючной программы. 

## Интерфейс

Web интерфейс доступен после запуска игры по адресу [http://127.0.0.1:5000/](http://127.0.0.1:5000/). Отображается игровое поле и таблица результатов. Единственное неочевидное поле в таблице - timeout. Это количество ходов, которое робот пропустил из-за того, что функция `move()` выполнялась дольше необходимого лимита.

## Настройки игры

Все настройки игры хранятся в файле `config`. Его нельзя переименовывать и перемещать. В файле конфигурации две секции: `pathfinders` и `web`. Секция `pathfinders` содержит настройки самой игры.

* `map_file` - путь к файлу с картой. По соглашению карты лежат в директории `maps`.
* `move_timeout` - количество секунд на 1 ход для каждого игрока. Может быть дробным.
* `players` - директория с программами для роботов. Менять не нужно.
* `num_coins` - количество монет на карте. Монеты располагаются на карте в случайном порядке.
* `max_moves` - максимальное количество ходов в игре. После достижения максимального количество ходов игра прекращается даже если ещё остались монеты.

Секция `web` содержит только один параметр: `field_size` - размер клетки игрового поля в пикселях в web интерфейсе. Если поле не вмещается на один экран этот параметр можно уменьшить.

После изменения параметров игры необходимо перезапустить игру и, в ряде случаев, обновить интерфейса страницу в браузере.

## Собственные карты

Составлять собственные карты легко. Необходимо создать файл с картой (лучше в директории `maps`) и прописать путь к файлу карты в конфигурационный файл игры.

Файл игры выглядит так:
```
#......#........#
#...###...####.#.
....#.....#..#...
....#.......#...#
..#....##........
.......#....###..
.......#....#....
.......#....#....
....####.........
```

`.` - пустое поле
`#` - стена

После изменения карты в конфигурационном файле следует не только перезапустить игру, но и обновить страницу в браузере. 

## Программирование робота

Для добавления робота достаточно положить файл программы в директорию `players` (Директорию можно изменить в конфиге). Требования к программе:

1. Файл должен иметь расширение `.py`
2. Файл должен содержать синтаксически верный python код.
3. В файле должна быть определена функция с двумя агрументами `move(info, ctx)`

Если все три условия соблюдены, то при запуске игры появится игрок имя которого будет совпадать с именем файла (без `.py`). Передвижение робота на каждом ходе задается при помощи функции `move(info, ctx)`. Функция должна возвращать значение от 0 до 3, которое обозначает направление следующего шага робота:
 
* 0 - вверх
* 1 - вниз
* 2 - влево
* 3 - вправо

Если функция вернет любое другое значение, то робот останется на месте на этом ходу, но может продолжить движения на последующих ходах. Если в функции возникнет ошибка (вызовется исключение), то робот до конца игры останется на месте.

Для вычисления следующего хода функция может использовать информацию о текущей игровой ситуации. Информация о карте лабиринта, расположении монет и соперников содержится в переменной info, который является первым аргументом функции. info - это словарь, который может содержать следующие поля:
 
* `info["x"]` - текущая координата робота на игровом поле по горизонтали (отсчитывается слева начинается с 0)
* `info["y"]` - аналогично по вертикали (отсчитывается сверху начинается с 0)
* `info["coins"]` - координаты монет. Список tuples (x, y), например `info["coins"] = [(1, 3), (3, 0), (0, 4)]`
* `info["players"]` - координаты других игроков. Список tuples (x, y), например `info["playes"] = [(2, 2)]`
* `info["map"]` - карта лабиринта. Представлена в виде двумерного массива из 0 и 1. 1 - стена, 0 - свободное поле.

Все координаты отсчитываются от верхнего левого угла поля начиная с (0, 0). В последующих редакциях игры состав полей в словаре `info` может меняться.

Необходимо отдельно отметить устройство карты. Карта это двумерный массив, то есть список списков. Например:

```python
info["map"] = [
[0, 0, 0, 1, 0],
[0, 0, 0, 0, 0],
[0, 0, 0, 0, 0],
[1, 1, 0, 1, 0]
]
```
Карта - это список "строк", поэтому чтобы узнать, что находится в точке с координатами x, y необходимо обратиться к элементу `info["map"][y][x]`, а не к `info["map"][х][y]`.

Второй агрумент функции - пустой словарь `ctx` в который можно записывать произвольные поля. При следующем вызове функции эти данные стануться в словаре `ctx`. Такми образом его можно использовать для хранения результатов вычислений между последующими вызовами функции `move()`.

## Примеры программ управления роботом

### "Всегда вверх"

```python
def move(info, ctx):
    return 0 # Движемся вверх всегда
```

### "Наблюдатель"

```python
def move(info, ctx):
    # Ничего не делаем печатаем карту и список сокровищ
    print(info["map"])
    print(info["coins"]) 
    # Робот никуда не двинется, проигнорирует ошибочное значение
    return None
```

### "Глючный робот"

```python
def move(info, ctx):
    return 10/0
```
После старта игры вываливается ошибка.
```
ERROR. Process 'robot' on move function. integer division or modulo by zero.
```
Робот больше не двигается, но для остальных игра продолжается.

### "ЕГГОГ"

Некорректный код на Python: нет двоеточия, табуляция. 

```python
def move(info, ctx)
return 0
```
Игра не запускается с ошибкой:
```
ERROR: Error loading player robot: invalid syntax (robot.py, line 3)
```
Необходимо или поправить ошибку, или удалить файл с ошибкой из директории с программами роботов.

### "Случайные блуждания"

В диретории `players` уже есть два примера программ для роботов: chuck и rocky. В обоих случаях направление следующего хода выбирается случайно. Но в алгоритме chuck при этом проверяется то, что соответсвующий ход возможен. Обратите внимание на результаты соревнований двух алгоритмов.

## Баги и особенности

* Игра не ограничивает количество игроков, но web интерфейс не может отобразить более 7 роботов на поле. Ограничение связано с патитрой цветов для роботов - там сейчас только 7 цветов. Можно поправить при необходимости.
* Игра должна завершаться по Ctrl-C в консоли запуска. В очень редких случаях flask не завершается и программа подвисает. Процесс необходимо убивать при помощи SIGTERM (kill)
* Полный путь к директории pathfinders не должен содержать русских букв. Иначе Flask может выдавать ошибку при загрузке данных из директории static. `# -*- coding: utf-8 -*-` не помогает. В качестве workaround надо переместить игру в директорию не содержащую русских букв.
* На Windows некорректно отрабатывает текущий код импорта модулей игроков. Python ищет модули по стандартному пути path и в основной диретории игры. В качестве workaround нужно поместить модуль с программой игрока в директорию `players` а также копию в основную директорию рядом с игрой. Исследуем проблему, возможно будет фикс.
* Роботы всегда совершали ход в определенном порядке, который определялся при загрузке модулей. Проблема поправлена в текущей версии.
* Если робот появлялся на клетке с монетой, то он не её на забирал. Поправлено в текущей версии.

Сообщить о найденных неполадках можно по email mihkulemin(at)gmail.com
