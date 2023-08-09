# JOINS, відносини, практика запитів

## Відносини між таблицями

Як ми вже обговорювали, PostgreSQL відноситься до реляційних баз даних. Реляційні БД характеризуються наявністю таблиць та відносин.

Існує 3 типи відносин:

- "**Один до одного**" - коли один запис у першій таблиці відповідає одному запису у другій таблиці. Зустрічається такий зв'язок досить рідко. Така зв'язок чи надмірна, тобто. може сенс просто об'єднати дані в одну таблицю, або це результат модернізації архітектури, і таке рішення кимось прийнято обґрунтовано.

![](https://zametkinapolyah.ru/wp-content/uploads/2016/05/%D0%BE%D0%B4%D0%B8%D0%BD-%D0%BA%D0%BE-%D0%BE%D0%B4%D0%BD%D0%BE%D0%BC%D1%83.png)

- "**Один до багатьох**" - коли одного запису в першій таблиці відповідає кілька записів в іншій таблиці. Наприклад, клієнт магазину може мати кілька номерів. Один клієнт – один запис у таблиці клієнта, його номери телефонів – записи у таблиці телефонів. Один клієнт відноситься до багатьох номерів телефонів, але зворотний зв'язок - номер телефону до клієнта - багато хто до одного.

![](https://zametkinapolyah.ru/wp-content/uploads/2016/05/%D0%9E%D0%B4%D0%B8%D0%BD-%D0%BA%D0%BE-%D0%BC%D0%BD%D0%BE%D0%B3%D0%B8%D0%BC.png)

- "**Багато до багатьох**" - коли одному запису в першій таблиці відповідає кілька записів у другій таблиці, але одному запису другої таблиці може відповідати кілька записів першої таблиці. Класичний приклад - книги та автори, адже автор може мати багато книг, і кожна книга може мати кілька авторів.
Реалізується з допомогою створення таблиці зв'язків, куди копіюються ключі записів обох таблиць.

![](https://zametkinapolyah.ru/wp-content/uploads/2016/05/%D0%9C%D0%BD%D0%BE%D0%B3%D0%B8%D0%B5-%D0%BA%D0%BE-%D0%BC%D0%BD%D0%BE%D0%B3%D0%B8%D0%BC.png)

## Об'єднання (JOINS)


![JOINS](https://www.postgresqltutorial.com/wp-content/uploads/2018/12/PostgreSQL-Joins.png)
Запити до таблиці досить рідкісні. Найчастіше запити до баз даних пишуться з метою отримати інформацію з кількох таблиць, інформація з яких поєднується за певними умовами.

Створимо таблиці авторів, книг, жанрів та таблицю зв'язків для авторів та книг (багато хто до багатьох):

```sql

CREATE TABLE authors (
	id SERIAL PRIMARY KEY,
	name VARCHAR(200) NOT NULL DEFAULT 'lore',
	year DATE NOT NULL DEFAULT '1970-01-01'
);

CREATE TABLE

CREATE TABLE books (
	id SERIAL PRIMARY KEY,
	title VARCHAR(200) NOT NULL DEFAULT 'noname',
	genre_id INT NOT NULL DEFAULT 0
);

CREATE TABLE

CREATE TABLE genres (
	id SERIAL PRIMARY KEY,
	genre VARCHAR(100) NOT NULL DEFAULT 'unknown'
);

CREATE TABLE

CREATE TABLE authors_books (
	author_id INT NOT NULL DEFAULT 0,
	book_id INT NOT NULL DEFAULT 0
);

CREATE TABLE
```

Далі слід заповнити таблицю даними:

```sql
INSERT INTO genres (genre) VALUES
	('SF'),
	('novel'),
	('story'),
	('horror');

INSERT INTO books(title, genre_id) VALUES
	('The Master and Margarita', 2),
	('Faust', 0),
	('White Fang', 3),
	('Dune', 1),
	('War and Peace', 2);

INSERT INTO authors (name) VALUES
	('Frank Herbert'),
	('Mikhail Bulgakov'), 
	('Jack London'), 
	('Johann Goethe'), 
	('Robert Heinlein');

INSERT INTO authors_books (author_id, book_id) VALUES 
	(1, 4),
	(2, 1),
	(3, 3),
	(4, 2);

```

Дані готові, тепер можна вивчати об'єднання таблиць. Почнемо ми з об'єднання двох таблиць в одному запиті:

```sql
SELECT title, genre
FROM books
INNER JOIN genres ON (genres.id = books.genre_id);

       title              | genre
--------------------------+-------
 The Master and Margarita | novel
 White Fang               | story
 Dune                     | SF
 War and Peace            | novel
(4 rows)

```

**`INNER JOIN`** виводить записи з лівої таблиці (першої з двох таблиць, які він об'єднує), для яких знайдеться відповідний запис у правій (другій та останній у списку) таблиці. Якщо відповідності у правій таблиці немає, такий запис не виводиться. В даному випадку можна бачити, що запис про книгу "Faust" не виводиться. Це відбувається через те, що `genre_id` цей запис дорівнює нулю, а такого жанру в таблиці жанрів немає.
Так само в результат не потрапляє жанр horror, тому що немає жодної книги з таким жанром.

```sql
SELECT title, genre
FROM books
LEFT JOIN genres ON (genres.id = books.genre_id);

       title              | genre
--------------------------+-------
 The Master and Margarita | novel
 Faust                    |
 White Fang               | story
 Dune                     | SF
 War and Peace            | novel

(5 rows)

```
**`LEFT JOIN`** виводить **ВСІ** записи з лівої таблиці. Для тих записів, яким знаходиться відповідність у правій, він, аналогічно **INNER JOIN**, виведе відповідні дані з другої таблиці. Для тих, яким відповідності не знайшлося, він нічого не виведе в стовпцях правої таблиці, залишивши їх порожніми.

Щоб визначити, яким книгам не знайдені у відповідність жанри, можна виконати запит із підзапитом:

```sql
SELECT title
FROM books
WHERE NOT EXISTS (
	SELECT * 
	FROM genres 
	WHERE books.genre_id = genres.id
);

 title
-------
 Faust
(1 row)
```

Цей шлях вважається правильнішим, ніж класичний:

```sql
SELECT title, genre 
FROM books 
LEFT JOIN genres ON (genres.id = books.genre_id)
WHERE genre IS NULL;


 title
-------
 Faust
(1 row)

```

**`RIGHT JOIN`** надходить аналогічним чином з правою таблицею: виводить усі записи з неї, додаючи записи з лівої. Де відповідності немає, залишає в стовпцях лівої таблиці порожнечу.

```sql
SELECT title, genre
FROM books
RIGHT JOIN genres ON (genres.id = books.genre_id);

       title              | genre
--------------------------+--------
 The Master and Margarita | novel
 White Fang               | story
 Dune                     | SF
 War and Peace            | nov
                          | horror
(5 rows)
```

Якщо у вас виникне потреба побачити всі книги та всі жанри незалежно від наявності відповідних записів в іншій таблиці, але де зв'язки є – зв'язати таблиці, можна об'єднати результати двох запитів:


```sql
SELECT title, genre 
FROM books 
LEFT JOIN genres ON (genres.id = books.genre_id) 
UNION 
SELECT title, genre 
FROM books RIGHT JOIN genres ON (genres.id = books.genre_id);


       title              | genre
--------------------------+--------
 Faust                    |
                          | horror
 War and Peace            | novel
 White Fang               | story
 Dune                     | SF
 The Master and Margarita | novel
(6 rows)
```

Аналогічного результату можна досягти і спеціальним видом об'єднань - **`FULL JOIN`**:

```sql
SELECT title, genre
FROM books
FULL JOIN genres ON (genres.id = books.genre_id);


       title              | genre
--------------------------+--------
 The Master and Margarita | novel
 Faust                    |
 White Fang               | story
 Dune                     | SF
 War and Peace            | novel
                          | horror
(6 rows)
```

Якщо поля зв'язку називаються однаково, наприклад, поле id у таблиці genres називається genre_id, запит можна трохи спростити. Для демонстрації перейменуємо поле та виконаємо цей запит:

```sql
ALTER TABLE genres RENAME COLUMN id TO genre_id;

ALTER TABLE

SELECT title, genre
FROM books 
RIGHT JOIN genres USING(genre_id);


       title              | genre
--------------------------+--------
 The Master and Margarita | novel
 White Fang               | story
 Dune                     | SF
 War and Peace            | novel
                          | horror
(5 rows)


```

Результат буде той же, що і у звичайного ** RIGHT JOIN запиту, але сам запит коротше.

## Практика

- Об'єднати авторів із книгами за допомогою таблиці зв'язків
- Побудувати фамільне дерево по чоловічій лінії

## Корисні посилання

[ACID](https://habr.com/ru/post/555920/)

[Indexes](https://tproger.ru/articles/indeksy-v-postgresql/)

[Домашка](hw03.md)

[Наступний урок](psql4.md)
