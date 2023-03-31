# PostgreSQL, CRUD данных

Когда база данных создана и таблицы в ней созданы, когда все это сделано правильным пользователем с правильными правами и уровнем доступа, самое время заполнить таблицы данными. В работе с данными есть всего 4 действия: создание, получение, изменение и удаление данных. Для обозначения этих действий используется аббревиатура **CRUD: Create, Read, Update, Delete**.

Для экспериментов с данными нам понадобится таблица. К примеру, таблица слов из словаря и таблица словарей. На всякий случай - команды создания этих таблиц и описание их:

```sql
create table word (id serial, word varchar(255), vocabulary_id integer);
CREATE TABLE

create table vocabulary (id serial, name varchar(255), info text);
CREATE TABLE

\d vocabulary
                                    Table "public.vocabulary"
 Column |          Type          | Collation | Nullable |                Default
--------+------------------------+-----------+----------+----------------------------------------
 id     | integer                |           | not null | nextval('vocabulary_id_seq'::regclass)
 name   | character varying(255) |           |          |
 info   | text                   |           |          |

\d word
                                       Table "public.word"
    Column     |          Type          | Collation | Nullable |             Default
---------------+------------------------+-----------+----------+----------------------------------
 id            | integer                |           | not null | nextval('word_id_seq'::regclass)
 word          | character varying(255) |           |          |
 vocabulary_id | integer                |           |          |

```

## CRUD данных - Create, добавление данных (INSERT)

Для добавления данных в таблицу используется оператор **`INSERT INTO`**. Оператор **`INSERT INTO`** бывает нескольких видов, и мы рассмотрим основные:

### Simple insert

Простейший вариант вставки данных в таблицу выглядит следующим образом:

```sql
insert into vocabulary (name) values ('verbs');
INSERT 0 1
```

Проверим наши данные простейшим **`select`**:

```sql
select * from vocabulary;
 id |          name           | info
----+-------------------------+------
  1 | verbs                   |
(1 row)
```

В данном примере мы вставляем запись в таблицу vocabulary, указывая конкретное значение для каждого столбца.

### Multiple insert

```sql
insert into vocabulary (name) values ('IT'), ('Silicon Valley season 1');
INSERT 0 2
```

Проверим:

```sql
select * from vocabulary;

 id |          name           | info
----+-------------------------+------
  1 | verbs                   |
  2 | IT                      |
  3 | Silicon Valley season 1 |
(3 rows)
```
Код выше вставляет данные в последующих скобках в соответствующие столбцы, указанные в первых скобках. Таким образом, вначале необходимо перечислить столбцы (поля), в которые планируется вносить данные, а затем через запятую перечислить обернутые скобочками наборы данных для этих полей. Такой запрос позволяет добавлять несколько записей разом.

### Insert from select

Возможна так же вставка данных из результата запроса:

```sql
insert into vocabulary select * from vocabulary;
INSERT 0 3

select * from vocabulary;

 id |          name           | info
----+-------------------------+------
  1 | verbs                   |
  2 | IT                      |
  3 | Silicon Valley season 1 |
  1 | verbs                   |
  2 | IT                      |
  3 | Silicon Valley season 1 |
(6 rows)
```
Здесь приведен довольно простой пример, но, по сути дела, если вы построите запрос данных **`SELECT`** таким образом, чтобы количество столбцов в результате соответствовало необходимому, указанному во внешнем запросе **`INSERT`**, вы можете встроить запрос довольно серьезного уровня сложности.

## CRUD данных - Read, выборка данных (SELECT)

Запрос **`SELECT`** используется для получения данных, и никоим образом их не изменяет. Структура его довольно сложна, и мы попробуем разобрать ее постепенно и поэтапно.

Минимальный возможный запрос выглядит так:

```sql
select 1;
 ?column? 
----------
        1
(1 row)

```
Единственным обязательным ключевым словом в запросе **`select`** является слово `select` - выбрать. 

После этого слова следует писать:

