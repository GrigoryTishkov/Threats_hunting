# Практическая работа с использованием СУБД DuckDB

gtishckov@yandex.ru

## Анализ данных сетевого трафика с использованием аналитической in-memory СУБД DuckDB

### Цель работы

1. Изучить возможности СУБД DuckDB для обработки и анализ больших данных
2. Получить навыки применения DuckDB совместно с языком программирования R
3. Получить навыки анализа  метаинфомации о сетевом трафике
4. Получить навыки применения облачных технологий хранения, подготовки и анализа данных: Yandex Object Storage, Rstudio Server

### Исходные данные

1. Программное обеспечение Windows 11
2. Библиотека dplyr
3. Библиотека DuckDB
4. Rstudio Server

### План

1. Подключиться по ssh к Rstudio Server
2. Выполнить задания

### Шаги

1. Скачиваем данные с поомщью функции download.file
```{r}
#download.file("https://storage.yandexcloud.net/arrow-datasets/tm_data.pqt",destfile = "tm_data.pqt")
```
2. Загружаем данные пакетов в таблицу tbl
```{r}
library(duckdb)
library(dplyr)
```
```{r}
con <- dbConnect(duckdb())
dbExecute(con,"CREATE TABLE tbl as SELECT * FROM read_parquet('tm_data.pqt')")
```
```{r}
[1] 105747730
```
3. Приступаем к выполнению заданий

a) Найдите утечку данных из вашей сети

```{r}
dbGetQuery(con,"SELECT src FROM tbl
WHERE (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%') 
AND NOT (dst LIKE '12.%' AND dst LIKE '13.%' AND dst LIKE '14.%')
GROUP BY src
order by sum(bytes) desc
limit 1")
```
```{r}
      src
1 13.37.84.125
```
b) Найдите утечку данных из вашей сети 2

```{r}
dbGetQuery(con,"SELECT 
    time,
    COUNT(*) AS trafictime
FROM (
    SELECT 
        timestamp,
        src,
        dst,
        bytes,
        (
            (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
            AND (dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%')
        ) AS trafic,
        EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS time
    FROM tbl
) sub
WHERE trafic = TRUE AND time BETWEEN 0 AND 24
GROUP BY time
ORDER BY trafictime DESC;")
```

```{r}
   time count_star()
1     0       169068
2     1       168539
3     2       168711
4     3       169050
5     4       168422
6     5       168283
7     6       169015
8     7       169241
9     8       168205
10    9       168283
11   10       168750
12   11       168684
13   12       168892
14   13       169617
15   14       169028
16   15       168355
17   16      4490576
18   17      4483578
19   18      4489386
20   19      4487345
21   20      4482712
22   21      4487109
23   22      4489703
24   23      4488093
```

```{r}
dbGetQuery(con,"
SELECT src
FROM (
    SELECT src, SUM(bytes) AS total_bytes
    FROM (
        SELECT *,
            EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS time
        FROM tbl
    ) sub
    WHERE src <> '13.37.84.125'
        AND (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
        AND (dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%')
        AND time BETWEEN 1 AND 15
    GROUP BY src
) grp
ORDER BY total_bytes DESC
LIMIT 1;")
```

```{r}
          src     total
1 12.55.77.96 289566918
```

c) Найдите утечку данных из вашей сети 3

```{r}
dbExecute(con,"CREATE TEMPORARY TABLE task31 AS
SELECT src, bytes, port
FROM tbl
WHERE src <> '13.37.84.125'
    AND src <> '12.55.77.96'
    AND (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
    AND (dst NOT LIKE '12.%' OR dst NOT LIKE '13.%' OR dst NOT LIKE '14.%');")
```

```{r}
   port (max(bytes) - avg(bytes))
1    37               174312.0109
2    39               163385.8261
3   105               162681.0147
4    40               160069.5736
5    75               159551.3910
6    89               159019.7193
7   102               158498.2456
8    81               157301.8304
9   119               155052.6375
10   74               154721.2971
11  118               151821.6760
12   29               150729.4813
13  114               149597.0905
14   52               149457.2073
15   56               148400.3599
16   55               147062.5682
17   92               146951.4479
18   57               145375.1491
19   44               144456.6315
20   65               143321.7749
21  115               140656.5430
22   34                  840.5010
23   50                  785.4990
24   72                  754.4533
25   82                  749.4924
26   68                  745.6607
27   27                  741.4552
28   96                  739.3726
29   23                  737.3030
30   22                  731.6059
31  121                  731.4658
32   80                  723.7275
33   77                  720.5506
34   61                  710.6674
35   26                  701.5026
36   94                  700.8950
37   79                  693.4624
38  124                  212.4346
39   25                    0.0000
40   42                    0.0000
41   51                    0.0000
42   90                    0.0000
43  106                    0.0000
44  112                    0.0000
45  117                    0.0000
46  123                    0.0000
```

```{r}
dbGetQuery(con,"SELECT port, AVG(bytes) AS mean_bytes, MAX(bytes) AS max_bytes, SUM(bytes) AS sum_bytes, MAX(bytes) - AVG(bytes) AS Raz
FROM task31
GROUP BY port
HAVING MAX(bytes) - AVG(bytes) != 0
ORDER BY Raz DESC
LIMIT 1;")
```

```{r}
dbGetQuery(con,"SELECT src
FROM (
    SELECT src, AVG(bytes) AS mean_bytes
    FROM task31
    WHERE port = 37
    GROUP BY src
) AS task32
ORDER BY mean_bytes DESC
LIMIT 1;")
```
```{r}
          src avg(bytes)
1 13.46.35.35    37748.2
```
### Оценка результата

Был скачан и проанализирован пакет данных tm_data, были выполнены три задания.

### Вывод 

Мы познакомились с DuckDB и применили его для анализа сетевого трафика.