# opentelemetry-instrumentation-entity

[![Quality Gate Status](https://sonar.openbsl.ru/api/project_badges/measure?project=opentelemetry-instrumentation-entity&metric=alert_status)](https://sonar.openbsl.ru/dashboard?id=opentelemetry-instrumentation-entity)
[![Coverage](https://sonar.openbsl.ru/api/project_badges/measure?project=opentelemetry-instrumentation-entity&metric=coverage)](https://sonar.openbsl.ru/dashboard?id=opentelemetry-instrumentation-entity)

Инструментирование [entity](https://github.com/nixel2007/entity) для [OpenTelemetry SDK](https://github.com/nixel2007/opentelemetry): операции менеджера сущностей, запросы к СУБД, соединения и транзакции его источника данных становятся спанами и метриками. Аналог инструментирования JDBC и Hibernate в Java: ORM ничего не знает об OpenTelemetry, а библиотека подписывается на его события через интерфейс `НаблюдательИсточникаДанных`.

## Установка

```sh
opm install opentelemetry-instrumentation-entity
```

Требует entity 4.4 и OneScript 2.2: контекст исполнения обе библиотеки ведут в данных потока исполнения (`ТекущийПоток`).

## Быстрый старт

```bsl
#Использовать entity
#Использовать opentelemetry
#Использовать opentelemetry-instrumentation-entity

Сдк = ОтелАвтоконфигурация.Инициализировать();

МенеджерСущностей = Новый МенеджерСущностей(Тип("КоннекторPostgreSQL"), СтрокаСоединения);
МенеджерСущностей.ДобавитьКлассВМодель(Тип("Автор"));
МенеджерСущностей.ДобавитьНаблюдателя(Новый ОтелНаблюдательИсточникаДанных(
    Сдк.ПолучитьТрассировщик("entity"),
    Сдк.ПолучитьМетр("entity")
));
МенеджерСущностей.Инициализировать();

Авторы = МенеджерСущностей.Получить(Тип("Автор")); // спан "Получить Автор" с дочерним "SELECT Авторы"
```

Наблюдатель, зарегистрированный до `Инициализировать`, видит и создание таблиц. В приложениях на [Autumn](https://github.com/autumn-library/autumn) регистрацию выполняет `autumn-opentelemetry` вместе с `autumn-data`, вручную ничего делать не нужно.

## Сигналы

| Сигнал | Имя | Атрибуты |
| --- | --- | --- |
| Спан INTERNAL | `{Операция} {ТипСущности}` - `Сохранить Автор`, `Получить Автор`, `Инициализировать` | `code.namespace`, `code.function.name`, `entity.type`, `entity.table`, `entity.depth`, `entity.result.count`, `error.type` |
| Спан CLIENT | `{Операция} {Таблица}` - `SELECT Авторы`, `INSERT Авторы`, `COMMIT` | `db.system.name`, `db.namespace`, `db.collection.name`, `db.operation.name`, `db.query.text`, `db.response.returned_rows`, `server.address`, `server.port`, `error.type` |
| Гистограмма, с | `db.client.operation.duration` | `db.system.name`, `db.namespace`, `db.collection.name`, `db.operation.name`, `server.address`, `server.port`, `error.type` |
| Гистограмма, с | `entity.operation.duration` | `entity.operation`, `entity.type`, `error.type` |
| Счетчик | `entity.entities` | `entity.operation`, `entity.type` |
| Счетчик | `entity.transactions` | `entity.transaction.result`: `commit`, `rollback`, `failed`, `abandoned` |
| Датчик | `db.client.connection.count` | `db.client.connection.pool.name`, `db.client.connection.state`: `used`, `idle` |
| Датчик | `db.client.connection.max`, `db.client.connection.pending_requests` | `db.client.connection.pool.name` |
| Гистограмма, с | `db.client.connection.wait_time`, `db.client.connection.create_time` | `db.client.connection.pool.name` |
| Счетчик | `db.client.connection.timeouts` | `db.client.connection.pool.name` |

Спаны вложены как вызовы: операция из прикладного кода - корень, разыменование ссылок и чтение подчиненных таблиц - дочерние операции, запросы к СУБД - листья. Каскад и N+1 видны как вложенность. `BEGIN`, `COMMIT` и `ROLLBACK` - обычные спаны запроса, долгоживущего спана транзакции нет.

Ключи `db.*`, `server.*`, `error.type` и `code.*` берутся из модуля `ОтелСемантическиеСоглашения` SDK и совпадают с semantic conventions OpenTelemetry; гистограммы длительности - в секундах с границами бакетов из соглашений. Ключи `entity.*` - собственные: значения в них могут быть кириллическими (имена операций и типов), ключи - латинские.

Текст запроса в `db.query.text` содержит плейсхолдеры, а не значения параметров. Описание соединения (`db.namespace`, `server.address`, имя пула) не содержит пароля.

## Настройки

Третий параметр конструктора - структура настроек:

| Ключ | По умолчанию | Действие |
| --- | --- | --- |
| `ТекстЗапроса` | `Истина` | Писать текст запроса в `db.query.text` |
| `Метрики` | `Истина` | Регистрировать инструменты и писать метрики |

```bsl
Наблюдатель = Новый ОтелНаблюдательИсточникаДанных(Трассировщик, Метр, Новый Структура("ТекстЗапроса", Ложь));
```

`Неопределено` вместо трассировщика выключает спаны, вместо метра - метрики. Выключенный трассировщик SDK дает незаписывающие спаны, метрики при этом пишутся.

## Документация

- [Руководство](docs/product/010-index.md)
- [Справочник API](docs/api/ОтелНаблюдательИсточникаДанных.md)
- [Наблюдатели источника данных в entity](https://github.com/nixel2007/entity/blob/master/docs/Наблюдатели.md)

## Лицензия

MIT License. Подробности в файле [LICENSE.md](LICENSE.md).
