# Конспект по работе с Git

## Основные концепции

### **Git** - система контроля версий

-   Отслеживает изменения в файлах
-   Позволяет работать с историей изменений
-   Поддерживает совместную работу

### **Репозиторий** - папка с проектом под контролем Git

-   **Локальный** - на вашем компьютере
-   **Удаленный** - на GitHub/GitLab/etc.

## Основной рабочий процесс

### 1. **Инициализация проекта**

``` bash
# Создать Git репозиторий в папке
git init

# Создать .gitignore файл
touch .gitignore
```

### 2. **Подготовка файлов**

``` bash
# Добавить все файлы в staging area
git add .

# Добавить конкретный файл
git add filename.py

# Добавить несколько файлов
git add file1.py file2.py

# Добавить все файлы определенного типа
git add *.py
```

### 3. **Создание коммита**

``` bash
# Создать коммит с сообщением
git commit -m "Описание изменений"

# Примеры сообщений:
git commit -m "Add user authentication"
git commit -m "Fix bug in login function"
git commit -m "Update README documentation"
```

### 4. **Подключение к удаленному репозиторию**

``` bash
# Добавить удаленный репозиторий
git remote add origin https://github.com/username/repository.git

# Проверить подключенные репозитории
git remote -v

# Переименовать основную ветку в main
git branch -M main
```

### 5. **Отправка на GitHub**

``` bash
# Первый пуш (установка upstream)
git push -u origin main

# Последующие пуши
git push
```

## Полезные команды

### **Проверка статуса**

``` bash
# Посмотреть статус файлов
git status

# Посмотреть изменения в файлах
git diff

# Посмотреть изменения в staged файлах
git diff --staged
```

### **История коммитов**

``` bash
# Полная история
git log

# Краткая история (одна строка на коммит)
git log --oneline

# История с графиком веток
git log --graph --oneline
```

### **Работа с ветками**

``` bash
# Посмотреть все ветки
git branch

# Посмотреть ветки с дополнительной информацией
git branch -vv

# Создать новую ветку
git branch feature-name

# Переключиться на ветку
git checkout branch-name

# Создать и переключиться на новую ветку
git checkout -b feature-name
```

### **Получение изменений с GitHub**

``` bash
# Получить изменения (без слияния)
git fetch origin

# Получить и слить изменения
git pull origin main

# Или просто (если upstream настроен)
git pull
```

## Типичный рабочий процесс

### **Ежедневная работа:**

``` bash
# 1. Начать работу
git pull                    # Получить последние изменения

# 2. Внести изменения в код
# ... редактируете файлы ...

# 3. Подготовить к коммиту
git add .                   # Добавить все изменения
git status                  # Проверить что добавлено

# 4. Создать коммит
git commit -m "Add new feature"

# 5. Отправить на GitHub
git push
```

### **Создание нового проекта:**

``` bash
# 1. Создать папку проекта
mkdir my-project
cd my-project

# 2. Инициализировать Git
git init

# 3. Создать .gitignore
touch .gitignore
# Добавить в .gitignore нужные исключения

# 4. Добавить файлы
git add .

# 5. Первый коммит
git commit -m "Initial commit"

# 6. Создать репозиторий на GitHub
# (через веб-интерфейс)

# 7. Подключить к GitHub
git remote add origin https://github.com/username/repository.git
git branch -M main

# 8. Отправить на GitHub
git push -u origin main
```

## Полезные файлы

### **.gitignore** - исключения из Git

    # Python
    __pycache__/
    *.py[cod]
    venv/
    env/

    # IDE
    .vscode/
    .idea/

    # OS
    .DS_Store
    Thumbs.db

    # Logs
    *.log

## Проверка и диагностика

### **Проверить настройки:**

``` bash
# Настройки пользователя
git config --list

# Установить пользователя (если нужно)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### **Проверить состояние:**

``` bash
# Статус репозитория
git status

# Подключенные репозитории
git remote -v

# Текущая ветка
git branch

# История коммитов
git log --oneline
```

## Важные моменты

### **Коммиты:**

-   `git commit` создает коммит **локально**
-   Коммиты **НЕ отправляются** автоматически на GitHub
-   Для отправки нужен `git push`

### **Push:**

-   `git push` отправляет **все коммиты** на GitHub
-   Первый раз нужен `-u` для установки upstream
-   В дальнейшем достаточно просто `git push`

### **Pull:**

-   `git pull` получает изменения с GitHub
-   Рекомендуется делать перед началом работы

### **Безопасность:**

-   Всегда проверяйте `git status` перед коммитом
-   Используйте понятные сообщения коммитов
-   Регулярно делайте `git push`
