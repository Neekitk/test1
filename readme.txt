Test1
## Настройка Git
### Добавление имени
git config --global user.name "Nekit"
### Добавление почты
git config --global user.email "nikstriker64@gmail.com"
### Проверяем правильность выполнения команд
git config --list

## Добавление SSH-ключей
### Создаём новый SSH-ключ
ssh-keygen -t ed25519 -C "nikstriker64@gmail.com" 
### Добавляем SSH-ключ в ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
### Показать SSH-ключ
cat ~/.ssh/id_ed25519.pub

# 1. Подготовка репозитория
### Создаем и переходим в каталог
mkdir lab1_git
cd lab1_git
### Инициализируем новый локальный репозиторий
git init
### Создаем файл с описанием лабораторной
echo "# Лабораторная работа №1" > README.md
echo "В данной лабораторной работе показан процесс работы с Git через командную строку" >> README.md
### Исключаем временные и сборочные файлы C#
echo "bin/" > .gitignore
echo "obj/" >> .gitignore
echo "*.user" >> .gitignore
echo "*.suo" >> .gitignore
echo "*.log" >> .gitignore
echo "*.tmp" >> .gitignore
### Добавляем все файлы в индекс
git add .
### Делаем первый коммит с сообщением
git commit -m "Инициализация лабораторной работы: добавлены README и .gitignore"
### # Подключаем удалённый репозиторий
git remote add origin git@github.com:Neekitk/Lab1_git.git 
### Переименовываем ветку master в main
git branch -M main
### Отправляем первый коммит в удалённый репозиторий
git push -u origin main

# 2. Git Flow — работа с ветками
### Создаем новую ветку develop, переключаемся на неё и отправляем
git checkout -b develop
git push -u origin develop
### Создаем каталог и файл с программой
mkdir src 
echo 'using System;

namespace Lab1
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Лабораторная работа №1");
        }
    }
}' > src/Program.cs
### Добавляем, коммитим и отправляем файл в удалённый репозиторий
git add src/Program.cs
git commit -m "Создан основной файл программы"
git push -u origin develop
git checkout -b feature/validation
### Создаем файл для проверки валидации данных
echo 'using System;

namespace Lab1
{
    public static class Validator
    {
        public static bool ValidateInput(string input)
        {
            return int.TryParse(input, out _);
        }
    }
}' > src/Validator.cs
git add src/Validator.cs
git commit -m "Добавлен класс для проверки типа данных"
git push -u origin feature/validation
### Переходим в ветку develop и делаем слияние без fast-forward
git checkout develop
git merge --no-ff feature/validation -m "Слияние: добавлен класс валидации"
git push -u origin develop
git checkout -b feature/extra
### Создаем файл с доп. функционалом
echo 'using System;

namespace Lab1
{
    public static class ExtraFeature
    {
        public static void ShowMessage()
        {
            Console.WriteLine("Дополнительный функционал добавлен!");
        }
    }
}' > src/ExtraFeature.cs
git add src/ExtraFeature.cs
git commit -m "Добавлен класс с дополнительным функционалом"
git push -u origin feature/extra
### Добавляем еще информацию в файл README
echo "## Дополнительная информация о лабораторной работы" >> README.md
git add README.md
git commit -m "Обновлён README с дополнительной информацией"
git push -u origin feature/extra

# 3. Работа с ветками (rebase, cherry-pick)
git checkout feature/validation
### Выполняем перебазирование (перепривязка коммитов к новой ветке) и принудительно отправляем в удалённый репозиторий
git rebase develop
git push --force origin feature/validation
git checkout develop
### Переносим выбранный коммит из другой ветки
git cherry-pick cd7b4e8
git push -u origin develop

# 4. Откаты изменений
git checkout develop
### Создаем новый коммит, отменяющий изменения
git revert 08874f3
git push -u origin develop
### Откатываем последний коммит, но изменения остаются, коммитим и принудительно отправляем
git reset --soft HEAD~1
git commit -m "Добавление после soft reset"
git push --force origin develop
### Полностью откатываем состояние
git reset --hard HEAD~1
### Восстанавливаем файл из коммита, добавляем, коммитим и принудительно отправляем
git restore src/Program.cs
git add src/Program.cs
git commit -m "Восстановлен файл Program.cs после restore"
git push --force origin develop

# 5. Git Hooks
## Проверка на TODO - pre-commit и автоматическое выполнение перед созданием коммита
echo '#!/bin/bash
if grep -r "TODO" src/; then
  echo "В коде найдены TODO"
  exit 1
fi' > .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
## Проверка на пустые сообщения - commit-msg и автоматическое выполнение перед созданием коммита
echo '#!/bin/bash
if [ -z "$(cat $1)" ]; then
  echo "Пустое сообщение коммита"
  exit 1
fi' > .git/hooks/commit-msg
chmod +x .git/hooks/commit-msg
## Проверка на ошибку компиляции - pre-push и автоматическое выполнение перед созданием коммита
echo '#!/bin/bash
echo "Проверка перед отправкой"
dotnet build src/ >/dev/null 2>&1 || { 
    echo "Ошибка компиляции"; 
    exit 1; 
}' > .git/hooks/pre-push
chmod +x .git/hooks/pre-push
### Добавляем файл с описанием хуков и отправляем его
echo "Созданы хуки: pre-commit, commit-msg, pre-push" > hooks_info.txt
git add hooks_info.txt
git commit -m "Добавлено описание созданных Git-хуков"
git push -u origin develop

# 6. Решение конфликтов
git checkout -b feature/conflict
echo 'Console.WriteLine("Конфликтная ветки");' >> src/Program.cs
git add src/Program.cs
git commit -m "Добавлено сообщение из конфликтной ветки"
git push -u origin feature/conflict
git checkout develop
echo 'Console.WriteLine("Ветка разработки");' >> src/Program.cs
git add src/Program.cs
git commit -m "Добавлено сообщение из ветки разработки"
git push -u origin develop
## Вызов merge с конфликтом
git merge feature/conflict
## Разрешаем конфликт вручную
git add src/Program.cs 
git commit -m "Исправлен конфликт при слиянии"
git push -u origin develop

# 7. Работа с тегами
## Тег для основной версии
git tag -a v1.0 -m "Релиз версии v1.0"
git push origin v1.0
## Добавление нового функционала
git checkout -b feature/additional
echo 'Console.WriteLine("Добавлен новый функционал v1.1-beta");' >> src/Program.cs
git add src/Program.cs
git commit -m "Добавлен новый функционал для бета-версии"
git checkout develop
git merge --no-ff feature/additional -m "Слияние: добавлен новый функционал v1.1-beta"
git push -u origin develop
## Тег бета-версии с функционалом
git tag -a v1.1-beta -m "Бета-версия v1.1"
git push origin v1.1-beta
