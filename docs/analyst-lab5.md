# Интеграция архитектурных диаграмм аналитика

Лабораторная работа №5 по дисциплине «Информационные системы и технологии».
Автор: Даниил Жарков, группа ИДБ-23-14, роль в команде — аналитик.

## Зона ответственности

Контур аналитики ИИ-платформы базы знаний и маркетинга: сбор рыночных данных,
витрины ClickHouse, показатели, отчёты и еженедельные дайджесты.
В L3 раскрыт контейнер `analytics_service`, в L4 — компонент `export_comp`.

| Представление | Исходный файл | Изображение после CI |
| --- | --- | --- |
| Контекст L1 | `src/context/analyst_context.puml` | `analyst_context.png` |
| Контейнеры L2 | `src/containers/analyst_containers.puml` | `analyst_containers.png` |
| Компоненты L3 | `src/components/analyst_components.puml` | `analyst_components.png` |
| Классы L4 | `src/classes/analyst_classes.puml` | `analyst_classes.png` |
| Ландшафт L0 | `src/landscape/analyst_landscape.puml` | `analyst_landscape.png` |
| Динамика | `src/dynamic/analyst_dynamic.puml` | `analyst_dynamic.png` |
| Развёртывание | `src/deployment/analyst_deployment.puml` | `analyst_deployment.png` |

## Согласование L2 и L3

На L3 входящий запрос `agent_service → analytics_api_comp` уже был указан,
а локальная диаграмма L2 аналитика не показывала вызывающий контейнер и его
связь с сервисом аналитики. При интеграции на L2 добавлены `agent_service`
и связь №44 с `analytics_service`. ID, назначение и протокол совпадают
с диаграммой контейнеров vibecoder и диаграммой компонентов аналитика.

| Связь | На уровне L2 | На уровне L3 | Протокол |
| --- | --- | --- | --- |
| 25 | `api_gateway → analytics_service` | `api_gateway → analytics_api_comp` | HTTP/JSON, TCP 8004 |
| 44 | `agent_service → analytics_service` | `agent_service → analytics_api_comp` | HTTP/JSON, TCP 8004 |
| 61 | `analytics_service → dwh` | `dwh_repo_comp → dwh` | HTTP, TCP 8123 |
| 62 | `analytics_service → object_storage` | `an_storage_comp → object_storage` | HTTPS (S3 API), TCP 9000 |
| 63 | `analytics_service → message_broker` | `an_event_comp → message_broker` | AMQP, TCP 5672 |

### Идентификаторы компонентов

| ID | Назначение |
| --- | --- |
| `analytics_api_comp` | Принимает запросы к аналитике |
| `metrics_comp` | Рассчитывает показатели использования и эффективности |
| `benchmark_comp` | Сопоставляет проекты компании с рыночными показателями |
| `report_comp` | Формирует отчёты и запускает дайджесты по расписанию |
| `export_comp` | Преобразует отчёт в CSV, XLSX или PDF |
| `dwh_repo_comp` | Выполняет запросы к витринам ClickHouse |
| `an_storage_comp` | Сохраняет файлы в MinIO |
| `an_event_comp` | Публикует событие `report.ready` |

Вызовы внутри `analytics_service` имеют тип `Python Call`, без сетевого порта.
На внешних связях указаны сетевой протокол и порт. На C4-диаграммах применён
стандартный `Rel()`; направляющие макросы `Rel_R`, `Rel_L`, `Rel_D`, `Rel_U`
не используются.

## Стык L3 и L4

`report_comp` передаёт готовый документ в модуль экспорта. Реализованный
публичный контракт `ExportService`:

```python
export(document: ReportDocument, fmt: ExportFormat) -> bytes
content_type(fmt: ExportFormat) -> str
filename(document: ReportDocument, fmt: ExportFormat) -> str
set_strategy(strategy: ExportStrategy) -> None
```

Реализация находится в `code/analyst_code/analyst/export_comp/service.py`.
Экспорт возвращает байты; сохранение выполняет клиент хранилища,
публикацию `report.ready` — публикатор событий. Экспорт сам не рассылает письма.

## Локальный PlantUML Server в Docker

После запуска Docker Desktop контейнер создаётся один раз:

```bash
docker run -d \
  --name ist-lab5-plantuml \
  -p 127.0.0.1:8080:8080 \
  plantuml/plantuml-server:jetty
```

Для уже существующего остановленного контейнера:

```bash
docker start ist-lab5-plantuml
```

Проверка:

```bash
docker ps --filter name=ist-lab5-plantuml
```

Ожидаются статус `Up` и публикация `127.0.0.1:8080->8080/tcp`.
В браузере открывается `http://localhost:8080`.
В `.vscode/settings.json` задано:

```json
{
  "plantuml.render": "PlantUMLServer",
  "plantuml.server": "http://localhost:8080"
}
```

Открыть `.puml` и вызвать предпросмотр Alt+D (Option+D на macOS).
При выбранном `PlantUMLServer` изображение создаёт сервер в контейнере.
Подключения C4 через `!include https://raw.githubusercontent.com/...`
по-прежнему требуют доступа к сети.

## Автоматическая сборка

Конфигурация команды: `.github/workflows/render-docs.yml`.
Workflow запускается для каждого Pull Request в `main`, после push в `main`
и вручную через `workflow_dispatch`. Фильтров по путям нет, поэтому проверка
выполняется и для PR, меняющих только документацию или настройки VS Code.

Этапы: получение исходников, установка Java и Graphviz, загрузка PlantUML,
проверка синтаксиса всех `src/**/*.puml`, генерация PNG во временную папку
runner и публикация артефакта `diagrams`. При ошибке синтаксиса проверка
завершается с ненулевым кодом. Workflow не коммитит изображения в Git:
актуальные PNG находятся в артефакте, а `docs/images` содержит прежние снимки.
Во втором столбце таблицы выше приведены исходники, в третьем — имена PNG
внутри артефакта. Срок хранения артефактов — 90 дней.

Подтверждённая сборка после интеграции тимлида:
[Render PlantUML docs №5](https://github.com/dankidm/IST-labs-7sem/actions/runs/45301593616).
Она запущена push в `main` после слияния PR №13 24.11.2026, завершилась
со статусом Success за 1 минуту 56 секунд и сформировала артефакт `diagrams`
размером 5,45 MB. Это исходная сборка; она не подтверждает результат нового PR аналитика.

## Учебная проверка обработки ошибки

В отдельной ветке аналитика планируется убрать закрывающую кавычку
в аргументе протокола связи №44, получить отказ этапа `Check syntax`,
сохранить лог и убедиться, что обязательная проверка `render` блокирует слияние.
Следующий коммит должен восстановить кавычку и пройти CI.
Ошибка вводится намеренно для проверки механизма; она не является
обнаруженным дефектом исходного проекта. В `main` должна попасть исправленная версия.
Факт выполнения, номера запусков и результаты фиксируются после проверки.

## Что проверить при ревью этого изменения

- Добавленная связь №44 согласована с L3 и с представлением vibecoder.
- ID существующих контейнеров и компонентов сохранены.
- У всех связей L2 есть описание и протокол.
- Обновлённая диаграмма открывается через Docker-сервер.
- Проверка `render` в новом PR завершилась успешно, артефакт `diagrams` доступен.
- Тимлид оставил содержательное ревью до слияния PR.

Статус и логи нового PR, замечания ревью и итоговое слияние фиксируются
после их фактического выполнения и включаются в индивидуальный отчёт.
