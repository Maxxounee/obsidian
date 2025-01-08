
## Содержание:

* [Логирование](#Логирование)
* [Команды](#Команды)
* [Terminal](#Terminal)

## Полезные ссылки:

* [Ознакомительный курс git how to](https://githowto.com/)
* [Интерактивная обучалка learn git branching](https://learngitbranching.js.org/?locale=ru_RU)
* [Объяснение git rebase](https://selectel.ru/blog/tutorials/how-to-rebase-commits-and-branches/)


## #Логирование:

```bash
# Конфигурировение отображения лога
git config --global format.pretty '%h %ad | %s%d [%an]'
git config --global log.date short

# Логирование 
git log
git log --oneline
```

## #Команды:

```bash
# Переключение между ветками
git switch <branch-name>

# Rebase
git rebase 
```

### Rebase
`Наложение коммитов поверх другого базового коммита.`

```bash
# Наложение коммитов из HEAD на branch-to
git rebase <branch-to | hash>

# Наложение коммитов из branch-from на branch-to
git rebase <branch-to | hash> <branch-from>

# Интерактивный режим VIM 
git rebase <to> [-i | --interactive]


```
## Terminal
```bash
# Просмотр содержимого файла
cat filename.txt 
# Редактирование файла 
nano filename.txt
# Создание директории 
mkdir dirName
```
