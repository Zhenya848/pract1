Отчёт по практическому занятию №1

Цель работы

Научиться выполнять простые действия с файлами и каталогами в Linux из командной строки и сравнивать работу командной строки Windows и Linux.

Задача 1.

Решение

grep -Eo '^[^:]+' /etc/passwd | sort

Результат

_apt
backup
bin
_chrony
daemon
dhcpcd
fwupd-refresh
games
irc
landscape
list
lp
mail
man
messagebus
news
nobody
polkitd
pollinate
proxy
root
sshd
sync
sys
syslog
systemd-network
systemd-resolve
tcpdump
tss
uucp
uuidd
www-data

Задача 2. 

Решение

awk '$1 !~ /^#/ && NF >= 2 && $2 ~ /^[0-9]+$/ { print $2, $1 }' /etc/protocols \
    | sort -nr -k1,1 \
    | head -n 5

Результат проверки

262 mptcp
143 ethernet
142 rohc
141 wesp
140 shim6

Задача 3.

Решение

if (( $# == 0 )); then
    printf 'Использование: %s "текст"\n' "$(basename "$0")" >&2
    exit 1
fi

text=$*
width=$(( ${#text} + 2 ))
border=$(printf '%*s' "$width" '' | tr ' ' '-')

printf '+%s+\n' "$border"
printf '| %s |\n' "$text"
printf '+%s+\n' "$border"

Пример запуска

./banner "Hello from RTU MIREA!"

Результат

+-----------------------+
| Hello from RTU MIREA! |
+-----------------------+

Задача 4. 

Решение

if (( $# != 1 )); then
    printf 'Использование: %s <файл>\n' "$(basename "$0")" >&2
    exit 1
fi

LC_ALL=C grep -Eo '[A-Za-z_][A-Za-z0-9_]*' "$1" | sort -u

Задача 5. 

Решение

if (( $# != 1 )); then
    printf 'Использование: %s <программа>\n' "$(basename "$0")" >&2
    exit 1
fi

src=$1
if [[ ! -f $src ]]; then
    printf 'Ошибка: файл не найден: %s\n' "$src" >&2
    exit 1
fi

name=$(basename "$src")
chmod 755 "$src"
sudo install -m 755 "$src" "/usr/local/bin/$name"
printf 'Команда зарегистрирована: /usr/local/bin/%s\n' "$name"

Задача 6. 

Решение

dir=${1:-.}

if [[ ! -d $dir ]]; then
    printf 'Ошибка: каталог не найден: %s\n' "$dir" >&2
    exit 1
fi

find "$dir" -type f \( -name '*.c' -o -name '*.js' -o -name '*.py' \) -print0 |
while IFS= read -r -d '' file; do
    first=$(head -n 1 -- "$file" || true)
    ext=${file##*.}
    has_comment=0

    case "$ext" in
        c|js)
            [[ $first =~ ^[[:space:]]*(//|/\*) ]] && has_comment=1
            ;;
        py)
            [[ $first =~ ^[[:space:]]*# ]] && has_comment=1
            ;;
    esac

    if (( has_comment == 0 )); then
        printf '%s\n' "$file"
    fi
done

Задача 7.

Решение

#!/usr/bin/env bash
set -euo pipefail

dir=${1:-.}

if [[ ! -d $dir ]]; then
    printf 'Ошибка: каталог не найден: %s\n' "$dir" >&2
    exit 1
fi

find "$dir" -type f -print0 |
    xargs -0 -r sha256sum |
    sort -k1,1 |
    awk '
    function print_group() {
        if (count > 1) {
            printf "Хэш: %s\n", hash
            for (i = 1; i <= count; ++i) printf "  %s\n", files[i]
            printf "\n"
        }
    }
    {
        if ($1 != hash) {
            print_group()
            hash = $1
            count = 0
            delete files
        }
        line = $0
        sub(/^[^ ]+[ ]+/, "", line)
        files[++count] = line
    }
    END { print_group() }'

Задача 8.

Решение

if (( $# < 2 || $# > 3 )); then
    printf 'Использование: %s <каталог> <расширение> [архив.tar]\n' "$(basename "$0")" >&2
    exit 1
fi

dir=$1
ext=$2
archive=${3:-archive.tar}
ext=${ext#.}

if [[ ! -d $dir ]]; then
    printf 'Ошибка: каталог не найден: %s\n' "$dir" >&2
    exit 1
fi

archive_abs=$(readlink -f -- "$archive")

if ! find "$dir" -type f -name "*.$ext" -print -quit | grep -q .; then
    printf 'Файлы с расширением .%s не найдены.\n' "$ext" >&2
    exit 1
fi

(
    cd -- "$dir"
    find . -type f -name "*.$ext" -print0 | tar --null -cf "$archive_abs" --files-from=-
)
printf 'Создан архив: %s\n' "$archive_abs"

Пример

./archive_by_ext.sh ./data txt txts.tar

Результат теста

В тестовом каталоге были найдены два .txt-файла. В архив попали:

./b.txt
./a.txt

Задача 9. 

Решение

if (( $# != 2 )); then
    printf 'Использование: %s <входной_файл> <выходной_файл>\n' "$(basename "$0")" >&2
    exit 1
fi

sed 's/    /\t/g' "$1" > "$2"

Проверка

Для строки:

a    b      c

результат:

a\tb\t  c

Задача 10.

Решение

dir=${1:-.}

if [[ ! -d $dir ]]; then
    printf 'Ошибка: каталог не найден: %s\n' "$dir" >&2
    exit 1
fi

find "$dir" -type f -empty -print

Результат теста

/mnt/data/pract1_test/task10/empty.bin
/mnt/data/pract1_test/task10/empty.txt
/mnt/data/pract1_test/task10/sub/empty.py
