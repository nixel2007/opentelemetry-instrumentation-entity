---
title: Семантические соглашения
---

# Семантические соглашения

Какие атрибуты [семантических соглашений OpenTelemetry](https://opentelemetry.io/docs/specs/semconv/)
заполняет `opentelemetry-instrumentation-entity`. В таблицах — stable-атрибуты и метрики semconv
(версия 1.44.0); атрибуты и метрики в статусе development и устаревшие атрибуты отмечены отдельно.
Группы атрибутов названы так же, как в semconv.

Ключи `entity.*` — собственные атрибуты библиотеки, они описаны в [обзоре](/opentelemetry-instrumentation-entity/index.md).

## Спаны запросов

[Database client spans](https://opentelemetry.io/docs/specs/semconv/db/database-spans/)

Спан вида `CLIENT` на каждый запрос к СУБД.

| Атрибут | Значение |
|---|---|
| `db.system.name` | система СУБД коннектора: `postgresql`, `sqlite`, `inmemory`, `json` |
| `db.namespace` | база данных, путь к файлу или каталогу |
| `db.collection.name` | таблица запроса |
| `db.operation.name` | `SELECT`, `INSERT`, `DELETE`, `CREATE TABLE`, `BEGIN`, `COMMIT`, `ROLLBACK` |
| `db.query.text` | текст запроса с плейсхолдерами; у коннекторов без SQL не заполняется, выключается настройкой `ТекстЗапроса` |
| `server.address`, `server.port` | сервер СУБД; у файловых баз не заполняются |
| `error.type` | тип ошибки, если запрос завершился ошибкой, см. [Error](#error) |

Из значений `db.system.name` в semconv описаны `postgresql` и `sqlite`, `inmemory` и `json` — собственные.

### Атрибуты в статусе development

| Атрибут | Значение |
|---|---|
| `db.response.returned_rows` | число строк, которые вернул `SELECT` |

## Спаны операций

Спан вида `INTERNAL` на каждую операцию менеджера сущностей.

### Code

[Code](https://opentelemetry.io/docs/specs/semconv/registry/attributes/code/)

| Атрибут | Значение |
|---|---|
| `code.function.name` | операция: `Сохранить`, `Получить`, `ПолучитьОдно`, `Удалить`, `Инициализировать`, `ВыполнитьСКоннектором`, `ВычислитьСКоннектором` |
| `error.type` | тип ошибки, если операция завершилась ошибкой, см. [Error](#error) |

### Устаревшие атрибуты

| Атрибут | Значение |
|---|---|
| `code.namespace` | `МенеджерСущностей`. В semconv атрибут устарел: пространство имён входит в полное имя `code.function.name` |

## Метрики

[Database client metrics](https://opentelemetry.io/docs/specs/semconv/db/database-metrics/)

| Метрика | Атрибуты |
|---|---|
| `db.client.operation.duration` | `db.system.name`, `db.namespace`, `db.collection.name`, `db.operation.name`, `server.address`, `server.port`, `error.type` |

Длительность — в секундах, с границами бакетов из semconv.

### Метрики в статусе development

| Метрика | Атрибуты |
|---|---|
| `db.client.connection.count` | `db.client.connection.pool.name`, `db.client.connection.state`: `used`, `idle` |
| `db.client.connection.max` | `db.client.connection.pool.name` |
| `db.client.connection.pending_requests` | `db.client.connection.pool.name` |
| `db.client.connection.wait_time` | `db.client.connection.pool.name` |
| `db.client.connection.create_time` | `db.client.connection.pool.name` |
| `db.client.connection.timeouts` | `db.client.connection.pool.name` |

Атрибуты `db.client.connection.pool.name` и `db.client.connection.state` — тоже в статусе development.

Гистограммы `entity.operation.duration` и `entity.repository.invocation.duration` — собственные
метрики библиотеки. Из атрибутов semconv в них есть `error.type`, а в
`entity.repository.invocation.duration` ещё `code.function.name` — имя метода хранилища сущностей.

## Исключения

[Exceptions in logs](https://opentelemetry.io/docs/specs/semconv/exceptions/exceptions-logs/)

Исключение операции или запроса записывается записью лога с серьёзностью `WARN`. Атрибуты
`exception.*` заполняет ядро из переданного исключения, см. [семантические соглашения opentelemetry](/opentelemetry/semconv.md).

| Событие | Статус в semconv |
|---|---|
| `db.client.operation.exception` | development, [Database exceptions](https://opentelemetry.io/docs/specs/semconv/db/database-exceptions/) |
| `entity.operation.exception` | собственное событие библиотеки |

## Error

[Error](https://opentelemetry.io/docs/specs/semconv/registry/attributes/error/)

| Атрибут | Значение |
|---|---|
| `error.type` | [ОтелУтилиты.ТипОшибки](/api/opentelemetry/Ядро/ОтелУтилиты.md): код ошибки библиотеки errors или код нативной ошибки OneScript, для нераспознанной ошибки — `_OTHER`. Спан и метрика одного события получают одно значение |
