Переменные окружения --- это пары КЛЮЧ=ЗНАЧЕНИЕ, которые хранят
настройку приложения вне кода: пароли, токены, URL базы, режимы
(dev/prod), флаги. Идея из «12-factor app»: конфиг должен жить вне
репозитория, чтобы код одинаково работал в разных средах (локально,
CI/CD, прод).

.env --- удобный локальный файл с такими парами для разработки. Он не
обязателен, но сильно упрощает жизнь.

⸻

# Базовые правила (жёсткие, но спасают)

1.  Никогда не коммить .env. Добавь в .gitignore:

```{=html}
<!-- -->
```
    .env
    .env.*
    !.env.example

2.  Делай шаблон: коммить .env.example без секретов (только ключи и
    безопасные значения/заглушки).\
3.  Разделяй среды: .env.development, .env.test, .env.production, а для
    локальных секретов --- .env.local (в гите отсутствует).\
4.  Секрет утёк → немедленно ротация: отозвать ключ, выдать новый,
    вычистить историю (git filter-repo/BFG), задокументировать в ретро.\
5.  Секреты не логировать и не класть в клиентский бандл.

⸻

# Как это работает в разных стеках

## 1) Node.js / Express (универсально)

Устанавливаем пакет:

``` bash
npm i dotenv
```

Файл .env:

    PORT=3000
    DATABASE_URL=postgres://user:pass@localhost:5432/app
    JWT_SECRET=dev_secret

Код:

``` js
// index.js
import 'dotenv/config'; // или: require('dotenv').config()
import express from 'express';

const app = express();
const port = process.env.PORT ?? 3000;

app.get('/health', (_, res) => res.json({ ok: true }));

app.listen(port, () => {
  console.log(`Server on :${port}`);
});
```

## 2) Python / Flask

``` bash
pip install python-dotenv
```

.env:

    FLASK_ENV=development
    DATABASE_URL=postgresql+psycopg://user:pass@localhost:5432/app
    SECRET_KEY=dev_secret

Код:

``` python
from flask import Flask
from dotenv import load_dotenv
import os

load_dotenv()  # подтягивает .env в os.environ

app = Flask(__name__)
app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]
```

## 3) Next.js / React-мира нюансы

-   Next.js автоматически читает .env.local, .env.development,
    .env.production.\
-   В браузер протекают только переменные с префиксом `NEXT_PUBLIC_`.
    Остальные доступны на сервере.\
-   Пример:

```{=html}
<!-- -->
```
    # только на сервере
    DATABASE_URL=...
    # попадёт в клиентский бандл
    NEXT_PUBLIC_API_BASE=https://api.example.com

В Vite: только `VITE_*` доступны в клиенте →
`import.meta.env.VITE_API_URL`.

⸻

# Как дружить с Git

.gitignore (минимум):

    .env
    .env.*
    !.env.example

.env.example (коммитим):

    PORT=3000
    DATABASE_URL=postgres://USER:PASSWORD@HOST:5432/DBNAME
    JWT_SECRET=__SET_ME__
    NEXT_PUBLIC_API_BASE=http://localhost:3000

Если .env уже попал в историю: 1. Отозвать все ключи.\
2. Удалить файл из индекса и сделать коммит:

``` bash
git rm --cached .env
git commit -m "chore(security): stop tracking .env"
```

3.  Переписать историю:

-   `git filter-repo --path .env --invert-paths`
-   или BFG Repo-Cleaner.\

4.  Всем разработчикам:

``` bash
git fetch --all
git reset --hard origin/main
```

⸻

# Прод и CI/CD: где хранить секреты

В проде .env часто не нужен: платформа отдаёт переменные сама.

-   **Vercel/Netlify/Render/Fly.io**: секреты задаются в панели
    проекта.\
-   **Docker / Compose**:

``` yaml
services:
  api:
    image: app:latest
    env_file: .env  # для локалки
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
```

-   **GitHub Actions**: Settings → Secrets → Actions.\
    Workflow:

``` yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
        run: |
          npm ci
          npm run build
```

⸻

# Как задавать переменные в ОС (macOS)

-   Один запуск:

``` bash
PORT=4000 node index.js
```

-   В текущую сессию:

``` bash
export PORT=4000
```

-   Постоянно (`~/.zshrc`):

``` bash
export PORT=4000
export EDITOR=code
```

Затем `source ~/.zshrc`.

⸻

# Типичные ошибки и как их избегать

-   Секреты в фронтенде: всё, что уходит в браузер, нельзя считать
    секретом.\
-   Жёстко вбитые креды в коде: никогда `const PASSWORD = "123"`. Всегда
    `process.env.PASSWORD`.\
-   Смешивание сред: разные `.env.*` для dev/test/prod.\
-   Логи: не печатай `process.env` целиком.\
-   Кэш: переменные на этапе сборки «запекаются» в бандл. Меняешь →
    пересобери.

⸻

# Мини-практикум

1.  Создай `.env.example`.\
2.  Добавь `.env` в `.gitignore`.\
3.  Подключи загрузку env в коде.\
4.  Проверь локально и в CI/хостинге.\
5.  Добавь в README раздел «Config».

⸻

# Чек-лист перед коммитом

-   `.env` в `.gitignore`?\
-   Есть `.env.example`?\
-   Секреты не светятся в клиентском бандле?\
-   README объясняет настройку?\
-   В CI secrets заведены?
