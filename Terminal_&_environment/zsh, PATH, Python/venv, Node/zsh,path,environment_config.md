Ниже — обновлённый конспект “как взрослый программист управляет терминалом/окружением”, с твоими текущими настройками внутри.

⸻

КОНСПЕКТ: Терминал, zsh, PATH, Python/venv, Node/nvm, агенты

0) Главная идея

Терминал — это не “магия”, а окружение: набор переменных и правил, которые определяют, какие команды запускаются и откуда.
Ключевая переменная — PATH.

⸻

1) Два типа оболочки: login shell и interactive shell

Login shell

Запуск “как вход в сессию” (часто так стартует macOS Terminal).
Используется для базовых переменных окружения и PATH, которые должны быть доступны широко.

Файл: ~/.zprofile

Interactive shell

Интерактивная работа: prompt, автодополнение, алиасы, плагины, удобства.

Файл: ~/.zshrc

Проверка типа текущей сессии:

echo $0
[[ -o login ]] && echo "login shell" || echo "not login"
[[ -o interactive ]] && echo "interactive shell" || echo "not interactive"


⸻

2) Что такое PATH и почему порядок решает всё

PATH — список папок. Когда ты вводишь команду (python3, node, git), zsh ищет её слева направо по PATH и запускает первое совпадение.

Показать PATH красиво:

echo $PATH | tr ':' '\n' | nl

Узнать, откуда реально берётся команда:

type -a python3
type -a pip3
type -a node
type -a brew


⸻

3) Откуда вообще берётся PATH (3 источника)
	1.	Система macOS
Базовые пути (/usr/bin, /bin, Cryptexes и т.д.)
	2.	Твои файлы zsh
~/.zprofile и ~/.zshrc
	3.	Подключённые инструменты/скрипты
Например nvm.sh сам меняет PATH, чтобы активная версия Node была первой.

⸻

4) Твоя текущая конфигурация (что мы настроили)

~/.zprofile (login, база)

eval "$(/opt/homebrew/bin/brew shellenv)"
export PATH="/opt/homebrew/opt/python@3.13/libexec/bin:$PATH"

Что это делает:
	•	brew shellenv добавляет Homebrew-пути (/opt/homebrew/bin, /opt/homebrew/sbin).
	•	libexec/bin делает так, что python3 и pip3 берутся из Homebrew python@3.13, а не из /usr/local/bin.

~/.zshrc (interactive, инструменты)

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

export PATH="$PATH:/Users/tamerlan/.antigravity/antigravity/bin"

typeset -U path
path=($path)

Что это делает:
	•	nvm подключается и ставит Node из nvm в начало PATH (это нормально).
	•	Antigravity есть в PATH, но добавлен в конец, чтобы не подменять базовые команды.
	•	дедупликация убирает повторяющиеся пути в PATH.

⸻

5) Почему раньше была “каша” и как мы её убрали

Причины каши:
	•	PATH настраивался несколькими местами и мог дублироваться.
	•	Antigravity раньше стоял в начале PATH, мог перехватывать команды.
	•	несколько Python-источников одновременно (/usr/local, системный, brew).
	•	повторный source мог временно раздувать PATH в текущем окне.

Решение:
	•	Базу (brew + python) держим в .zprofile.
	•	Инструменты (nvm, antigravity) — в .zshrc.
	•	Antigravity — в конец.
	•	Дедупликация в .zshrc.

⸻

6) Что такое source и зачем он нужен

source FILE = выполнить настройки из файла прямо сейчас в текущей сессии.

Пример:

source ~/.zshrc

Зачем:
	•	ты поменял файл и хочешь применить без открытия нового окна.

Нюанс:
	•	если в файле есть export PATH="X:$PATH", повторный source может временно сделать дубли в PATH в этом окне (у тебя это компенсируется дедупликацией в .zshrc).

⸻

7) Python и venv: как работать “чисто” и без засора системы

Главное правило