- функции и операторы postgreSQL ([например, функции работы с датой и временем](https://www.tutorialspoint.com/postgresql/postgresql_date_time.htm))

```sql
select CURRENT_TIME;

    current_time
--------------------
 16:54:45.183026+03
(1 row)
```

- строки и числа
- поля таблиц, временных таблиц и представлений, которые мы собираемся выбирать
- производные от этих полей
- выражения

Чаще всего **`SELECT`** используется для работы с данными из таблиц, и для этого нужно указать эти самые таблицы. Таблицы, по которым осуществляется выборка, перечисляются после ключевого слова **`FROM`**:

```sql
SELECT * FROM books;
```

Приведенный выше запрос выбирает все поля (за это отвечает звездочка) из таблицы books.

Максимально подробная схема запроса `select` выглядит так:

```sql
SELECT
    <field1>,
    <field2>,
    <field3>
    ...
FROM
    <table1>,
    <table2>,
    <joins>,
    <views>,
    <temp_table>
    ...
WHERE
    <cond>
GROUP BY
    <field 1>    
ORDER BY
    <field1> ASC
    <field3> DESC
HAVING
    <cond with aggr function>
LIMIT
    N,M
```


Ниже приведено текстовое описание основных элементов структуры запроса **`SELECT`**, более подробное описание с примерами будет приведено позже.

1. После ключевого слова **`SELECT`** идет перечень полей таблиц, функций, вычисляемых из этих полей, констант, независимых от записей функций. Для указания всех полей исопльзуется звездочка. Этот пункт является единственным обязательным пунктом в запросе **`SELECT`**, остальные опциональны.
2. Далее, после ключевого слова **`FROM`** следует перечень таблиц, представлений и временных таблиц, откуда ведется выборка. Таблицы могут быть просто перечислены, а могут быть присоединены к другим таблицам по описанным отдельно правилам, т.е. при помощи **`JOIN`**.
3. Далее следует условие **`WHERE`**, пропускающее только те записи, которые удовлетворяют перечисленным в **`WHERE`** условиям. Все не прошедшие проверку записи отфильтровываются и не демонстрируются.
4. После фильтра **`WHERE`** может следовать группировка записей. 
5. Группировка выполняется при помощи ключевых слов **`GROUP BY`**. Суть группировки в том, что записи могут объединяться по признаку или нескольким признакам в одну запись, которая несет в себе некую общую для всех записей группы информацию или результат обработки информации по всей группе. Конструкция **`GROUP BY`** может включать в себя ключевое слово **`HAVING`**, позволяющее фильтровать результаты группировки.
6. Важно отметить, что группировка позволяет использовать аггрегатные функции как в **`HAVING`**, так и после **`SELECT`**. 
5. После группировки может иметь место сортировка записей при помощи ключевых слов **`ORDER BY`**. При группировке указывается поле или перечень полей, по которому необходимо отсортировать, также можно указать направление сортировки. По умолчанию осуществляется сортировка по возрастанию. Сортировка по убыванию делается при помощи ключевого слова **`descending`** или **`desc`**.
6. В конце запроса возможно добавление ограничений на количество записей. Слово **`LIMIT`** и цифрой после указывает, сколько записей вы хотите видет в результате. Если после слова **`LIMIT`** добавить две цифры через запятую, вы увидите второе число - количество записей после пропущенного первого числа-количества записей, т.е. **`LIMIT 20, 5`** пропустит 20 записей и покажет вам 5 следующих.
7. PostgreSQL предоставляет альтернативу LIMIT - **`FETCH`**. ПО мнению разработчиков, **`FETCH`** лучше соответствует стандартам SQL языка.

## Практика по SELECT и INSERT

### DISTINCT

```sql
select * from vocabulary;
 id |          name           | info
----+-------------------------+------
  1 | verbs                   |
  2 | IT                      |
  3 | Silicon Valley season 1 |
  1 | verbs                   |
  2 | IT                      |
  3 | Silicon Valley season 1 |
(6 rows)

select distinct * from vocabulary;
 id |          name           | info
----+-------------------------+------
  2 | IT                      |
  1 | verbs                   |
  3 | Silicon Valley season 1 |
(3 rows)
```
Условие **`distinct`** отбрасывает дубикаты в результате запроса, оставляя только уникальные записи.

### WHERE

Добавим несколько слов в нашу таблицу слов:

```sql
insert into word (word, vocabulary_id) values('have', 1), ('IP', 2), ('Kanban', 3);
INSERT 0 3

insert into word (word, vocabulary_id) values('have', 7), ('TCP/IP', 2), ('Function', 3);
INSERT 0 3
```

Теперь выберем все слова и несколько раз отфильтруем их при помощи **`where`**:

```sql
select * from word;
 id |   word   | vocabulary_id
----+----------+---------------
  1 | have     |             1
  2 | IP       |             2
  3 | Kanban   |             3
  4 | have     |             7
  5 | TCP/IP   |             2
  6 | Function |             3
(6 rows)

select word from word where id > 5;
   word
----------
 Function
(1 row)

select word from word where id > 3;
   word
----------
 have
 TCP/IP
 Function
(3 rows)
```

Чуть больше фильтрации и перечисление полей:

```sql
select word from word where vocabulary_id < 4 and id < 6;
  word
--------
 have
 IP
 Kanban
 TCP/IP
(4 rows)

select id, word, vocabulary_id from word where vocabulary_id < 4 and id < 6;
 id |  word  | vocabulary_id
----+--------+---------------
  1 | have   |             1
  2 | IP     |             2
  3 | Kanban |             3
  5 | TCP/IP |             2
(4 rows)
```

### GROUP BY

Группировать можно данные, которые повторяются в группах и не будут противоречить условиям группировки. При группировке можно и логично использовать аггрегатные функции. Аггрегатные функции - особые функции SQL, которые применяются либо ко всем записям в результате выборки, либо к группам. Count - одна из таких функций. 


```sql
select vocabulary_id from word where vocabulary_id < 4 
and id < 6 group by vocabulary_id;
 vocabulary_id
---------------
             3
             1
             2
(3 rows)

select count(*), vocabulary_id from word where vocabulary_id < 4
and id < 6 group by vocabulary_id;
 count | vocabulary_id
-------+---------------
     1 |             3
     1 |             1
     2 |             2
(3 rows)
```

### GROUP BY HAVING

Ключевое слово HAVING добавляется только после GROUP BY с целью дополнительной фильтрации результатов запроса. WHERE фильтрует их до группировки, HAVING фильтрует сгруппированные.

```sql
select count(*), vocabulary_id from word where vocabulary_id < 4
and id < 6 group by vocabulary_id having count(*) > 1;

 count | vocabulary_id
-------+---------------
     2 |             2
(1 row)
```

В данном запросе показываются только те группы слов, которые встретились более одного раза.

### ORDER BY

```sql
select * from word order by vocabulary_id;
 id |   word   | vocabulary_id
----+----------+---------------
  1 | have     |             1
  2 | IP       |             2
  5 | TCP/IP   |             2
  3 | Kanban   |             3
  6 | Function |             3
  4 | have     |             7
(6 rows)

select * from word order by 3;
 id |   word   | vocabulary_id
----+----------+---------------
  1 | have     |             1
  2 | IP       |             2
  5 | TCP/IP   |             2
  3 | Kanban   |             3
  6 | Function |             3
  4 | have     |             7
(6 rows)

select * from word order by 3, 2;
 id |   word   | vocabulary_id
----+----------+---------------
  1 | have     |             1
  2 | IP       |             2
  5 | TCP/IP   |             2
  6 | Function |             3
  3 | Kanban   |             3
  4 | have     |             7
(6 rows)
```

### LIMIT и OFFSET

```sql
select * from word order by 3, 2 limit 3;
 id |  word  | vocabulary_id
----+--------+---------------
  1 | have   |             1
  2 | IP     |             2
  5 | TCP/IP |             2
(3 rows)

select * from word order by 3, 2 limit 3 offset 3;
 id |   word   | vocabulary_id
----+----------+---------------
  6 | Function |             3
  3 | Kanban   |             3
  4 | have     |             7
(3 rows)
```
## Полезные ссылки

**Типы данных:**


[типы данных](https://www.tutorialspoint.com/postgresql/postgresql_data_types.htm)

[identity column](http://www.postgresqltutorial.com/postgresql-identity-column/)

[serial column](http://www.postgresqltutorial.com/postgresql-serial/)
**Прочее:**


[Метакоманды типа `\q`:](https://www.postgresql.org/docs/current/app-psql.html#APP-PSQL-META-COMMANDS)


[Аггрегатные функции](https://metanit.com/sql/postgresql/4.5.php)


[FETCH](http://www.postgresqltutorial.com/postgresql-fetch/)

[Домашка](hw02.md)

[Следующий урок](psql3.md)
