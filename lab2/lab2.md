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
<img src = './pics/3.png' />

<img src = './pics/4.png' />

<img src = './pics/5.png' />

> Задание 2
1. Что означают символы `@@`

Оператор сопоставления @@ возвращает true, если	tsvector соответствует tsquery.

2. Почему второй запрос возвращает `true`, а третий не возвращает

Третий запрос вернул `false` т.к. не нашёл слово `foxhound`.

3. Выполните запрос
```sql
SELECT to_tsvector('Russian', 'Съешь ещё этих мягких французских булок, да выпей чаю.')
    @@ to_tsquery('Russian','булка');
```

<img src = './pics/6.png' />

Почему слово булка не находится?

4. Используйте функцию `select ts_lexize('russian_stem', 'булок');` для того чтобы понять почему.

<img src = './pics/7.png' />

<img src = './pics/8.png' />

`булка` и `булок` разные лексемы, поэтому запрос из предыдущего пункта вернул false.

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

<img src = './pics/9.png' />

<img src = './pics/10.png' />

Почему первый запрос возвращает `true`, а второй не возвращает?

<img src = './pics/11.png' />

<img src = './pics/12.png' />

<img src = './pics/13.png' />

Опять-таки, `пирожки` и `пирожок` имеют различные лексемы.

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

<img src = './pics/14.png' />

<img src = './pics/15.png' />

<img src = './pics/16.png' />

<img src = './pics/17.png' />

> Задание 3
1. Приведите аналогичные запросы для любого предложения на русском

```sql
--И
SELECT to_tsvector('Идет медведь по лесу, видит — машина горит. Сел в нее и сгорел.') 
	@@ to_tsquery('Russian', 'медведь & лес');

--ИЛИ
SELECT to_tsvector('Идет медведь по лесу, видит — машина горит. Сел в нее и сгорел.') 
	@@ to_tsquery('Russian', 'медведь | волк');

--отрицание
SELECT to_tsvector('Идет медведь по лесу, видит — машина горит. Сел в нее и сгорел.') 
	@@ to_tsquery('Russian', '!волк');

--группировка
SELECT to_tsvector('Идет медведь по лесу, видит — машина горит. Сел в нее и сгорел.') 
	@@ to_tsquery('Russian', 'медведь & лес | (машина & !сгорел)');
```

<img src = './pics/18.png' />

<img src = './pics/19.png' />

<img src = './pics/20.png' />

<img src = './pics/21.png' />

Все эти запросы возвращают `true`.

2. Почему для английского языка не нужно указывать язык в первом аргументе и какой анализатор используется если никакой не указан?

Первый аргумент в запросе обозначает язык, его значение по умолчанию 'english'. При использовании другого языка указывать его нужно обязательно.

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
