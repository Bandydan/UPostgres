# PostgreSQL, CRUD даних

Коли база даних створена та таблиці в ній створені, коли все це зроблено правильним користувачем з правильними правами та рівнем доступу, саме час заповнити таблиці даними. У роботі з даними є лише 4 дії: створення, отримання, зміна та видалення даних. Для позначення цих процесів використовується абревіатура **CRUD: Create, Read, Update, Delete**.

Для експериментів із даними нам знадобиться таблиця. Наприклад, таблиця слів зі словника та таблиця словників. Про всяк випадок - команди створення цих таблиць та їх опис:

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

## CRUD даних - Create, додавання даних (INSERT)

Для додавання даних до таблиці використовується оператор **`INSERT INTO`**. Оператор **`INSERT INTO`** буває декількох видів, і ми розглянемо основні:

### Simple insert

Найпростіший варіант вставки даних у таблицю виглядає так:

```sql
insert into vocabulary (name) values ('verbs');
INSERT 0 1
```

Перевіримо наші дані найпростішим **`select`**:

```sql
select * from vocabulary;
 id |          name           | info
----+-------------------------+------
  1 | verbs                   |
(1 row)
```

У цьому прикладі ми вставляємо запис в таблицю vocabulary, вказуючи конкретне значення для кожного стовпця.

### Multiple insert

```sql
insert into vocabulary (name) values ('IT'), ('Silicon Valley season 1');
INSERT 0 2
```

Перевіримо:

```sql
select * from vocabulary;

 id |          name           | info
----+-------------------------+------
  1 | verbs                   |
  2 | IT                      |
  3 | Silicon Valley season 1 |
(3 rows)
```
Код вставляє дані в наступних дужках у відповідні стовпці, зазначені в перших дужках. Таким чином, спочатку необхідно перерахувати стовпці (поля), в які планується вносити дані, а потім через кому перерахувати обгорнуті дужками набори даних для цих полів. Такий запит дозволяє додавати кілька записів разом.

### Insert from select

Можлива також вставка даних із результату запиту:

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
Тут наведено досить простий приклад, але, по суті, якщо ви збудуєте запит даних **`SELECT`** таким чином, щоб кількість стовпців в результаті відповідала необхідному, вказаному в зовнішньому запиті **`INSERT`**, ви можете вбудувати запит досить серйозного рівня складності.

## CRUD даних - Read, вибірка даних (SELECT)

Запит **`SELECT`** використовується для отримання даних, і жодним чином їх не змінює. Структура його досить складна, і спробуємо розібрати її поступово і поетапно.

Мінімальний можливий запит має такий вигляд:

```sql
select 1;
 ?column? 
----------
        1
(1 row)

```
Єдиним обов'язковим ключовим словом у запиті **`select`** є слово `select` - вибрати.

Після цього слова слід писати:

