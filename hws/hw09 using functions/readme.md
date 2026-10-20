# Функции в Bash

## Описание
Домашнее задание по дисциплине "Операционные системы и среды". Цель работы — освоить создание и использование функций в Bash: передача аргументов, локальные переменные, код возврата, перенаправление вывода, команда `eval`. Работа выполняется в гостевой ОС Debian (VirtualBox), в терминале.

## Что понадобится
- Oracle VirtualBox с импортированным образом ВМ.
- Терминал ВМ, вход как root (или `sudo`).
- Word/PDF для оформления отчёта.

## Вопрос: чем отличаются exit и return?
`return` завершает работу функции и при необходимости передаёт код возврата (0–255). `exit` завершает работу всей программы (скрипта/оболочки), а не только функции.

## Задание 1. Функция вывода текста в заданный поток

Функция принимает текст и «место»:
- `1` — стандартный поток вывода (stdout);
- `2` — стандартный поток ошибок (stderr);
- иное значение — имя файла для дозаписи.

```bash
#!/bin/bash

textToStreams() {
    local txt="$1"
    local place="$2"

    if [ "$place" = "1" ]
    then
        echo "$txt"
    elif [ "$place" = "2" ]
    then
        echo "$txt" >&2
    else
        echo "$txt" >> "$place"
    fi

    return 0
}

textToStreams "$1" "$2"
```

Проверка:
```bash
touch staff.txt
./funcTask1.sh 'meow meow meow' 1
./funcTask1.sh 'meow meow meow' 2
./funcTask1.sh 'meow meow meow' staff.txt
cat staff.txt
```

## Задание 2. Функция вывода информации о системе

Функция принимает описание параметра (`pwd`, `user`, `version`, `date`) и выводит соответствующую информацию. Неизвестный параметр — сообщение об ошибке и код возврата 1.

```bash
#!/bin/bash

sysInfo() {
    local param="$1"

    if [ -z "$param" ]
    then
        echo "Function needs option"
        return 1
    fi

    case "$param" in
        "pwd")
            echo -n "Current directory: "
            pwd
            ;;
        "user")
            echo -n "Current user: "
            whoami
            ;;
        "version")
            echo -n "Kernel version: "
            uname -a
            ;;
        "date")
            echo -n "Current date: "
            date
            ;;
        *)
            echo "Unknown option. Available options: pwd, user, version, date"
            return 1
            ;;
    esac

    return 0
}

echo "--Directory option test--"
sysInfo 'pwd'
echo
echo "--User option test--"
sysInfo 'user'
echo
echo "--Version option test--"
sysInfo 'version'
echo
echo "--Date option test--"
sysInfo 'date'
echo
echo "--Error test--"
sysInfo 'hahaha'
```

## Задание 3. Функция вывода значения переменной по имени (eval)

Функция использует `eval` для косвенного обращения к переменной. Программа читает имена переменных из stdin до `Ctrl + D` (EOF).

```bash
#!/bin/bash

printVal() {
    local var="$1"
    eval "echo \${$var}"
}

echo "Input variable name for output (Ctrl + D for exit):"

name="Leo"
breed="stupid orange cat"
os="Not Hannah Montana Linux"
time="it's late."
titan="attack"
singer="Ado"

while read -r line
do
    if [ -n "$line" ]
    then
        printVal "$line"
    fi
done

echo "Programm ended"
```

Проверка: запустить скрипт, вводить имена (`name`, `breed`, `os`, `time`, `titan`, `singer`), затем `Ctrl + D`.

## Замечания по выполнению
- В Bash в определении функции аргументы в скобках не указываются: `имя() { ... }`. Вызов — по имени без скобок, аргументы через пробел.
- Внутри функции аргументы доступны как `$1`, `$2`, ... — как у скрипта.
- `local` объявляет локальную переменную, видимую только внутри функции.
- `return N` задаёт код возврата функции (не значение!). Вернуть строку через `return` нельзя — только через `echo` и подстановку.
- `eval "echo \${$var}"` — косвенное обращение: сначала подставляется имя переменной, затем `eval` выполняет получившуюся команду `echo ${имя}`.
- Перенаправления: `>` — в stdout, `>&2` — в stderr, `>> файл` — дозапись в файл.
- Не забывайте давать скриптам права на исполнение: `chmod +x script.sh`.
- В отчёте на скриншоте функции из задания 1 (`textToStreams "$1" "$2"`) виден вызов, но не виден сам запуск — для проверки лучше явно показать `./funcTask1.sh ...` в терминале.

## Требования к отчёту
- Оформляется в Word/PDF.
- Каждый шаг фиксируется: код + скриншот выполнения.
- В конце — вывод: какие знания извлечены, какие приёмы работы с функциями освоены.

## Файлы
- `ДЗ8. Функции.pdf` — методичка
- `Отчет функции в Bash.docx` — выполненный отчёт