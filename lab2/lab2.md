# Использование FTS в PostgreSQL
#### 1. Тип tsvector
Выполните запрос
```sql
SELECT to_tsvector('The quick brown fox jumped over the lazy dog.');
```
В ответ будет возвращён список [лексем](https://en.wikipedia.org/wiki/Lexeme)
```sql
                to_tsvector
'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2
```
<img src = './pics/1.png' />

> Задание 1
1. Изучите документацию к функции `to_tsvector`
2. Вызовите эту функцию для следующей строки: `Съешь ещё этих мягких французских булок, да выпей чаю`
<img src = './pics/2.png' />
3. Почему в векторе нет слова `да`?

Слово `да` в данном примере распозналось как стоп-слово. Стоп-словами называются слова, которые встречаются очень часто, практически в каждом документе, и поэтому не имеют различительной ценности. Таким образом, при полнотекстовом поиске их можно игнорировать. Например, в каждом английском тексте содержатся артикли a и the, так что хранить их в индексе бессмысленно. Однако стоп-слова влияют на позиции лексем в значении tsvector, от чего, в свою очередь, зависит ранжирование.

#### 2. Тип tsquery
Выполните по очереди запросы
```sql
--№1
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('fox');
--№2
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('foxes');
--№3 
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('foxhound');
```

> Задание 2
1. Что означают символы `@@`
2. Почему второй запрос возвращает `true`, а третий не возвращает
3. Выполните запрос
```sql
SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских булок, да выпей чаю.')
    @@ to_tsquery('Russian','булка');
```
Почему слово булка не находится?
4. Используйте функцию `select ts_lexize('russian_stem', 'булок');` для того чтобы понять почему.
5. Замените в предложении слово `булок`, на слово `пирожков`
Выполните запросы
```sql
--№1
SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских пирожков, да выпей чаю.')
    @@ to_tsquery('Russian','пирожки');
--№2
SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских пирожков, да выпей чаю.')
    @@ to_tsquery('Russian','пирожок');
```
Почему первый запрос возвращает `true`, а второй не возвращает?

#### 3. Операторы
Выполните запрос
```sql
--И
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('fox & dog');

--ИЛИ
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('fox | rat');

--отрицание
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('!clown');

--группировка
SELECT to_tsvector('The quick brown fox jumped over the lazy dog')  
    @@ to_tsquery('fox & (dog | rat) & !mice');
```
> Задание 3
1. Приведите аналогичные запросы для любого предложения на русском
2. Почему для английского языка не нужно указывать язык в первом аргументе и какой анализатор используется если никакой не указан?


#### 4. Поиск фраз
Изучите документацию по [операторам](https://www.postgresql.org/docs/current/functions-textsearch.html) FTS
Выполните запрос
```sql

SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских булок, да выпей чаю.')
    @@ to_tsquery('Russian','мягких<2>булок');
```
> Задание 4
1. Что означает число 2 в операторе `<->`
2. Модифицируйте запрос так, чтобы можно было найти фразу `съешь ещё`
3. Для чего нужно использовать функцию `phraseto_tsquery`

#### 5. Утилиты
1. Приведите примеры использования функций `ts_debug` и  `ts_headline`
