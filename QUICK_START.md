# Quick Start — Python проект

## Что нужно установить

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- [Context7 MCP](https://github.com/upstash/context7) — актуальная документация библиотек
- [uv](https://docs.astral.sh/uv/getting-started/installation/) — пакетный менеджер Python
- Python 3.13+: `uv python install 3.13`
- GitHub CLI: `gh auth login`

---

## Новый проект

### 1. Создать папку и скопировать фреймворк

```bash
mkdir my-project && cd my-project
cp -rp ~/.claude/. .claude/
```

### 2. Инициализировать проект

```
/init-project
```

Создаёт `.claude/` со структурой Project Knowledge, `CLAUDE.md`, `.gitignore`, инициализирует git, создаёт приватный GitHub-репозиторий, ветки `main` и `dev`.

### 3. Заполнить документацию проекта

```
/init-project-knowledge
```

Интервью о проекте → заполняет `architecture.md`, `patterns.md`, `deployment.md`, создаёт бэклог фич.

**Обязательно укажи в ответах на интервью:**
- Язык: Python 3.13
- Пакетный менеджер: uv
- Тесты: pytest
- Линтер/форматтер: ruff
- Типизация: mypy (strict)

### 4. Настроить инфраструктуру

```
/write-code настрой инфраструктуру проекта
```

Агент запустит скилл `infrastructure-setup`, который создаст:

```
my-project/
├── src/
│   └── {package_name}/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── test_smoke.py
├── pyproject.toml
├── .pre-commit-config.yaml   # gitleaks + ruff
├── .env.example
└── .gitignore
```

Готовые шаблоны лежат в `~/.claude/shared/templates/infrastructure/`.

---

## Разработка фичи

```
/new-user-spec      → интервью → user-spec.md (что делаем)
/new-tech-spec      → исследование кода → tech-spec.md (как делаем)
/decompose-tech-spec → разбивка на задачи → tasks/*.md
/do-feature         → команда агентов выполняет все задачи параллельно
/done               → обновляет документацию, архивирует фичу
```

### Одна задача вместо всей фичи

```
/do-task            → TDD → код → ревью → коммит
```

### Код без спецификации

```
/write-code         → план → тесты → код → ревью
```

---

## Структура Python-проекта

```
my-project/
├── src/
│   └── {package_name}/
│       ├── __init__.py
│       ├── main.py
│       └── ...
├── tests/
│   ├── unit/
│   │   └── test_*.py
│   ├── integration/
│   │   └── test_*.py
│   └── test_smoke.py
├── pyproject.toml          # зависимости, mypy, ruff, pytest
├── uv.lock
├── .pre-commit-config.yaml
├── .env                    # не коммитить
├── .env.example            # коммитить
├── .gitignore
└── .claude/
    └── skills/
        └── project-knowledge/
            └── references/
                ├── project.md
                ├── architecture.md
                ├── patterns.md
                └── deployment.md
```

---

## Полезные команды

```bash
uv sync                  # установить зависимости
uv add {package}         # добавить зависимость
uv add --dev {package}   # добавить dev-зависимость
uv run pytest            # запустить тесты
uv run pytest --cov=src  # с coverage
uv run ruff check .      # линтер
uv run ruff format .     # форматтер
uv run mypy src/         # проверка типов
pre-commit run --all-files  # все хуки разом
```

---

## CI/CD

GitHub Actions конфиг: `~/.claude/shared/templates/infrastructure/.github/workflows/ci.yml.template`

Скопируй в `.github/workflows/ci.yml` и замени `{package_name}` на имя твоего пакета.

Pipeline: lint → typecheck → tests → deploy (только на main).

---

## Справка

| Команда | Что делает |
|---------|-----------|
| `/init-project` | Создаёт структуру, git, GitHub-репозиторий |
| `/init-project-knowledge` | Интервью → заполняет документацию проекта |
| `/new-user-spec` | Интервью → спецификация требований |
| `/new-tech-spec` | Исследование кода → техническая спецификация |
| `/decompose-tech-spec` | Техспек → атомарные задачи с TDD |
| `/do-feature` | Команда агентов выполняет фичу параллельно |
| `/do-task` | Одна задача: TDD → код → ревью |
| `/write-code` | Код без спецификации: план → TDD → ревью |
| `/done` | Обновляет документацию, архивирует фичу |
