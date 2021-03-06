# краткое пособие для переводчика

## введение

в игре используется два основных строковых типа:
* STRING — строки в однобайтовой кодировке.
* TRANSLATE - строки в кодировке UTF8.

при наличии TTF или растровых шрифтов с корректной поддержкой юникодной кириллицы строки с типом STRING могут быть использованы для отображения русских букв при использовании кодировки CP-1251. тип TRANSLATE также зависит от шрифтов, никакого механизма fallback не предусмотрено, при отсутствии нужного глифа происходит CTD.

основная и, практически, единственная проблема — совпадение имен многих скриптовых переменных с обычными текстовыми ресурсами. к примеру, строка "CARGO" одновременно является внутриигровым типом, идентификатором элемента интерфейса и, собственно, "ГРУЗОМ"; однако перевод этой строки использовать невозможно. в качестве TODO можно для всех подобных строк изменить идентификатор на любой еще неиспользуемый чтобы разделить строку-перевод и строку-все-остальное во внутреннем словаре (пока только вручную).

## подготовка

отредактировать в *0-include.cmd* переменные *RGDIR* (путь к установленной игре) и *WORK* (путь к рабочему каталогу, в котором и будет происходить распаковка/редактирование). остальные переменные используются в следующих пакетных файлах и изменению не подлежат.

## распаковка

1. запустить *1-unpack_DAT.cmd*. в рабочем каталоге появятся подкаталоги *unpack_** с распакованными **.DAT*, **.IMAGESET* и **.ANIMATION*. 
2. запустить *2-generate_strings_list.cmd*. после сканирования распакованных ресурсов должны появится файлы:
 * *list_all_string.txt* — символьно отсортированный список со строками вида ````index, type, "string"````, где type равен S(tring) или T(ranslate).
 * *lang_original.lua* — пара таблиц с индексной сортировкой по всем строковым значениям (по умолчанию взятым только из *.DAT файлов).
 * **list_conflicted_ids.lua** — список *конфликтых* строк, которые одновременно представлены с типом STRING и TRANSLATE. то есть эти строки напрямую переводить (и вообще изменять их) категорически нельзя.
 * *list_duplicated_ids.lua* — список строк с одинаковыми идентификатором, но разными значениями. в оригинальной игре таких не наблюдается.
 * *lang_original.pbm* — графическое представление заполнения строкового словаря. черная точка отображает отсутствующий (неизвестный) ключ, белая — имеющийся.
3. запустить *3-export_DAT.cmd*. начнется экспорт всех ранее распакованных ресурсов (из п. 1), на выходе кроме них появится
* *list_missing_ids.txt* — лог всех неизвестных имен переменных в виде их хеша. влияет исключительно на читаемость и понимание экспортированных скриптов. более подробно этот список расписан в [dict_missing.lua](https://github.com/hhrhhr/Lua-utils-for-Rebel-Galaxy/blob/master/dict_missing.lua) основного репозитория.

## работа с переводом

1. создать в рабочем каталоге подкаталог *translate_DAT*, в котором и должны находится переводимые скрипты (скопированные из *unpack_DAT*) с сохранением файловой структуры.
2. переводим-переводим-переводим...
3. запустить *991-strings_check.cmd*. начнется проверка и сравнение измененных строк в каталоге *translate_DAT* с оригиналами из *unpack_DAT*, результатом будет:
 * *t_list_all_string.txt* — промежуточный словарь, не представляет интереса.
 * *t_list_conflicted_ids.txt* — строки, которые нужно вернуть к оригинальному значению (см. *list_conflicted_ids.lua*).
 * **t_list_duplicated_ids.lua** — для нормальной работы **список должен быть пустой**. иначе это список строк, которые имеют одинаковый индентификатор, но разные значения. такое может произойти, если один ID содержится в нескольких файлах и в процессе перевода получились отличающиеся фразы. на каждый конфликт приводится оригинальное значение (закомментаренное) и минимум две строки. этот файл необходимо отредактировать так, чтобы осталась только одна строка с уникальным ID, далее выполняется следующий шаг.
4. запустить *992-update_strings.cmd*. начнется тестовый импорт–экспорт для проверки корректности измененных скриптов. при этом в случае отредактированного файла *t_list_duplicated_ids.lua* из предыдущего шага все перечисленные в нем строки приведутся к одному значению. далее следует повторить **шаг 3** (проверка). если список дублей пустой, то переходим дальше.
если же нет, то нужно найти все оригинальные файлы содержащие упомянутые строки и скопировать их в каталог *translate_DAT*. но только если эти строки действительно можно переводить (см. введени и пример про CARGO). в ином случае в списке дублирующихся строк придется вернуть оригинальные значения и опять повторить **шаг 4**.
5. запустить *993-import_translate.cmd*. все скрипты из каталога *translate_DAT* будут импортированы в каталог *import_DAT* (с предварительной его очисткой) в готовом для архивирования виде.
6. финальная проверка, запустить *994-strings_last_check.cmd*. пойдет проверка уже между оригинальными и импортированными .DAT-файлами. тут важен еще один выходной файл *z_list_duplicated_ids.lua*, который должен быть пустой. иначе опять возвращаемся на **шаг 3**.
7. упаковка, *995-pack_translate.cmd*. все ресурсы из каталога *import_DAT* будут упакованы в архив *DATA_RUS.PAK* и скопированы в каталог игры. там уже должны быть архив с локализованными шрифтами, теперь все готово для запуска игры и тестов.
