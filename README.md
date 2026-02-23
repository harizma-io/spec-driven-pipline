# Claude Code Framework

Фреймворк для AI-First разработки с Claude Code — 19 skills, 19 агентов, 9 команд.

## Что это?

Коллекция методологий, автоматизаций и шаблонов для [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code). Решает ключевую проблему — **потерю контекста между сессиями** через:

- Распределённую базу знаний вместо монолитного CLAUDE.md
- Spec-driven pipeline: Context → User Spec → Tech Spec → Tasks → Code
- Оркестрацию специализированных агентов с quality gates
- Автоматизацию качества на каждом этапе (review, тесты, security audit)

## Что здесь лежит

| Папка | Описание |
|---|---|
| `skills/` | Модульные пакеты знаний — методологии, workflow, гайды |
| `agents/` | Специализированные субагенты для валидации, ревью, QA |
| `commands/` | Slash-команды (`/init-project`, `/do-task`, ...) |
| `shared/` | Шаблоны проектов, интервью, work-директорий |
| `CLAUDE.md` | Пример глобального CLAUDE.md для методологии |

## Как работает методология

```
Планирование проекта          Разработка фичи
─────────────────────          ──────────────────────────────────────

/init-project                  /new-user-spec
    ↓                              ↓
project-planning skill         Интервью → user-spec.md
    ↓                              ↓
documentation-writing          /new-tech-spec
                                   ↓
                               tech-spec.md + tasks/*.md
                                   ↓
                               /do-task (или /do-feature для параллельного выполнения)
                                   ↓
                               TDD → Code → Review → Security Audit
                                   ↓
                               Pre-deploy QA → Deploy → Post-deploy QA
```

Два режима выполнения:
- **`/do-task`** — последовательно, по одной задаче за раз
- **`/do-feature`** — параллельно, команда агентов выполняет задачи волнами

