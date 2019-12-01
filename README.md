# Аналитика посещаемости сайта в clickHouse
1. Создание таблицы для логирования посещаемости сайта:

```clickhouse
create table visits
(
	UserID UInt32,
	RegionID UInt32,
	url String,
	visitAt DateTime
)
engine = MergeTree PARTITION BY ((visitAt, UserID), (visitAt, RegionID)) ORDER BY (visitAt, UserID, RegionID)  SETTINGS index_granularity = 8192;

```

2.  Запись логов в базу данных лучше осуществлять пакетами. Для  этого можно использовать стандартные механизмы логирования HTTP-сервер, например Apache, в нем храниться вся необходимая информация, которую в дальнейшем можно будет использовать для записи пакета в базу данных в базу данных.

3. Запрос на получение статистики по посещаемости сайта по дням за последний месяц в разрезе стран, из которых пользователи заходили на сайт

```clickhouse
SELECT RegionID, toYYYYMM(visitAt),  count(*) AS count FROM visits WHERE toYYYYMM(visitAt) = toYYYYMM(now()) GROUP BY RegionID, toYYYYMM(visitAt) ORDER BY RegionID ASC, toYYYYMM(visitAt) ASC
```

4. Запрос на удаление неактуальных данных, более одного года, по пользователям:
```clickhouse
ALTER TABLE visits DELETE WHERE toDate(visitAt) < toDate(subtractYears(now(), 1))
```
