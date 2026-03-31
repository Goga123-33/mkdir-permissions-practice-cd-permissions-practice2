# КОМАНДЫ ДЛЯ ВЫПОЛНЕНИЯ В GITHUB CODESPACES

## Подготовка к работе
```bash
cd ~/permissions-practice
mkdir -p myapp/scripts myapp/config myapp/src
```

## Задание 1: Читаем права доступа (ls -l)
```bash
cd ~/permissions-practice
touch script.sh config.conf secret.txt readme.md
mkdir project_folder
ls -l
```

**Результаты для таблицы:**
- script.sh: -rw-r--r-- (644)
- config.conf: -rw-r--r-- (644)
- secret.txt: -rw-r--r-- (644)
- readme.md: -rw-r--r-- (644)
- project_folder/: drwxr-xr-x (755)

## Задание 2: Изменяем права (chmod)

### Шаг 1: Сделать скрипт исполняемым
```bash
chmod +x script.sh
ls -l script.sh
```
Результат: -rwxr-xr-x (755)

### Шаг 2: Убрать доступ к секретному файлу
```bash
chmod 600 secret.txt
ls -l secret.txt
```
Результат: -rw------- (600)

### Шаг 3: Сделать файл только для чтения
```bash
chmod 444 readme.md
ls -l readme.md
```
Результат: -r--r--r-- (444)

### Шаг 4: Попробовать записать (будет ошибка - это нормально!)
```bash
echo "test" >> readme.md
```
Ожидаемая ошибка: Permission denied

### Шаг 5: Закрыть папку от посторонних
```bash
chmod 700 project_folder
ls -l
```
Результат: drwx------ (700)

### Шаг 6: Проверить все права
```bash
ls -la
```

## Задание 3: Опасные права

### Шаг 1: Создать файл с опасными правами
```bash
echo "db_password = supersecret123" > bad_config.conf
echo "api_key = abc456xyz" >> bad_config.conf
chmod 777 bad_config.conf
ls -l bad_config.conf
```
Результат: -rwxrwxrwx (777) - ОПАСНО!

### Шаг 2: Найти все файлы с правами 777
```bash
find . -perm 777
```

### Шаг 3: Найти конфиги, доступные для чтения всем
```bash
find . -name "*.conf" -perm /o+r
```

### Шаг 4: Исправить опасные права
```bash
chmod 600 bad_config.conf
ls -l bad_config.conf
```
Результат: -rw------- (600)

### Шаг 5: Убедиться что опасных файлов нет
```bash
find . -perm 777
```
Должно быть пусто!

## Задание 4: Владелец файла (chown)

### Шаг 1: Посмотреть владельцев
```bash
ls -l
whoami
id
```

### Шаг 2: Узнать свою группу
```bash
groups
```

### Шаг 3: Создать новую группу
```bash
sudo groupadd dev-team
getent group dev-team
```

### Шаг 4: Добавить себя в группу
```bash
sudo usermod -aG dev-team $(whoami)
newgrp dev-team
groups
```

### Шаг 5: Сменить группу-владельца файла
```bash
sudo chown :dev-team script.sh
ls -l script.sh
```

### Шаг 6: Сменить группу для всей папки
```bash
sudo chown -R :dev-team project_folder/
ls -la project_folder/
```

## Задание 5: Финальная проверка проекта

### Шаг 1: Посмотреть структуру
```bash
ls -lR myapp/
```

### Шаг 2: Настроить права на файлы
```bash
# Скрипт деплоя должен запускаться
chmod 755 myapp/scripts/deploy.sh

# Конфиг с данными БД - тол��ко владелец
chmod 600 myapp/config/database.conf

# Исходный код - читать всем, писать только владельцу
chmod 644 myapp/src/main.py

# README - читать всем
chmod 644 myapp/README.md
```

### Шаг 3: Финальная проверка
```bash
ls -lR myapp/
find myapp/ -perm 777
```

### Шаг 4: Сохранить в Git
```bash
cd ~/permissions-practice
git init
git add .
git commit -m "setup: настроены права доступа для всех файлов проекта"
git remote add origin https://github.com/Goga123-33/mkdir-permissions-practice-cd-permissions-practice2.git
git push -u origin main
```

## ОЖИДАЕМЫЕ РЕЗУЛЬТАТЫ

Таблица 1 - После Задания 1:
| Файл | Символьно | Числово |
|------|-----------|---------|
| script.sh | -rw-r--r-- | 644 |
| config.conf | -rw-r--r-- | 644 |
| secret.txt | -rw-r--r-- | 644 |
| readme.md | -rw-r--r-- | 644 |
| project_folder/ | drwxr-xr-x | 755 |

Таблица 2 - После Задания 2:
| Файл | Права | Команда chmod |
|------|-------|---------------|
| script.sh | -rwxr-xr-x | chmod +x script.sh |
| secret.txt | -rw------- | chmod 600 secret.txt |
| readme.md | -r--r--r-- | chmod 444 readme.md |
| project_folder/ | drwx------ | chmod 700 project_folder |

Таблица 3 - Итоговый проект:
| Файл | Права | Обоснование |
|------|-------|-------------|
| scripts/deploy.sh | -rwxr-xr-x (755) | Скрипт должен быть исполняемым для владельца и группы |
| config/database.conf | -rw------- (600) | Конфиг с паролями - доступ только владельцу |
| src/main.py | -rw-r--r-- (644) | Исходный код - читать всем, писать только владельцу |
| README.md | -rw-r--r-- (644) | Документация - читать всем, писать только владельцу |

## ОТВЕТ НА ВОПРОС

**Почему для файла database.conf нужны права 600, а не 644?**

Потому что database.conf содержит чувствительные данные (пароли базы данных, учётные данные). Права 644 позволили бы любому пользователю системы прочитать эти пароли, что создаёт серьёзную уязвимость безопасности. Права 600 (rw-------) ограничивают доступ только владельцу файла, защищая конфиденциальные данные от несанкционированного доступа другими пользователями.