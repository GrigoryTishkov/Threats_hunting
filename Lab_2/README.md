# Основы обработки данных с помощью R и Dplyr

gtishckov@yandex.ru

## Цель работы

Развить практические навыки использования функций обработки данных пакета dplyr – функции select(), filter(), mutate(), arrange(), group_by()

## Исходные данные

1.  Программное обеспечение Windows 11

2.  Rstudio Desktop

3.  Интерпретатор языка R 4.4.1

4.  Программный пакет dplyr

## План

1.  Установить программный пакет dplyr

2.  Проанализировать набор данных

3.  Ответить на вопросы

## Шаги

1.  Для начала прохождения курса необходимо установить программный пакет dplyr.

```{r}         
install.packages("dplyr")
```

2.  Загружаем библиотеку. и выполняем задания


```{r}
library(dplyr)
```

a\) Сколько строк в датафрейме?

```{r}
starwars %>% nrow()

```

```{r}
[1] 87
```
b\) Сколько столбцов в датафрейме?

```{r}
starwars %>% ncol()
```

```{r}
[1] 14
```
c\) Как просмотреть примерный вид датафрейма?

```{r}
starwars %>% glimpse()
```

```{r}
Rows: 87
Columns: 14
$ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
$ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
$ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
$ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
$ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
$ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
$ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
$ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
$ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
$ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
$ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
$ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
$ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
$ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…
```

d\) Сколько уникальных рас персонажей (species) представлено в данных?

```{r}
starwars %>% distinct(species)

```

```{r}
[1] 38
```
e\) Найти самого высокого персонажа.

```{r}

max_values <- starwars 
```

f\) Найти всех персонажей ниже 170

```{r}
starwars %>% filter(!is.na(height) & height < 170) %>% select(name,height)
```

```{r}
# A tibble: 22 × 2
   name                  height
   <chr>                  <int>
 1 C-3PO                    167
 2 R2-D2                     96
 3 Leia Organa              150
 4 Beru Whitesun Lars       165
 5 R5-D4                     97
 6 Yoda                      66
 7 Mon Mothma               150
 8 Wicket Systri Warrick     88
 9 Nien Nunb                160
10 Watto                    137
# ℹ 12 more rows
```

g\) Подсчитать ИМТ (индекс массы тела) для всех персонажей. ИМТ подсчитать по формуле

```{r}
starwars %>% filter(!is.na(mass) & !is.na(height)) %>% mutate(bmi = mass / (height/100)^2) %>% select(name,bmi)

```

```{r}
# A tibble: 87 × 2
   name                   imt
   <chr>                <dbl>
 1 Luke Skywalker     0.00260
 2 C-3PO              0.00269
 3 R2-D2              0.00347
 4 Darth Vader        0.00333
 5 Leia Organa        0.00218
 6 Owen Lars          0.00379
 7 Beru Whitesun Lars 0.00275
 8 R5-D4              0.00340
 9 Biggs Darklighter  0.00251
10 Obi-Wan Kenobi     0.00232
# ℹ 77 more rows
```

h\) Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по отношению массы (mass) к росту (height) персонажей.

```{r}
    starwars %>% filter(!is.na(mass) & !is.na(height)) %>% mutate(stretch = mass / height) %>% arrange(desc(stretch)) %>% slice(1:10) %>% select(name,stretch)
```

```{r}
# A tibble: 10 × 2
   name                      v
   <chr>                 <dbl>
 1 Jabba Desilijic Tiure 7.76 
 2 Grievous              0.736
 3 IG-88                 0.7  
 4 Owen Lars             0.674
 5 Darth Vader           0.673
 6 Jek Tono Porkins      0.611
 7 Bossk                 0.595
 8 Tarfful               0.581
 9 Dexter Jettster       0.515
10 Chewbacca             0.491
```
i\) Найти средний возраст персонажей каждой расы вселенной Звездных войн.

```{r}
    starwars %>% filter(!is.na(species) & !is.na(birth_year)) %>% group_by(species) %>% summarise(average_age = mean(birth_year, na.rm = TRUE))
```

```{r}
# A tibble: 15 × 2
   species        `mean(birth_year)`
   <chr>                       <dbl>
 1 Cerean                       92  
 2 Droid                        53.3
 3 Ewok                          8  
 4 Gungan                       52  
 5 Human                        53.7
 6 Hutt                        600  
 7 Kel Dor                      22  
 8 Mirialan                     49  
 9 Mon Calamari                 41  
10 Rodian                       44  
11 Trandoshan                   53  
12 Twi'lek                      48  
13 Wookiee                     200  
14 Yoda's species              896  
15 Zabrak                       54  
```

j\)Найти самый распространенный цвет глаз персонажей вселенной Звездных войн.

```{r}
    starwars %>% filter(!is.na(eye_color)) %>% group_by(eye_color) %>% summarise(count = n()) %>% arrange(desc(count)) %>% slice(1)
```

```{r}
# A tibble: 1 × 2
  eye_color   sum
  <chr>     <int>
1 brown        21
```

k\) Подсчитать среднюю длину имени в каждой расе вселенной Звездных войн.

```{r}
    starwars %>% filter(!is.na(species) & !is.na(name)) %>% mutate(name_length = nchar(name)) %>% group_by(species) %>% summarise(len = mean(name_length, na.rm = TRUE))
```

```{r}
# A tibble: 37 × 2
   species   `mean(nlen)`
   <chr>            <dbl>
 1 Aleena           12   
 2 Besalisk         15   
 3 Cerean           12   
 4 Chagrian         10   
 5 Clawdite         10   
 6 Droid             4.83
 7 Dug               7   
 8 Ewok             21   
 9 Geonosian        17   
10 Gungan           11.7 
# ℹ 27 more rows
```
## Оценка результата

В результате работы была скачана библиотека dplyr и были выполнены задания с использованием набора данных starwars.

## Вывод

Были изучены функции библиотеки dplyr. Были выполнены поставленные практикой задачи.
