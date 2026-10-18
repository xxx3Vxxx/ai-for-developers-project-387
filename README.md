# Календарь звонков

[![hexlet-check](https://github.com/xxx3Vxxx/ai-for-developers-project-387/actions/workflows/hexlet-check.yml/badge.svg)](https://github.com/xxx3Vxxx/ai-for-developers-project-387/actions)

Сервис для бронирования календаря, разработанный совместно с ИИ.

Учебный проект Хекслета: https://ru.hexlet.io/programs/ai-for-developers
Как это должно работать: https://files.hexlet.app/a/2ipc5m

## Демо

Опубликованное приложение: https://xxx3vxxx-hexlet-ai-for-developers-387.onrender.com

> Сервис развёрнут на бесплатном плане Render и засыпает после простоя — первый запрос может занять до минуты.
> Файловая система бесплатного плана эфемерна: база SQLite (брони) очищается при каждом редеплое.

## Стек

- TypeScript
- Бекенд: [Fastify](https://fastify.dev/)
- Фронтенд: [Vite](https://vite.dev/) + [React](https://react.dev/) + [Mantine](https://mantine.dev/)
- Тесты: [Vitest](https://vitest.dev/)
- Линтер и формат: ESLint + Prettier
- Пакетный менеджер: npm (workspaces)

## Установка

Требуется Node.js >= 22.

```bash
git clone https://github.com/xxx3Vxxx/ai-for-developers-project-387.git
cd ai-for-developers-project-387
npm install
```

## Использование

Запуск бекенда и фронтенда одновременно:

```bash
npm run dev
```

Отдельные сервисы:

```bash
npm run dev:backend  # только бекенд (tsx watch)
npm run dev:frontend # только фронтенд (Vite dev server)
```

- Бекенд: http://localhost:3000 (`GET /health` → `{"status":"ok"}`)
- Фронтенд: http://localhost:5173
- Запросы фронтенда на `/api/*` проксируются на бекенд (`http://localhost:3000`).

### Переменные окружения

Вся конфигурация читается из переменных окружения один раз при старте
(`backend/src/config.ts`, ADR-0013). Файл `.env` автоматически не подхватывается —
задайте переменные в окружении; полный список с дефолтами — в `.env.example`.
Некорректное значение переменной роняет старт с понятной ошибкой.

| Переменная            | Дефолт                   | Значение                                                   |
| --------------------- | ------------------------ | ---------------------------------------------------------- |
| `PORT`                | `3000`                   | целое 1–65535                                              |
| `HOST`                | `0.0.0.0`                | непустая строка                                            |
| `DATABASE_PATH`       | `./data/calendar.sqlite` | непустой путь к файлу SQLite; каталог создаётся при старте |
| `SERVICE_TIMEZONE`    | `Europe/Moscow`          | валидная IANA-зона                                         |
| `WORK_DAY_START`      | `09:00`                  | `HH:MM` (24 ч), начало раньше конца                        |
| `WORK_DAY_END`        | `18:00`                  | `HH:MM` (24 ч)                                             |
| `BOOKING_WINDOW_DAYS` | `14`                     | целое 1–365                                                |

### Контракты API

HTTP-контракт описан на [TypeSpec](https://typespec.io/) в `contracts/main.tsp`; из него
генерируются OpenAPI 3.0.0 и типы (`contracts/generated/openapi.d.ts`), которыми
пользуются бекенд и фронтенд. Артефакты не коммитятся: они создаются командой
`npm run contracts:generate` (она же выполняется автоматически перед `typecheck`,
`test` и `build`).

Проверки:

```bash
npm test          # тесты (Vitest)
npm run lint      # линтер (ESLint)
npm run typecheck # проверка типов (tsc)
npm run build     # сборка

npm run format       # форматирование (Prettier)
npm run format:check # проверка форматирования
```

Те же команды доступны через Makefile:

```bash
make dev    # бекенд + фронтенд
make contracts-generate # TypeSpec → OpenAPI → типы
make test   # тесты
make lint   # линтер
make check  # линтер + типы + тесты
make ci     # полный прогон, как в GitHub Actions
make docker-build # собрать прод-образ
make docker-run   # запустить прод-образ с томом под БД
make clean  # удалить node_modules и артефакты сборки
make help   # список всех команд
```

### Прод-образ (Docker)

Приложение поставляется одним образом из корневого `Dockerfile` (ADR-0014):
контейнер отдаёт и API, и статику фронта, а `GET /` возвращает приложение.
Контракты генерируются внутри сборки образа, `PORT`/`HOST` читаются из
окружения, а SQLite лежит в `/app/data/calendar.sqlite`.

```bash
make docker-build            # docker build -t calendar:local .
make docker-run              # docker run --rm -p 3000:3000 -v calendar-data:/app/data calendar:local
```

`docker-run` поднимает контейнер на `http://localhost:3000` с именованным томом
под базу, поэтому данные не теряются между перезапусками. Без тома поднимается
свежая БД (схема идемпотентна, сид — `INSERT OR IGNORE`).

---

<details>
<summary>Автоматические тесты Хекслета</summary>

Тесты запускаются на каждый коммит. За запуск отвечает файл `.github/workflows/hexlet-check.yml` — не удаляйте и не переименовывайте ни его, ни репозиторий.

</details>

## Воркфлоу ИИ-агента

В репозитории четыре воркфлоу с агентом OpenCode (экшен `anomalyco/opencode/github`).
Все работают на модели `opencode/qwen3.8-flash`; где смотреть прогоны — вкладка
[Actions](https://github.com/xxx3Vxxx/ai-for-developers-project-387/actions),
фильтр по имени воркфлоу.

| Воркфлоу              | Чем запускается                                                                                 | Назначение                                                   | Прогоны                                                                                                                         |
| --------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| `opencode.yml`        | комментарий к issue/PR с `/oc` или `/opencode` (`issue_comment`, `pull_request_review_comment`) | ассистент: объясняет issue, чинит баги, открывает PR         | [Actions → opencode](https://github.com/xxx3Vxxx/ai-for-developers-project-387/actions/workflows/opencode.yml)                  |
| `opencode-review.yml` | PR: `opened`, `synchronize`, `reopened`, `ready_for_review`                                     | автоматическое ревью pull request                            | [Actions → OpenCode PR Review](https://github.com/xxx3Vxxx/ai-for-developers-project-387/actions/workflows/opencode-review.yml) |
| `opencode-triage.yml` | issue: `opened`                                                                                 | триаж новых issue (ссылки на docs, guidance)                 | [Actions → Issue Triage](https://github.com/xxx3Vxxx/ai-for-developers-project-387/actions/workflows/opencode-triage.yml)       |
| `opencode-todo.yml`   | расписание: понедельник 09:00 UTC, плюс ручной `workflow_dispatch`                              | недельный скан TODO/FIXME, создаёт issue-отчёт `todo-report` | [Actions → OpenCode TODO Scan](https://github.com/xxx3Vxxx/ai-for-developers-project-387/actions/workflows/opencode-todo.yml)   |

### Триггеры и защита от ботов

- Ассистент (`opencode.yml`) запускается не на каждый комментарий, а только при
  упоминании команды; набор команд задан явно через `mentions: /opencode,/oc`
  (совпадает с дефолтом экшена, но зафиксирован в воркфлоу, чтобы поведение
  не менялось молча при обновлении экшена). Условие `if` в воркфлоу дублирует
  тот же набор — экшен фильтрует уже принятые запуски.
- Во всех воркфлоу есть фильтр событий от ботов и от самого приложения:
  `github.event.comment.user.type != 'Bot'` (комментарии),
  `github.event.pull_request.user.type != 'Bot'` (PR),
  `github.event.issue.user.type != 'Bot'` (issue). Тип `Bot` в GitHub API
  получают и сервисные аккаунты (`github-actions[bot]`, `release-please[bot]`),
  и GitHub-приложения, включая `opencode-agent[bot]`, — так агент не реагирует
  на собственные ответы и зацикливание исключено. В `opencode-todo.yml` фильтр
  не нужен: он стартует по расписанию и вручную, пользовательских событий
  не принимает.

### Кто может звать агента

Репозиторий публичный, поэтому круг вызывающих ассистента ограничен: комментарий
с `/oc` обрабатывается только если автор — `OWNER`, `COLLABORATOR` или `MEMBER`
(проверка `author_association` в условии `if`). Случайный прохожий не сможет
заставить агента выполнять действия в CI. Остальные воркфлоу комментариев от
публики не читают: review и triage открываются самими PR/issue, TODO-скан —
расписанием.

### Права (permissions) по воркфлоу

Права выданы отдельно каждому воркфлоу; запись через `GITHUB_TOKEN` разрешена
только там, где агент создаёт задачи:

| Воркфлоу              | permissions                                                                | Почему                                                                                                                           |
| --------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `opencode.yml`        | `id-token: write`, `contents: read`, `pull-requests: read`, `issues: read` | ответы, ветки и PR агент пишет от имени GitHub-приложения OpenCode (обмен по OIDC — `id-token`), токену воркфлоу запись не нужна |
| `opencode-review.yml` | `id-token: write`, `contents: read`, `pull-requests: read`, `issues: read` | ревью-комментарии пишет приложение; воркфлоу только читает                                                                       |
| `opencode-triage.yml` | `id-token: write`, `contents: read`                                        | комментарий к issue пишет приложение; запись `GITHUB_TOKEN` не требуется                                                         |
| `opencode-todo.yml`   | `contents: read`, `issues: write`                                          | работает в режиме `use_github_token` и сам создаёт issue-отчёт; `id-token` убран — OIDC-обмен в этом режиме не происходит        |

### Share

Репозиторий публичный, поэтому решение принято и записано здесь: публикацию
сессий OpenCode (`share`-ссылки на opencode.ai) мы **не включаем**. В GitHub-экшене
мы явно не просим шарить сессию, в конфигурации проекта `share` не задан; лог
прогона остаётся только во вкладке Actions и доступен тем, кому доступен сам
публичный репозиторий. Публичных ссылок на расшифровки диалогов агента не
создаём.

### Самооценка работы с агентом

- **С первого прохода:** review, triage и TODO-скан были написаны и заработали
  сразу (зелёные прогоны, осмысленные ревью в PR #9/#10); ассистент по `/oc`
  корректно читал контекст issue и вносил правки в ветку PR.
- **Потребовало итераций:** TODO-скан пришлось укреплять двумя правками
  (ограничить область поиска исходниками, не создавать issue при пустом
  отчёте, добавить `concurrency`-группу); в этом задании при перепроверке
  вскрылись недостающий фильтр ботов в `opencode.yml`, неиспользуемое право
  `id-token: write` в `opencode-todo.yml` и `issues: write` в triage — их
  убрали. Вывод: агент быстро собирает каркас CI, но права, триггеры и
  защиту от зацикливания нужно вычитывать вручную как код.

## О Хекслете

[Хекслет](https://ru.hexlet.io/) — школа программирования: авторские программы обучения с практикой, поддержкой наставников и реальными проектами, которые остаются в резюме. Этот репозиторий — один из таких проектов.
