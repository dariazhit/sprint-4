# sprint-4

/* Проект «Секреты Тёмнолесья»
 * Цель проекта: изучить влияние характеристик игроков и их игровых персонажей 
 * на покупку внутриигровой валюты «райские лепестки», а также оценить 
 * активность игроков при совершении внутриигровых покупок
 * 
 * Автор: Житняя Дарья
 * Дата: 16.09.2025
*/

-- Часть 1. Исследовательский анализ данных
-- Задача 1. Исследование доли платящих игроков

-- 1.1. Доля платящих пользователей по всем данным:
-- Напишите ваш запрос здесь
```
SELECT
COUNT(id) AS total_users,
--COUNT(CASE WHEN payer=1 THEN id END) 
SUM(payer) AS paying_users,
SUM(payer)/COUNT(id)::NUMERIC(10, 2) AS paying_rate
FROM fantasy.users;
```
-- 1.2. Доля платящих пользователей в разрезе расы персонажа:
-- Напишите ваш запрос здесь
```
SELECT 
race,
race_id,
SUM(payer) AS paying_race,
COUNT(id) AS total_race,
SUM(payer)/COUNT(id)::NUMERIC(10, 2) AS paying_race_rate
FROM fantasy.users
JOIN fantasy.race USING(race_id)
GROUP BY race_id, race
ORDER BY paying_race DESC, total_race DESC, paying_race_rate DESC;
```
-- Задача 2. Исследование внутриигровых покупок

-- 2.1. Статистические показатели по полю amount:
-- Напишите ваш запрос здесь
```
SELECT
COUNT(transaction_id) AS total_events,
SUM(amount) AS total_amount,
MIN(amount) AS min_amount,
MAX(amount) AS max_amount,
AVG(amount) AS avg_amount,
PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY amount) AS median_amount,
STDDEV_SAMP(amount) AS sddev_amount
FROM fantasy.events;
```
-- 2.2: Аномальные нулевые покупки:
-- Напишите ваш запрос здесь
```
SELECT 
COUNT(CASE WHEN amount=0 THEN transaction_id END) AS zero_amount_count,
COUNT(CASE WHEN amount=0 THEN transaction_id END)/COUNT(transaction_id)::float AS zero_amount_rate
FROM fantasy.events;
```
-- 2.3: Популярные эпические предметы:
-- Напишите ваш запрос здесь
```
SELECT 
game_items,
item_code,
COUNT(transaction_id) AS total_buyings,
COUNT(transaction_id)/(
SELECT
COUNT(transaction_id)
FROM fantasy.events
WHERE amount>0)::float AS buyings_rate,
COUNT(DISTINCT id)/(
SELECT
COUNT(DISTINCT id)
FROM fantasy.events
WHERE amount>0
)::float AS users_rate
FROM fantasy.events
JOIN fantasy.items USING(item_code)
WHERE amount>0
GROUP BY item_code, game_items
ORDER BY users_rate DESC;
```

-- Часть 2. Решение ad hoc-задачbи
-- Задача: Зависимость активности игроков от расы персонажа:
-- Напишите ваш запрос здесь
```
WITH race_stats AS (
SELECT 
race_id,
COUNT(id) AS race_users
FROM fantasy.users
GROUP BY race_id),
--посчитали количество всех игроков для каждой расы
buying_stats AS (
SELECT 
race_id,
COUNT(DISTINCT id) AS buying_users,
COUNT(CASE WHEN payer=1 THEN id END)/COUNT(DISTINCT id)::float AS paying_buying_rate
FROM fantasy.users 
WHERE id IN (
SELECT DISTINCT id 
FROM fantasy.events
WHERE amount>0
)
GROUP BY race_id
),
-- посчитали количество игроков,совершивших покупку, и долю платящих среди них
buyings_rate AS (
SELECT 
race_id,
COUNT(transaction_id) AS total_buyings,
SUM(amount) AS total_amount
FROM fantasy.events 
JOIN fantasy.users USING(id)
WHERE amount>0
GROUP BY race_id
)
--посчитали статистику для покупок: их количество и сумму 
SELECT 
race,
race_id,
race_users,
buying_users,
buying_users/race_users::float AS buying_rate,
paying_buying_rate,
total_buyings/buying_users::float AS avg_buyings,
total_amount/total_buyings::float AS avg_amount,
total_amount/buying_users::float AS avg_sum_amount
FROM  race_stats
JOIN buying_stats USING(race_id)
JOIN buyings_rate USING(race_id)
JOIN fantasy.race USING(race_id);


```
