# Домашнее задание «Ветвления в Git» - Юрочкин В.А.

## Цель задания
В процессе работы над заданием вы потренеруетесь делать merge и rebase. В результате вы поймете разницу между ними и научитесь решать конфликты.

Обычно при нормальном ходе разработки выполнять rebase достаточно просто. Это позволяет объединить множество промежуточных коммитов при решении задачи, чтобы не засорять историю. Поэтому многие команды и разработчики предпочитают такой способ.

## Ход выполнения

### 1. Подготовка репозитория
Был склонирован пустой репозиторий, создан каталог `branching` и два скрипта с начальным содержимым (вывод всех параметров одной строкой через `$*`).

```
mkdir branching
```
```
cat > branching/merge.sh << 'EOF'
#!/bin/bash
#display command line options

count=1
for param in "$*"; do
    echo "\$* Parameter #$count = $param"
    count=$(( $count + 1 ))
done
EOF
```
```
cp branching/merge.sh branching/rebase.sh
```

Коммит и отправка в main:

git add branching
git commit -m "prepare for merge and rebase"
git push origin main

2. Ветка git-merge
Создана ветка git-merge, в которой последовательно изменялся файл merge.sh.

Первый коммит – замена $* на $@:

bash
# содержимое merge.sh
for param in "$@"; do
    echo "\$@ Parameter #$count = $param"
done
git commit -am "merge: @ instead *"
Второй коммит – переход на цикл while с shift:

bash
# содержимое merge.sh
while [[ -n "$1" ]]; do
    echo "Parameter #$count = $1"
    count=$(( $count + 1 ))
    shift
done
git commit -am "merge: use shift"
Ветка отправлена на GitHub:

bash
git push -u origin git-merge
3. Изменение в main (имитация работы другого разработчика)
Возврат в main и правка rebase.sh – добавлен разделитель и использование $@:

bash
# содержимое rebase.sh
for param in "$@"; do
    echo "\$@ Parameter #$count = $param"
done
echo "====="
git commit -am "main: update rebase.sh"
git push origin main
4. Ветка git-rebase от старого коммита
Найден хеш коммита prepare for merge and rebase (9b5e50d), выполнено переключение на него и создана ветка git-rebase.

Первый коммит (git-rebase 1):

bash
# содержимое rebase.sh
for param in "$@"; do
    echo "Parameter: $param"
done
echo "====="
git commit -am "git-rebase 1"
Второй коммит (git-rebase 2) – замена строки вывода:

bash
echo "Next parameter: $param"
git commit -am "git-rebase 2"
Ветка отправлена:

bash
git push -u origin git-rebase
5. Слияние git-merge в main
Выполнено без конфликтов, так как изменения затрагивали разные файлы.

bash
git checkout main
git merge git-merge
git push origin main
6. Rebase ветки git-rebase на main с объединением коммитов
Запущен интерактивный rebase:

bash
git checkout git-rebase
git rebase -i main
В редакторе для второго коммита указано fixup – оба коммита объединены в один.

Конфликт №1 – при применении первого коммита:

text
<<<<<<< HEAD
    echo "\$@ Parameter #$count = $param"
=======
    echo "Parameter: $param"
>>>>>>> 31a503c... git-rebase 1
Оставлена версия из HEAD (верхняя). После правки:

bash
git add branching/rebase.sh
git rebase --continue
Конфликт №2 – при попытке применить второй коммит (теперь он включается в первый):

text
<<<<<<< HEAD
    echo "\$@ Parameter #$count = $param"
=======
    echo "Next parameter: $param"
>>>>>>> e9df078... git-rebase 2
Оставлена строка из второго коммита (Next parameter...). Далее:

bash
git add branching/rebase.sh
git rebase --continue
Rebase успешно завершён, создан один коммит с объединёнными изменениями.

7. Принудительная отправка переписанной ветки
bash
git push --force-with-lease origin git-rebase
8. Финальное слияние git-rebase в main
После rebase ветка git-rebase находится непосредственно поверх main, поэтому слияние происходит в режиме fast-forward:

bash
git checkout main
git merge git-rebase
git push origin main
Итоговое состояние файлов
branching/merge.sh содержит цикл while с shift (версия из ветки git-merge).

branching/rebase.sh содержит цикл for с выводом Next parameter: $param и разделителем (результат rebase).

Результирующий граф
Итоговая история ветвлений и слияний доступна по ссылке:
https://github.com/victoryurochkin/02-git-03/network

Выводы
merge сохраняет историю ветвления и создаёт дополнительный коммит слияния.

rebase позволяет переписать историю, сделав её линейной, но требует разрешения конфликтов на каждом шаге и осторожного обращения с уже опубликованными ветками (force push).

В процессе работы были успешно разрешены конфликты, что закрепило практические навыки работы с Git.
