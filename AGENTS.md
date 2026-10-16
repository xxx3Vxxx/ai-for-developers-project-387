# AGENTS.md

Рабочие правила проекта «Календарь звонков» (учебный проект Хекслета,
<https://ru.hexlet.io/programs/ai-for-developers>).

## Что это

Рабочее приложение сервиса бронирования календаря, а не каркас.

- **Бекенд:** Fastify + SQLite. Отдаёт API ресурсов `config`, `event-types`, `slots`
  и `bookings` (плюс `GET /health`), хранит типы событий и брони в одной базе с
  идемпотентной схемой и сидом (ADR-0006), в проде отдаёт и статику фронта (ADR-0014).
- **Фронтенд:** Vite + React + Mantine. Три маршрута: лендинг `/`, визард бронирования
  гостя `/book` (ADR-0011) и экран владельца `/admin` (ADR-0012). Владелец и гость
  работают без авторизации (ADR-0001).
- **Контракт:** TypeSpec → OpenAPI 3.0.0 → типы; им следуют обе стороны (ADR-0008).
- Есть тесты (Vitest: контракты, бекенд, фронтенд с a11y-проверками), линтеры,
  CI, релизы и прод-образ в одном контейнере.

Язык предметной области — `GLOSSARY.md`, принятые решения — `docs/adr/` (0001–0016).
Прежде чем менять поведение, сверяйтесь с ADR и не противоречьте им молча.

## Стек

- **Язык:** TypeScript
- **Бекенд:** Fastify, `@fastify/static`, `better-sqlite3`
- **Фронтенд:** Vite + React + React Router + [Mantine](https://mantine.dev/),
  клиент на `openapi-fetch`
- **Контракты:** TypeSpec → OpenAPI 3.0.0 → типы (`openapi-typescript`) (ADR-0008)
- **Тесты:** Vitest (+ Testing Library и `jest-axe` на фронтенде)
- **Линтер/формат:** ESLint (flat config) + Prettier, проверка типов через `tsc`
- **Пакетный менеджер:** npm (workspaces: `contracts`, `backend`, `frontend`)

## Структура

```
.
├── GLOSSARY.md                # язык предметной области (один контекст на репозиторий)
├── docs/
│   ├── adr/                   # ADR 0001–0016 — принятые архитектурные решения
│   └── agents/                # правила для агентов: issue-tracker, triage-labels, domain
├── .env.example               # полный список переменных окружения с дефолтами (ADR-0013)
├── contracts/                 # TypeSpec-контракт HTTP API (ADR-0008)
│   ├── main.tsp               # источник истины контракта
│   ├── tspconfig.yaml         # эмиттер OpenAPI, версия пинится как 3.0.0
│   └── test/                  # проверка пайплайна генерации
│   # tsp-output/ и generated/ — артефакты, не коммитятся
├── backend/                   # Fastify API + статика фронта
│   ├── src/
│   │   ├── server.ts          # точка входа: loadConfig → buildApp → listen
│   │   ├── app.ts             # buildApp() — сборка приложения (используется и в тестах)
│   │   ├── config.ts          # env-конфиг, один раз, fail-fast (ADR-0013)
│   │   ├── db.ts              # схема SQLite + сид (ADR-0006)
│   │   ├── routes/            # HTTP-слой: config, event-types, slots, bookings
│   │   ├── event-types.ts, slots.ts, bookings.ts, time.ts   # доменные модули
│   │   ├── validation.ts, errors.ts, messages.ts            # валидация и ошибки (ADR-0005/0009/0010), тексты (ADR-0015)
│   │   └── static.ts          # раздача фронта и SPA-fallback (ADR-0014)
│   └── test/                  # 14 файлов: юниты + интеграция роутов через buildApp
├── frontend/                  # Vite + React + Mantine
│   ├── src/
│   │   ├── main.tsx, App.tsx  # маршруты /, /book, /admin
│   │   ├── api/               # openapi-fetch-клиент и запросы по ресурсам
│   │   ├── pages/             # LandingPage, BookingPage, AdminPage
│   │   ├── components/        # Layout, Header, SkipLink; book/*, admin/*
│   │   ├── copy/              # тексты UI (ADR-0015)
│   │   └── format/, validation.ts, vite-env.d.ts
│   └── test/                  # тесты через Testing Library, включая a11y
├── Dockerfile                 # прод-образ: API + статика фронта (ADR-0014)
├── .dockerignore              # контекст сборки образа
├── .github/workflows/
│   ├── ci.yml                 # npm ci → lint → typecheck → test → build
│   ├── release-please.yml     # релизы по Conventional Commits
│   ├── opencode.yml           # ассистент по комментариям вида `/oc ...`
│   └── hexlet-check.yml       # автопроверка Хекслета — НЕ ТРОГАТЬ
├── eslint.config.mjs, .prettierrc, tsconfig.base.json
└── Makefile                   # удобные команды (make help)
```

## Требования

- Node.js **>= 22**
- npm (идёт вместе с Node)

## Переменные окружения

Конфигурация читается один раз при старте из переменных окружения (`backend/src/config.ts`,
ADR-0013). Файл `.env` автоматически не подхватывается; полный список с дефолтами —
в `.env.example`. Некорректное значение роняет старт с внятной ошибкой, отката к
дефолту нет.

| Переменная            | Дефолт                   | Значение                                            |
| --------------------- | ------------------------ | --------------------------------------------------- |
| `PORT`                | `3000`                   | порт HTTP, целое 1–65535                            |
| `HOST`                | `0.0.0.0`                | хост HTTP                                           |
| `DATABASE_PATH`       | `./data/calendar.sqlite` | путь к файлу SQLite, каталог создаётся при старте   |
| `SERVICE_TIMEZONE`    | `Europe/Moscow`          | IANA-зона сервиса (ADR-0004)                        |
| `WORK_DAY_START`      | `09:00`                  | начало рабочего окна, `HH:MM` (24 ч), раньше конца  |
| `WORK_DAY_END`        | `18:00`                  | конец рабочего окна, `HH:MM` (24 ч)                 |
| `BOOKING_WINDOW_DAYS` | `14`                     | длина окна записи в календарных сутках, целое 1–365 |

## Команды

Все команды выполняются из корня репозитория. Скрипты — в `package.json`, цели —
`make help`. Ключевое:

- `npm install` — установка зависимостей.
- `npm run contracts:generate` — сгенерировать OpenAPI 3.0.0 и типы из
  `contracts/main.tsp`; выполняется автоматически перед `typecheck`, `test` и
  `build`. Артефакты (`contracts/tsp-output/`, `contracts/generated/`) не
  коммитятся.
- `make check` — линтер + типы + тесты (прогнать перед коммитом); `make ci` —
  то же плюс сборка, как в GitHub Actions.
- `make dev` (или `npm run dev`) — бекенд и фронтенд вместе; `make dev-backend` /
  `make dev-frontend` — по отдельности.
- `make build` и `make start` — собрать и запустить прод-вариант бекенда;
  `make test-watch` — Vitest в режиме наблюдения (backend).
- `npm run format` / `make format` — Prettier; `make format-check` — проверка
  форматирования.
- `make docker-build` / `make docker-run` — собрать и запустить прод-образ
  (API + статика фронта в одном контейнере, именованный том под SQLite, ADR-0014).
- backend :3000 (`GET /health` → `{"status":"ok"}`, API — под `/api/*`),
  frontend :5173; запросы фронтенда на `/api/*` проксируются на backend. В проде
  тот же порт отдаёт SPA: `GET` вне `/api` с `Accept: text/html` получает `index.html`.

## Правила коммитов

[Conventional Commits](https://www.conventionalcommits.org/):
`<type>(<scope>): <description>`. Типы: `feat`, `fix`, `docs`, `style`,
`refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`. `scope`
необязателен (например `backend`, `frontend`, `ci`), `description` — в
повелительном наклонении, без точки в конце.

Каждый коммит в `main` обязан следовать формату — от этого зависит
release-please (CHANGELOG и версия).

Пример: `feat(backend): add booking endpoint`

## CI и ограничения

CI (`.github/workflows/ci.yml`) запускается на каждый push и pull request:
`npm ci` → `npm run lint` → `npm run typecheck` → `npm test` → `npm run build`.
Держите прогон зелёным.

Не удаляйте и не редактируйте `.github/workflows/hexlet-check.yml` и не
переименовывайте репозиторий — это автопроверка Хекслета.

`.github/workflows/opencode.yml` запускает агента-ассистента по комментариям,
начинающимся с `/oc` (или содержащим `/opencode`). Запросы к нему формулируйте
явно; ключ провайдера лежит в секрете `OPENCODE_API_KEY` — не выносите секреты в
код, комменты и выводы агента.

## Agent skills

### Трекер задач

Задачи живут в GitHub Issues этого репозитория (через CLI `gh`). См.
`docs/agents/issue-tracker.md`.

### Метки триажа

Пять канонических ролей триажа отображаются 1:1 на метки: `needs-triage`,
`needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. См.
`docs/agents/triage-labels.md`.

### Документы предметной области

Один контекст: один `GLOSSARY.md` + `docs/adr/` в корне репозитория. См.
`docs/agents/domain.md`.
