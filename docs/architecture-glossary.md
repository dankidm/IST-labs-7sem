# Глоссарий архитектурных идентификаторов

ID обозначает один и тот же логический элемент во всех представлениях.
Основой служат сводные L1/L2; этот документ фиксирует их состав для ревью.

## Система роли и внешние системы

| ID | Тип | Название |
| --- | --- | --- |
| `marketer` | `Person` | Маркетолог |
| `bd_manager` | `Person` | Менеджер по развитию бизнеса |
| `marketing_head` | `Person` | Руководитель маркетинга |
| `kb_editor` | `Person` | Контент-менеджер базы знаний |
| `analyst` | `Person` | Аналитик рынка |
| `platform_admin` | `Person` | Администратор платформы |
| `kb_platform` | `System` | ИИ-платформа базы знаний и маркетинга |
| `doc_sources` | `System_Ext` | Корпоративные хранилища документов |
| `crm_system` | `System_Ext` | CRM компании |
| `corporate_idp` | `System_Ext` | Корпоративный каталог пользователей |
| `llm_provider` | `System_Ext` | Внешние ИИ-сервисы |
| `market_sources` | `System_Ext` | Источники рыночной аналитики |
| `bi_platform` | `System_Ext` | BI-платформа |
| `notification_gateway` | `System_Ext` | Сервисы доставки уведомлений |
| `payment_gateway` | `System_Ext` | Платёжный шлюз |

## Контейнеры L2

| ID | Тип | Название |
| --- | --- | --- |
| `web_app` | `Container` | Веб-приложение |
| `mobile_app` | `Container` | Мобильное приложение |
| `agent_service` | `Container` | ИИ-агент |
| `generation_service` | `Container` | Сервис генерации материалов |
| `auth_service` | `Container` | Сервис авторизации |
| `core_api` | `Container` | Сервис базы знаний (ядро) |
| `ingestion_service` | `Container` | Сервис обработки документов |
| `vector_db` | `ContainerDb` | Векторная БД |
| `market_collector` | `Container` | Сервис сбора рыночной аналитики (ETL) |
| `analytics_service` | `Container` | Сервис аналитики и инсайтов |
| `dwh` | `ContainerDb` | Аналитическое хранилище |
| `api_gateway` | `Container` | API-шлюз |
| `database` | `ContainerDb` | Реляционная СУБД |
| `message_broker` | `ContainerQueue` | Брокер сообщений |
| `cache` | `ContainerDb` | Кэш и сессии |
| `object_storage` | `ContainerDb` | Объектное хранилище |
| `llm_server` | `Container` | Локальный LLM-сервер |
| `monitoring` | `Container` | Система наблюдаемости |

## Компоненты L3

| Зона | ID | Название |
| --- | --- | --- |
| backend | `kb_api_comp` | API-маршрутизатор |
| backend | `auth_comp` | Модуль авторизации |
| backend | `kb_comp` | Сервис базы знаний |
| backend | `search_comp` | Сервис гибридного поиска |
| backend | `billing_comp` | Модуль подписок |
| backend | `kb_repo_comp` | Репозиторий данных |
| backend | `index_comp` | Клиент векторного индекса и кэша |
| backend | `storage_comp` | Клиент объектного хранилища |
| backend | `event_comp` | Шина событий |
| backend | `integration_comp` | Модуль внешних интеграций |
| analyst | `analytics_api_comp` | API аналитики |
| analyst | `metrics_comp` | Сервис показателей |
| analyst | `benchmark_comp` | Сервис сопоставления с рынком |
| analyst | `report_comp` | Сервис отчётов и дайджестов |
| analyst | `export_comp` | Модуль экспорта |
| analyst | `dwh_repo_comp` | Репозиторий витрин |
| analyst | `an_storage_comp` | Клиент хранилища отчётов |
| analyst | `an_event_comp` | Публикатор событий |
| vibecoder | `agent_api_comp` | API агента |
| vibecoder | `orchestrator_comp` | Оркестратор агента |
| vibecoder | `prompt_comp` | Сборщик промптов |
| vibecoder | `retrieval_comp` | Инструмент поиска знаний |
| vibecoder | `market_tool_comp` | Инструмент рыночных показателей |
| vibecoder | `llm_comp` | Шлюз языковых моделей |
| vibecoder | `memory_comp` | Память диалога |
| vibecoder | `dialog_repo_comp` | Репозиторий диалогов |

## Экземпляры на развёртывании

Псевдонимы `core_api_1/2`, `auth_service_1/2`, `agent_service_1/2`,
`generation_service_1/2` и `analytics_service_1/2` обозначают два экземпляра
соответствующего логического контейнера L2. `database_replica` — физическая
реплика `database`. ID узлов размещения не являются ID контейнеров.

## Правила изменений

- Повторное использование ID сохраняет назначение элемента.
- Новый общий ID или изменение протокола согласуется в PR с владельцами затронутых зон.
- Межпроцессные связи указывают протокол и порт; внутрипроцессные — `Python Call`.
- C4-связи используют `Rel()`. Направляющие макросы `Rel_R/Rel_L/Rel_D/Rel_U` не применяются.
- Глоссарий служит для ручного ревью. Текущая проверка PlantUML в CI сама по себе не подтверждает семантическое соответствие этому документу.