В проектах (особенно в venv) ставь пакеты так:

python -m pip install <package>

Вне venv:

python3 -m pip install <package>

Почему это железобетон:
	•	pip точно относится к тому python, который ты запускаешь.
	•	меньше риска “поставил не туда”.

⸻

8) Node / pnpm / nvm (как это у тебя устроено)
	•	Node версии управляются через nvm, поэтому .../.nvm/.../bin стоит первым.
	•	Проекты на Next/Vite ты ведёшь через pnpm (обычно поверх node/nvm).
Это правильная современная схема.

⸻

9) Antigravity: как держать “эксперимент” под контролем

Принцип:
	•	Antigravity должен быть доступен, но не должен подменять фундамент.

Поэтому:

export PATH="$PATH:/Users/tamerlan/.antigravity/antigravity/bin"

(в конец PATH)

⸻

10) CLI и агенты (Cursor Agent)

CLI

Command Line Interface — запуск из терминала командами.

У тебя Cursor Agent установлен как:
	•	~/.local/bin/agent
	•	~/.local/bin/cursor-agent

Сейчас ~/.local/bin не в PATH, поэтому:
	•	запуск по имени может не работать,
	•	но запуск по полному пути работает:

~/.local/bin/agent

Нужно ли добавлять ~/.local/bin в PATH?
	•	только если хочешь запускать команду без полного пути.
	•	добавлять лучше в конец:

export PATH="$PATH:$HOME/.local/bin"


⸻

11) Как быстро проверять, что всё “идеально” (диагностика)

Проверить откуда берутся команды:

type -a brew node python3 pip3

Показать порядок PATH:

echo $PATH | tr ':' '\n' | nl | head -n 20

Проверить дубли:

echo $PATH | tr ':' '\n' | sort | uniq -d


⸻

12) Что запомнить как программист (минимум, который даёт максимум)
	1.	PATH — порядок = приоритет.
	2.	.zprofile — база (login), .zshrc — интерактив (удобства).
	3.	type -a — лучший “детектор правды”.
	4.	Python-пакеты: python -m pip (в venv), python3 -m pip (вне venv).
	5.	Экспериментальные инструменты не держать “в начале PATH”.

⸻

Да, давай добавим в конспект блок про бэкапы/резервные копии (это прям must-have, потому что это твоя “кнопка отката”).

13) Резервные копии (backup) и быстрый откат

Что у тебя уже есть

Ты сделал “золотые” копии:
	•	~/.zprofile.GOLD
	•	~/.zshrc.GOLD

И у тебя есть автоматические бэкапы вида:
	•	~/.zshrc.bak.*
	•	~/.zprofile.bak.*

Как проверить, что бэкапы существуют

ls -la ~/.zprofile.GOLD ~/.zshrc.GOLD 2>/dev/null
ls -la ~/.zprofile.bak.* ~/.zshrc.bak.* 2>/dev/null

Как посмотреть содержимое (без редакторов)

nl -ba ~/.zprofile | head -n 80
nl -ba ~/.zshrc | head -n 120

Как сравнить текущее с GOLD (увидеть изменения)

diff -u ~/.zprofile.GOLD ~/.zprofile | less
diff -u ~/.zshrc.GOLD ~/.zshrc | less

Выход из less: q.

Как откатиться к GOLD (полный откат)

cp ~/.zprofile.GOLD ~/.zprofile
cp ~/.zshrc.GOLD ~/.zshrc
exec zsh -l

exec zsh -l перезапустит оболочку как login shell и подтянет всё заново.

Как откатиться к конкретному .bak (если надо)

Сначала найди нужный:

ls -t ~/.zshrc.bak.* | head
ls -t ~/.zprofile.bak.* | head

Потом восстанови:

cp ~/.zshrc.bak.YYYYMMDD-HHMMSS ~/.zshrc
cp ~/.zprofile.bak.YYYYMMDD-HHMMSS ~/.zprofile
exec zsh -l


⸻

