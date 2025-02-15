# Домашнее задание к занятию «Индексы»
*Асадбеков Асадбек*

## Задание 1

**Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.**


```bash
SELECT 
    ROUND(SUM(index_length) / SUM(data_length + index_length) * 100, 2) AS index_to_table_ratio_percent
FROM 
    information_schema.TABLES
WHERE 
    table_schema = 'sakila';
```

![alt text](https://github.com/asad-bekov/hw-15/blob/main/img/1.png)

## Задание 2

**Выполните explain analyze следующего запроса:**

```bash
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

1: Анализ запроса с помощью `EXPLAIN ANALYZE`

```bash
EXPLAIN ANALYZE
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name), SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p, rental r, customer c, inventory i, film f
WHERE DATE(p.payment_date) = '2005-07-30' 
  AND p.payment_date = r.rental_date 
  AND r.customer_id = c.customer_id 
  AND i.inventory_id = r.inventory_id;
```
![alt text](https://github.com/asad-bekov/hw-15/blob/main/img/2.png)

2. Узкие места

Оператор `DISTINCT` требует сортировки и удаления дубликатов, что может быть дорогостоящим.

Оконные функции (`SUM() OVER`) требуют дополнительных вычислений и памяти.

Использование функции `DATE()` на столбце `payment_date` не позволяет использовать индекс.

Использование неявных `JOIN` (через `WHERE`) может затруднить оптимизацию запроса.

![alt text](https://github.com/asad-bekov/hw-15/blob/main/img/2.1.png)

3. Оптимизация запроса

- Использование явных `JOIN`:

```bash
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name), SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE p.payment_date >= '2005-07-30' AND p.payment_date < '2005-07-31';
```

![alt text](https://github.com/asad-bekov/hw-15/blob/main/img/2.2.png)

- Устранение `DISTINCT`:
Если возможно, заменить `DISTINCT` на группировку или убедиться, что данные не дублируются.

- Оптимизация фильтрации:
Вместо `DATE(p.payment_date) = '2005-07-30'` использовать диапазон:

- Добавление индексов:
1. Индекс на `payment_date`:

```bash
CREATE INDEX idx_payment_date ON payment(payment_date);
```
2. Индекс на `rental_date`:

```bash
CREATE INDEX idx_payment_date ON payment(payment_date);
```

![alt text](https://github.com/asad-bekov/hw-15/blob/main/img/3.png)

## Задание 3*

**Самостоятельно изучите, какие типы индексов используются в `PostgreSQL`. Перечислите те индексы, которые используются в `PostgreSQL`, а в `MySQL` — нет.**

*Приведите ответ в свободной форме.*

**Индексы, которые есть в PostgreSQL, но отсутствуют в MySQL:**

1. **GIN (Generalized Inverted Index):**

- Используется для полнотекстового поиска и работы с `JSONB`.

2. **GIST (Generalized Search Tree):**

- Подходит для геометрических данных, полнотекстового поиска и других сложных типов данных.

3. **BRIN (Block Range INdex):**

- Оптимизирован для больших таблиц с данными, которые имеют естественную сортировку (например, временные метки).

4. **SP-GIST (Space-Partitioned Generalized Search Tree):**

- Подходит для данных с пространственным разделением (например, геоданные).

5. **Hash Index:**

- Хотя `hash`-индексы есть и в `MySQL`, в `PostgreSQL` они более гибкие и поддерживаются для большего числа типов данных.

---