## Требования

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- [Context7 MCP сервер](https://github.com/upstash/context7) — для доступа к актуальной документации библиотек

---

## Skills

### Планирование

| Skill | Описание |
|---|---|
| `methodology` | AI-First методология: spec-driven pipeline, структура проектов, экосистема skills/agents |
| `project-planning` | Планирование нового проекта: адаптивное интервью, заполнение project-knowledge и backlog |
| `user-spec-planning` | Создание user-spec.md через интервью с пользователем, сканирование кодовой базы, валидация |
| `tech-spec-planning` | Создание tech-spec.md: архитектура, решения, стратегия тестирования, план реализации |
| `task-decomposition` | Декомпозиция tech-spec на атомарные задачи с параллельным созданием и валидацией |

### Разработка

| Skill | Описание |
|---|---|
| `code-writing` | Универсальный процесс написания кода: план, TDD, ревью |
| `feature-execution` | Оркестрация фичи как тимлид: агенты по волнам, циклы ревью, коммит за волну |
| `infrastructure-setup` | Настройка инфраструктуры: фреймворк, Docker, pre-commit hooks, тесты, .gitignore |
| `deploy-pipeline` | Настройка CI/CD: GitHub Actions, деплой на платформы, управление секретами |

### Качество

| Skill | Описание |
|---|---|
| `code-reviewing` | Методология code review: 10 измерений проверки, процесс, стандарты качества |
| `test-master` | Методология тестирования: тестовая пирамида, когда какие тесты писать, стратегия |
| `security-auditor` | Аудит безопасности по OWASP Top 10: SQL injection, XSS, auth, криптография |
| `pre-deploy-qa` | Приёмочное тестирование перед деплоем: запуск тестов, проверка acceptance criteria |
| `post-deploy-qa` | Пост-деплой верификация: AVP на живом окружении через MCP-инструменты |

### Документация

| Skill | Описание |
|---|---|
| `documentation-writing` | Управление project-knowledge: создание, проверка, обновление базы знаний проекта |
| `prompt-master` | Гайд по написанию эффективных промптов для LLM |

### Мета

| Skill | Описание |
|---|---|
| `skill-master` | Гайд по созданию и обновлению skills |
| `skill-test-designer` | Проектирование тестовых сценариев для skills через интервью |
| `skill-tester` | Запуск тестовых сценариев с параллельными раннерами и детальным отчётом |

---

## Агенты

Субагенты запускаются через Task tool для специализированных задач.

### Валидаторы

| Агент | Описание |
|---|---|
| `tech-spec-validator` | Валидация структуры tech-spec, секций, стандартов |
| `task-validator` | Валидация task-файлов по шаблону и правилам |
| `task-creator` | Создание task-файлов из секции Implementation Tasks tech-spec |
| `completeness-validator` | Двусторонняя трассировка требований: user-spec ↔ tech-spec/tasks |
| `reality-checker` | Проверка task-файлов на соответствие реальной кодовой базе |
| `skeptic` | Верификация фактов в tech-spec/tasks — поиск миражей и несуществующих файлов |
| `userspec-quality-validator` | Валидация качества user-spec: структура, покрытие, тестируемость критериев |
| `userspec-adequacy-validator` | Валидация адекватности user-spec: осуществимость, масштаб, over/under-engineering |
| `interview-completeness-checker` | Оценка полноты интервью для user-spec planning |
| `skill-checker` | Валидация skills по стандартам skill-master |

### Ревьюеры

| Агент | Описание |
|---|---|
| `code-reviewer` | Code review реализации: архитектура, читаемость, error handling, тесты |
| `code-researcher` | Исследование кодовой базы: файлы, паттерны, тесты, интеграции, риски |
| `test-reviewer` | Анализ качества тестов: находит проблемы и даёт конкретные исправления |
| `deploy-reviewer` | Ревью CI/CD pipeline: GitHub Actions, secrets management, конфигурация |
| `infrastructure-reviewer` | Ревью инфраструктуры: структура, Docker, pre-commit hooks, .gitignore |
| `prompt-reviewer` | Ревью качества LLM-промптов по принципам prompt-master |
| `security-auditor` | Аудит безопасности по OWASP Top 10: код и архитектурные решения |

### QA

| Агент | Описание |
|---|---|
| `pre-deploy-qa` | Приёмочное тестирование: запуск тестов, проверка acceptance criteria |
| `post-deploy-qa` | Пост-деплой верификация на живом окружении через MCP-инструменты |

---

## Команды

| Команда | Описание |
|---|---|
| `/init-project` | Инициализация проекта: шаблон, git, GitHub |
| `/init-project-knowledge` | Первичное заполнение документации проекта |
| `/new-user-spec` | Создание user specification через интервью |
| `/new-tech-spec` | Создание технической спецификации и задач |
| `/decompose-tech-spec` | Декомпозиция tech-spec на атомарные задачи |
| `/do-task` | Выполнение задачи с TDD и quality gates |
| `/do-feature` | Параллельное выполнение фичи командой агентов |
| `/write-code` | Написание кода с процессом качества |
| `/done` | Финализация фичи: обновление документации, архивация |

---

## Структура репозитория

```
claude-code-framework/
├── CLAUDE.md                          # Пример глобального CLAUDE.md
├── skills/
│   ├── methodology/                   # AI-First методология
│   ├── project-planning/              # Планирование проектов
│   ├── user-spec-planning/            # User specifications
│   ├── tech-spec-planning/            # Technical specifications
│   ├── task-decomposition/            # Декомпозиция на задачи
│   ├── code-writing/                  # Процесс написания кода
│   ├── code-reviewing/                # Методология code review
│   ├── feature-execution/             # Оркестрация фичи
│   ├── infrastructure-setup/          # DevOps инфраструктура
│   ├── deploy-pipeline/               # CI/CD pipeline
│   ├── test-master/                   # Стратегия тестирования
│   ├── security-auditor/              # Аудит безопасности
│   ├── pre-deploy-qa/                 # Pre-deploy QA
│   ├── post-deploy-qa/                # Post-deploy QA
│   ├── documentation-writing/         # Управление документацией
│   ├── prompt-master/                 # Промпт-инжиниринг
│   ├── skill-master/                  # Создание skills
│   ├── skill-test-designer/           # Тесты для skills
│   └── skill-tester/                  # Запуск тестов skills
├── agents/                            # 19 субагентов
├── commands/                          # 9 slash-команд
└── shared/
    ├── templates/
    │   ├── new-project/               # Шаблон нового проекта
    │   └── infrastructure/            # Шаблоны инфраструктуры
    ├── interview-templates/           # Шаблоны интервью
    └── work-templates/                # Шаблоны work-директории
```

---

## Лицензия

MIT License — используйте свободно.

## Автор

Pavel Molyanov — [@pavel-molyanov](https://github.com/pavel-molyanov)
