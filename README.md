Пояснения к схеме данных: перед вами набор таблиц, которые содержат информацию о продажах некоторых товаров в 3-х различных магазинах, а также таблицу с планами продаж на уровень каждого магазина и каждого товара. 

В таблицах вида «shop_…» представлена информация о количестве продаж того или иного товара за конкретный день. 

В таблице plan хранится информация о планировании по продажам тех или иных товаров на конец месяца. 

В таблице products хранится общая информация по каждому из товаров.

Задание:

Вам выпала редкая возможность — помочь аналитикам оценить эффективность планирования продаж линейки отечественных смартфонов. Линейка насчитывает целых 3 модели: «Испорченный телефон», «Сарафанное радио» и «Патефон». Вы неплохого прокачали ваших аналитиков в SQL, и они даже смогли соорудить подобную конструкцию (см. выше). Однако она не лишена недостатков — в ней отсутствует одна важная связующая деталь (таблица). Добавить недостающую деталь будет вашим первым заданием. 

После того как схема приобретет законченный вид, вам необходимо решить главную задачу — собрать вашу первую витрину! Как уже изначально было озвучено, аналитикам нужно оценить, насколько отдел планирования хорошо делает свою работу. Для этого вам необходимо разработать SQL-скрипт, который формирует таблицу со следующим набором атрибутов:

shop_name — название магазина,
product_name — название товара,
sales_fact — количество фактических продаж на конец месяца,
sales_plan — количество запланированных продаж на конец месяца,
sales_fact/sales_plan — отношение количества фактических продаже к запланированному,
income_fact — фактический доход,
income_plan — планируемый доход,
income_fact/income_plan — отношение фактического дохода к запланированному.

![image](https://github.com/ddoser11/1t_hometask_2.5/assets/88766405/9890de27-4cd5-48c5-bbcc-7727e573e135)


# 1t_hometask_2.5 - SOLVING
1.В первой части не хватает таблицы с id магазина и названия магазина

2.
ALTER TABLE shop_dns add shop_name CHARACTER VARYING(20) DEFAULT 'shop_dns';
ALTER TABLE shop_mvideo add shop_name CHARACTER VARYING(20) DEFAULT 'shop_mvideo';
ALTER TABLE shop_sitilink add shop_name CHARACTER VARYING(20) DEFAULT 'shop_sitilink';


with shop_sales as (
SELECT t1.shop_name, t1.product_id, SUM(sales_cnt) AS sales_fact, date_trunc('month', date) as month
from (
SELECT shop_name, product_id, sales_cnt, date  
FROM shop_dns 
union all
SELECT shop_name, product_id, sales_cnt, date  
FROM shop_mvideo
union all
SELECT shop_name, product_id, sales_cnt, date  
FROM shop_sitilink) as t1
GROUP by  month, t1.shop_name, t1.product_id
ORDER by  month, t1.shop_name, t1.product_id)
select plan.shop_name,
products.product_name, 
shop_sales.sales_fact, 
plan_cnt AS sales_plan,
ROUND(sales_fact::numeric/plan_cnt::numeric,2) as "sales_fact/sales_plan" ,
sales_fact*products.price as "income_fact",
plan_cnt*products.price as "income_plan",
ROUND(shop_sales.sales_fact::numeric*products.price::numeric/plan_cnt::numeric/products.price::numeric,2) as "income_fact/income_plan",
month
from plan
full join shop_sales on plan.shop_name=shop_sales.shop_name and plan.product_id=shop_sales.product_id and month = date_trunc('month', plan_date)
full join products on  plan.product_id=products.product_id 
GROUP by  month, plan.shop_name, products.product_name, shop_sales.sales_fact, plan.plan_cnt, products.price
ORDER by month,   plan.shop_name, products.product_name

![image](https://github.com/ddoser11/1t_hometask_2.5/assets/88766405/c1dcd49d-0ef1-46a4-a90b-5a05703f441d)