- функції та оператори postgreSQL ([наприклад, функції роботи з датою та часом](https://www.tutorialspoint.com/postgresql/postgresql_date_time.htm))

```sql
select CURRENT_TIME;

    current_time
--------------------
 16:54:45.183026+03
(1 row)
```

- рядки та числа
- поля таблиць, тимчасових таблиць та уявлень, які ми збираємося вибирати
- похідні від цих полів
- Вирази

Найчастіше **`SELECT`** використовується для роботи з даними з таблиць, і для цього потрібно вказати ці таблиці. Таблиці, якими здійснюється вибірка, перераховуються після ключового слова **`FROM`**:

```sql
SELECT * FROM books;
```

Наведений вище запит вибирає всі поля (за це відповідає зірочка) із таблиці books.

Максимально докладна схема запиту `select` виглядає так:

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
    
ORDER BY
    <field1> ASC
    <field3> DESC
GROUP BY
    <field 1>
HAVING
    <cond with aggr function>
LIMIT
    N,M
```


Ниже приведено текстовое описание основных элементов структуры запроса **`SELECT`**, более подробное описание с примерами будет приведено позже.

1. Після ключового слова **`SELECT`** йде перелік полів таблиць, функцій, що обчислюються з цих полів, констант, незалежних від записів функцій. Для вказівки всіх полів використовується зірочка. Цей пункт є єдиним обов'язковим пунктом у запиті **`SELECT`**, інші опціональні.
2. Далі, після ключового слова **`FROM`** слідує перелік таблиць, уявлень та тимчасових таблиць, звідки ведеться вибірка. Таблиці можуть бути перераховані, а можуть бути приєднані до інших таблиць за описаними окремо правил, тобто. за допомогою **`JOIN`**.
3. Далі слідує умова **`WHERE`**, що пропускає лише ті записи, які задовольняють перерахованим у **`WHERE`** умовам. Всі записи, що не пройшли перевірку, відфільтровуються і не демонструються.
4. Після фільтру **`WHERE`** може слідувати групування записів.
5. Угруповання виконується за допомогою ключових слів **`GROUP BY`**. Суть угруповання в тому, що записи можуть об'єднуватися за ознакою або декількома ознаками в один запис, який несе у собі якусь загальну для всіх записів групи інформацію або результат обробки інформації по всій групі. Конструкція **`GROUP BY`** може включати ключове слово **`HAVING`**, що дозволяє фільтрувати результати угруповання.
6. Важливо відзначити, що угруповання дозволяє використовувати аггрегатні функції як в **`HAVING`**, так і після **`SELECT`**.
7. Після групування може відбутися сортування записів за допомогою ключових слів **ORDER BY**. При групуванні вказується поле або перелік полів, за яким необхідно відсортувати, також можна вказати напрямок сортування. За умовчанням здійснюється сортування за зростанням. Сортування за спаданням здійснюється за допомогою ключового слова **`descending`** або **`desc`**.
8. Наприкінці запиту можливе додавання обмежень на кількість записів. Слово **`LIMIT`** та цифрою після вказує, скільки записів ви хочете бачити в результаті. Якщо після слова **`LIMIT`** додати дві цифри через кому, ви побачите друге число - кількість записів після пропущеного першого числа-кількості записів, тобто. **`LIMIT 20, 5`** пропустить 20 записів та покаже вам 5 наступних.
9. PostgreSQL надає альтернативу LIMIT - **`FETCH`**. На думку розробників, **`FETCH`** краще відповідає стандартам SQL мови.

## Практика з SELECT та INSERT

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
Умова **`distinct`** відкидає дублікати в результаті запиту, залишаючи лише унікальні записи.

### WHERE

Додамо кілька слів до нашої таблиці слів:

```sql
insert into word (word, vocabulary_id) values('have', 1), ('IP', 2), ('Kanban', 3);
INSERT 0 3

insert into word (word, vocabulary_id) values('have', 7), ('TCP/IP', 2), ('Function', 3);
INSERT 0 3
```

Тепер виберемо всі слова та кілька разів відфільтруємо їх за допомогою **`where`**:

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

Трохи більше фільтрації та перерахування полів:

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

Групувати можна дані, які повторюються у групах і не суперечитимуть умовам угруповання. При групуванні можна логічно використовувати аггрегатні функції. Агрегатні функції - особливі функції SQL, які застосовуються до всіх записів в результаті вибірки, або до груп. Count – одна з таких функцій.

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

Ключове слово HAVING додається лише після GROUP BY для додаткової фільтрації результатів запиту. WHERE фільтрує їх до угруповання, HAVING фільтрує згруповані.

```sql
select count(*), vocabulary_id from word where vocabulary_id < 4
and id < 6 group by vocabulary_id having count(*) > 1;

 count | vocabulary_id
-------+---------------
     2 |             2
(1 row)
```

У цьому запиті відображаються лише ті групи слів, які зустрілися більше одного разу.

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
## Корисні посилання

**Типи даних:**


[типи даних](https://www.tutorialspoint.com/postgresql/postgresql_data_types.htm)

[identity column](http://www.postgresqltutorial.com/postgresql-identity-column/)

[serial column](http://www.postgresqltutorial.com/postgresql-serial/)
**Прочее:**


[Метакоманди типу `\q`:](https://www.postgresql.org/docs/current/app-psql.html#APP-PSQL-META-COMMANDS)


[Агрегатні функції](https://metanit.com/sql/postgresql/4.5.php)


[FETCH](http://www.postgresqltutorial.com/postgresql-fetch/)

[Домашка](hw02.md)

[Наступний урок](psql3.md)
