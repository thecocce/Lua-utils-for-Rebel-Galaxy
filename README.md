# Lua-utils-for-Rebel-Galaxy

## требования
* Lua 5.3 (http://www.lua.org/download.html)
* lua-zlib (https://github.com/brimworks/lua-zlib)
* lua-lfs (https://github.com/keplerproject/luafilesystem)

для Win x64 имеется готовый комплект, [lua531_zlib_lfs_x64.zip](https://mega.nz/#!mxgGkYYY!SFJRF2feU_DdMu7L8dH-Syww-uUiO4_qbzDNLFstY8E).

## использование

### работа с .DAT архивами

#### распаковка
````
lua unpack.lua <INPUT.PAK> [OUTPUT/dir [FILTER]]
````
*прим* выходной каталог должен существовать. без указания происходит вывод содержания архива без распаковки. при указании FILTER распаковываются только ресурсы заданного типа:

````
1  - MESH/MDL    2 - SKELETON    3 - DDS         4 - PNG/TGA/BMP
6  - OGG/WAV     9 - MATERIAL   10 - RAW        12 - IMAGESET
13 - TTF        15 - DAT        16 - LAYOUT     17 - ANIMATION
24 - PROGRAM    25 - FONTDEF    26 - COMPOSITOR 27 - FRAG/FX/HLSL/VERT
29 - PU         30 - ANNO       31 - SBIN       32 - WDAT
````

к именам всех файлов добавляется цифровой префикс вида *XX_имяфайла*, где *XX* — внутригровой тип ресурса. это требуется для скрипта упаковки, чтобы не хранить тип в отдельном файле.

#### упаковка
````
lua pack.lua <INPUT/dir> [OUTPUT.PAK]
````
*прим* имя создаваемого архива по умолчанию — *DATA2.PAK* в текущем каталоге. на первом уровне входного каталога должен находится только каталог *MEDIA*.

### ресурсы

#### конвертер DAT -> Lua -> DAT
````
lua dat2lua.lua <INPUT.DAT> [OUTPUT.lua]
lua lua2dat.lua <INPUT.lua> [OUTPUT.DAT]
````
*прим* без указания второго параметра *dat2lua* направляет вывод в консоль, *lua2dat* — в *OUT.DAT* в текущем каталоге.

конвертация происходит в валидный Lua-скрипт, который и используется для обратного преобразования. в начале файла находится строковый словарь, остальные строковые значения берутся из хеш–таблиц *dict_(types|ext).lua*. в случае отсутствия в словаре хеш записывается как *_XXXXXX*, где *XXXXXX* — его числовое значение.

аналогично можно конвертировать файлы с типами 12 (IMAGESET) и 17 (ANIMATION).

#### конвертер DAT -> TXT|XML
````
lua dat2txt.lua <INPUT.DAT> [DEBUG] [> OUTPUT.txt]
lua dat2xml.lua <INPUT.DAT> [> OUTPUT.xml]
````

### проверка хешей
````
lua hasher.lua <STRING1> [STRING2 [STRING3 [...]]
````
подсчет хеша для строки (строк) STRINGx и его вывод (десятичное значение, строка, шестнадцатеричное значение):
````
>lua hasher.lua STRING1 1STRING
1829089533, "STRING1", --0x6D05B0FD
3815255475, "1STRING", --0xE3682DB3
````

### подбор хеша
````
lua hash_brut.lua <HASH> [STRING] [DEPTH]

    HASH   : требуемый хеш
    STRING : неизменяемые начальные символы или пустая строка ("")
    NUMBER : глубина (т. е. количество перебираемых символов)
````
словарь по умолчанию — цифры, латинские буквы в верхнем регистре и символ подчеркивания (````0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ_````).

#### примеры
поиск хеша в строке длиной шесть символов начинающейся с ````CHA````:
````
>lua hash_brut.lua 44150820 CHA 3
44150820 CHANCE
keys found: 1
````

полный перебор всех четырехсимвольных комбинаций:
````
>lua hash_brut.lua 7057780 "" 4
7057780 UNJ4
7057780 UNIT
keys founded: 2
````
тут случилась коллизия, так как использованный алгоритм хеширования достаточно неустойчивый.

## примечания

````conflict_ids.lua```` — список идентификаторов, которые используются как в STRING, так и в TRANSLATE типах. поэтому эти строки нельзя переводить (конвертировать в Unicode). некоторые такие строки даже использованы в именах переменных в скриптах...