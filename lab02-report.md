# Лабораторна робота 2. Створення складних SQL запитів

## Загальна інформація

**Здобувач освіти:** Кондратюк Дмитро Михайлович
**Група:** ІПЗ-33
**Обраний рівень складності:** 3 (виконано рівні 1, 2 та 3)

> СУБД: PostgreSQL (Supabase). База даних: інтернет-магазин «ТехноМарт» з ЛР1
> (8 таблиць: regions — 25, suppliers — 8, categories — 10, products — 25, employees — 8, customers — 15, orders — 31, order_items — 47 рядків).
> Результати нижче — перші 10 рядків (для довших вибірок вказано загальну кількість).
> Для детермінованості сортування виконано за числовими ключами (`category_id`, `product_id` тощо), а не за текстовими полями.

## Виконання завдань

### Рівень 1

#### 1. З'єднання таблиць

**Завдання 1.1:** INNER JOIN - список товарів з категоріями та постачальниками

```sql
SELECT p.product_name, c.category_name, s.company_name AS supplier, p.unit_price
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
INNER JOIN suppliers s  ON p.supplier_id = s.supplier_id
ORDER BY c.category_id, p.product_id;
```

**Результат виконання:**
```
product_name                         | category_name          | supplier                     | unit_price
-------------------------------------+------------------------+------------------------------+-----------
iPhone 15 128GB Чорний               | Смартфони та телефони  | ТОВ "Електроніка Плюс"       | 29999.00  
Samsung Galaxy S24 256GB Фіолетовий  | Смартфони та телефони  | ПП "Комп'ютерні технології"  | 27999.00  
Xiaomi Redmi Note 13 Pro 128GB Синій | Смартфони та телефони  | ТОВ "Мобільний світ"         | 12999.00  
Google Pixel 8 128GB Білий           | Смартфони та телефони  | ТОВ "Техно Імпорт"           | 24999.00  
OnePlus 12 256GB Зелений             | Смартфони та телефони  | ТОВ "Електроніка Плюс"       | 22999.00  
MacBook Air M2 13" 256GB Сріблястий  | Ноутбуки та комп'ютери | ПП "Комп'ютерні технології"  | 45999.00  
Dell XPS 13 Plus Intel i7 512GB      | Ноутбуки та комп'ютери | ПАТ "Дніпро Електронікс"     | 52999.00  
ASUS ROG Strix G15 RTX 4060 1TB      | Ноутбуки та комп'ютери | ТОВ "Мобільний світ"         | 42999.00  
Lenovo ThinkPad X1 Carbon Gen 11     | Ноутбуки та комп'ютери | ТОВ "Львівські комп'ютери"   | 48999.00  
HP Pavilion 15 AMD Ryzen 7 512GB     | Ноутбуки та комп'ютери | ТОВ "Запорізька електроніка" | 28999.00  
... показано 10 з 25 рядків
```

**Пояснення:** Запит з'єднує три таблиці: `products` (основна), `categories` (за `category_id`) та `suppliers` (за `supplier_id`). INNER JOIN залишає лише ті рядки, для яких знайдено пару в обох довідниках. Усі 25 товарів мають і категорію, і постачальника, тому втрат немає: результат містить 25 рядків. Натомість три категорії без товарів («Фото та відео», «Розумний дім», «Спортивні технології») у цьому результаті відсутні — для INNER JOIN вони «зайві».


**Завдання 1.2:** LEFT JOIN - клієнти з кількістю замовлень

```sql
SELECT c.contact_name, c.customer_type, r.region_name,
       COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o  ON c.customer_id = o.customer_id
LEFT JOIN regions r ON c.region_id = r.region_id
GROUP BY c.customer_id, c.contact_name, c.customer_type, r.region_name
ORDER BY order_count DESC, c.customer_id;
```

**Результат виконання:**
```
contact_name                   | customer_type | region_name              | order_count
-------------------------------+---------------+--------------------------+------------
Петров Іван Миколайович        | individual    | м. Київ                  | 3          
Іванова Марія Сергіївна        | individual    | Харківська область       | 2          
Коваленко Андрій Петрович      | company       | м. Київ                  | 2          
Мельник Ольга Іванівна         | individual    | Львівська область        | 2          
Шевченко Віктор Олександрович  | company       | Харківська область       | 2          
Гриценко Наталія Володимирівна | individual    | Одеська область          | 2          
Білоус Дмитро Сергійович       | company       | Дніпропетровська область | 2          
Сидоренко Тетяна Миколаївна    | individual    | Львівська область        | 2          
Павленко Олексій Іванович      | individual    | Харківська область       | 2          
Федоренко Ірина Петрівна       | company       | м. Київ                  | 2          
... показано 10 з 15 рядків
```

**Додатковий запит, що показує різницю між INNER та LEFT JOIN:**

```sql
SELECT 'INNER JOIN' AS join_type, COUNT(DISTINCT c.category_id) AS categories_in_result
FROM categories c
INNER JOIN products p ON p.category_id = c.category_id
UNION ALL
SELECT 'LEFT JOIN', COUNT(DISTINCT c.category_id)
FROM categories c
LEFT JOIN products p ON p.category_id = c.category_id;
```

**Результат виконання:**
```
join_type  | categories_in_result
-----------+---------------------
INNER JOIN | 7                   
LEFT JOIN  | 10                  
(2 rows)
```

**Пояснення:** `customers` — ліва (головна) таблиця, тому LEFT JOIN повертає всіх 15 клієнтів, навіть якщо в `orders` для них немає записів (тоді `COUNT(o.order_id)` дорівнює 0, бо COUNT ігнорує NULL). У поточних даних кожен клієнт зробив мінімум 2 замовлення (Петров Іван — 3), тому для клієнтів INNER JOIN дав би той самий результат. Різниця добре видна на парі `categories` → `products`: INNER JOIN залишає 7 категорій (лише ті, де є товари), а LEFT JOIN — усі 10, бо включає і 3 порожні категорії з NULL у стовпцях товару. Тому LEFT JOIN доцільно використовувати, коли потрібен повний перелік головних сутностей (звіти, пошук «сиріт»), а INNER JOIN — коли потрібні лише пов'язані записи.


**Завдання 1.3:** Множинне з'єднання - детальна інформація про замовлення

```sql
SELECT o.order_id,
       o.order_date,
       cu.contact_name AS customer,
       r.region_name,
       e.last_name || ' ' || e.first_name AS manager,
       p.product_name,
       cat.category_name,
       oi.quantity,
       oi.unit_price,
       ROUND(oi.quantity * oi.unit_price * (1 - oi.discount), 2) AS line_total
FROM orders o
JOIN customers   cu  ON o.customer_id  = cu.customer_id
LEFT JOIN regions r  ON cu.region_id   = r.region_id
JOIN employees   e   ON o.employee_id  = e.employee_id
JOIN order_items oi  ON o.order_id     = oi.order_id
JOIN products    p   ON oi.product_id  = p.product_id
JOIN categories  cat ON p.category_id  = cat.category_id
ORDER BY o.order_id, p.product_id;
```

**Результат виконання:**
```
order_id | order_date | customer                      | region_name        | manager          | product_name                          | category_name                | quantity | unit_price | line_total
---------+------------+-------------------------------+--------------------+------------------+---------------------------------------+------------------------------+----------+------------+-----------
1        | 2024-01-15 | Петров Іван Миколайович       | м. Київ            | Коваленко Марія  | Xiaomi Redmi Note 13 Pro 128GB Синій  | Смартфони та телефони        | 1        | 12999.00   | 12349.05  
1        | 2024-01-15 | Петров Іван Миколайович       | м. Київ            | Коваленко Марія  | Мікрохвильова піч Panasonic NN-ST45KW | Побутова техніка             | 2        | 699.00     | 1398.00   
2        | 2024-01-20 | Коваленко Андрій Петрович     | м. Київ            | Коваленко Марія  | MacBook Air M2 13" 256GB Сріблястий   | Ноутбуки та комп'ютери       | 2        | 45999.00   | 82798.20  
2        | 2024-01-20 | Коваленко Андрій Петрович     | м. Київ            | Коваленко Марія  | iPad Air M2 11" 128GB Синій           | Планшети та електронні книги | 2        | 3999.00    | 7598.10   
3        | 2024-01-22 | Іванова Марія Сергіївна       | Харківська область | Гриценко Наталія | iPhone 15 128GB Чорний                | Смартфони та телефони        | 1        | 29999.00   | 29999.00  
3        | 2024-01-22 | Іванова Марія Сергіївна       | Харківська область | Гриценко Наталія | Пральна машина LG F4V5VS6W 9кг        | Побутова техніка             | 1        | 8999.00    | 8999.00   
4        | 2024-02-01 | Мельник Ольга Іванівна        | Львівська область  | Мельник Андрій   | Пральна машина LG F4V5VS6W 9кг        | Побутова техніка             | 1        | 8999.00    | 8999.00   
4        | 2024-02-01 | Мельник Ольга Іванівна        | Львівська область  | Мельник Андрій   | Мікрохвильова піч Panasonic NN-ST45KW | Побутова техніка             | 1        | 699.00     | 699.00    
5        | 2024-02-10 | Шевченко Віктор Олександрович | Харківська область | Гриценко Наталія | Dell XPS 13 Plus Intel i7 512GB       | Ноутбуки та комп'ютери       | 1        | 52999.00   | 48759.08  
5        | 2024-02-10 | Шевченко Віктор Олександрович | Харківська область | Гриценко Наталія | ASUS ROG Strix G15 RTX 4060 1TB       | Ноутбуки та комп'ютери       | 1        | 42999.00   | 39559.08  
... показано 10 з 47 рядків
```

**Аналіз складності:** Запит з'єднує 7 таблиць: `orders` → `customers` → `regions` (LEFT) → `employees` → `order_items` → `products` → `categories`. Усі умови з'єднання — рівності за первинним/зовнішнім ключем, тому кожен JOIN індексований по PK. Послідовність: стартуємо з `orders` (31 рядок); приєднання `customers`, `employees`, `regions` — зв'язки «багато-до-одного», вони лише збагачують рядок і не збільшують їх кількість; `order_items` — зв'язок «один-до-багатьох», саме тут кількість рядків зростає з 31 до 47 (за кількістю позицій); `products` і `categories` знову «багато-до-одного». Для `regions` використано LEFT JOIN, оскільки `region_id` у клієнта може бути NULL. Складність — висока за кількістю з'єднань, але лінійна за обсягом даних (планувальник PostgreSQL зазвичай обирає Hash Join / Nested Loop по індексах).


#### 2. Агрегатні функції

**Завдання 2.1:** Статистика товарів за категоріями

```sql
SELECT c.category_name,
       COUNT(p.product_id)          AS product_count,
       ROUND(AVG(p.unit_price), 2)  AS avg_price,
       MIN(p.unit_price)            AS min_price,
       MAX(p.unit_price)            AS max_price
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_id, c.category_name
ORDER BY product_count DESC, c.category_id;
```

**Результат виконання:**
```
category_name                | product_count | avg_price | min_price | max_price
-----------------------------+---------------+-----------+-----------+----------
Смартфони та телефони        | 5             | 23799.00  | 12999.00  | 29999.00 
Ноутбуки та комп'ютери       | 5             | 43999.00  | 28999.00  | 52999.00 
Телевізори та аудіо          | 4             | 46749.00  | 29999.00  | 62999.00 
Побутова техніка             | 3             | 17665.67  | 4999.00   | 28999.00 
Ігрові консолі та ігри       | 3             | 15999.00  | 12999.00  | 17999.00 
Аксесуари та комплектуючі    | 3             | 4565.67   | 699.00    | 8999.00  
Планшети та електронні книги | 2             | 28499.00  | 23999.00  | 32999.00 
Фото та відео                | 0             | NULL      | NULL      | NULL     
Розумний дім                 | 0             | NULL      | NULL      | NULL     
Спортивні технології         | 0             | NULL      | NULL      | NULL     
(10 rows)
```

**Завдання 2.2:** Продажі за регіонами з використанням HAVING

```sql
SELECT r.region_name,
       COUNT(DISTINCT o.order_id)        AS orders_count,
       SUM(oi.quantity)                  AS items_sold,
       ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2)              AS total_sales
FROM orders o
JOIN customers   cu ON o.customer_id = cu.customer_id
JOIN regions     r  ON cu.region_id  = r.region_id
JOIN order_items oi ON o.order_id    = oi.order_id
WHERE o.order_status <> 'cancelled'
GROUP BY r.region_id, r.region_name
HAVING SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) > 100000
ORDER BY total_sales DESC;
```

**Результат виконання:**
```
region_name              | orders_count | items_sold | total_sales
-------------------------+--------------+------------+------------
Дніпропетровська область | 6            | 33         | 894191.64  
м. Київ                  | 9            | 33         | 889560.66  
Харківська область       | 6            | 16         | 333315.00  
(3 rows)
```

Виручка рахується як `quantity * unit_price * (1 - discount)` по позиціях замовлення, регіон — регіон клієнта. HAVING відсікає регіони з виручкою ≤ 100 000: Львівська (≈ 64 тис.) та Одеська (≈ 91 тис.) області не потрапили до результату. Фільтр за агрегатом у WHERE неможливий, бо WHERE виконується до групування.

**Завдання 2.3:** Постачальники з кількістю товарів більше 2

```sql
SELECT s.company_name,
       COUNT(p.product_id)         AS product_count,
       ROUND(AVG(p.unit_price), 2) AS avg_price
FROM suppliers s
JOIN products p ON s.supplier_id = p.supplier_id
GROUP BY s.supplier_id, s.company_name
HAVING COUNT(p.product_id) > 2
ORDER BY product_count DESC, s.supplier_id;
```

**Результат виконання:**
```
company_name                | product_count | avg_price
----------------------------+---------------+----------
ТОВ "Електроніка Плюс"      | 6             | 27165.67 
ПП "Комп'ютерні технології" | 5             | 33199.00 
ТОВ "Мобільний світ"        | 4             | 19749.00 
ТОВ "Техно Імпорт"          | 3             | 35665.67 
(4 rows)
```

Із 8 постачальників умову `COUNT > 2` виконують 4; решта мають по 1–2 товари.


#### 3. Базові підзапити

**Завдання 3.1:** Товари з ціною вище середньої по категорії

```sql
SELECT p.product_name, p.unit_price, c.category_name,
       (SELECT ROUND(AVG(p3.unit_price), 2)
        FROM products p3
        WHERE p3.category_id = p.category_id) AS category_avg
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
WHERE p.unit_price > (
    SELECT AVG(p2.unit_price)
    FROM products p2
    WHERE p2.category_id = p.category_id
)
ORDER BY c.category_id, p.unit_price DESC;
```

**Результат виконання:**
```
product_name                        | unit_price | category_name          | category_avg
------------------------------------+------------+------------------------+-------------
iPhone 15 128GB Чорний              | 29999.00   | Смартфони та телефони  | 23799.00    
Samsung Galaxy S24 256GB Фіолетовий | 27999.00   | Смартфони та телефони  | 23799.00    
Google Pixel 8 128GB Білий          | 24999.00   | Смартфони та телефони  | 23799.00    
Dell XPS 13 Plus Intel i7 512GB     | 52999.00   | Ноутбуки та комп'ютери | 43999.00    
Lenovo ThinkPad X1 Carbon Gen 11    | 48999.00   | Ноутбуки та комп'ютери | 43999.00    
MacBook Air M2 13" 256GB Сріблястий | 45999.00   | Ноутбуки та комп'ютери | 43999.00    
LG OLED C3 65" 4K Smart TV          | 62999.00   | Телевізори та аудіо    | 46749.00    
Sony Bravia XR A80L 55" OLED        | 58999.00   | Телевізори та аудіо    | 46749.00    
Холодильник Samsung RB38T7762SA/UA  | 28999.00   | Побутова техніка       | 17665.67    
Пральна машина LG F4V5VS6W 9кг      | 18999.00   | Побутова техніка       | 17665.67    
... показано 10 з 14 рядків
```

Всього 14 товарів дорожчі за середнє по своїй категорії. Підзапит корельований: він посилається на `p.category_id` зовнішнього запиту, тому обчислюється окремо для кожного рядка `products`.

**Завдання 3.2:** Клієнти з замовленнями у 2024 році

```sql
SELECT c.customer_id, c.contact_name, c.customer_type, c.city
FROM customers c
WHERE c.customer_id IN (
    SELECT o.customer_id
    FROM orders o
    WHERE o.order_date >= '2024-01-01'
      AND o.order_date <  '2025-01-01'
)
ORDER BY c.customer_id;
```

**Результат виконання:**
```
customer_id | contact_name                   | customer_type | city  
------------+--------------------------------+---------------+-------
1           | Петров Іван Миколайович        | individual    | Київ  
2           | Іванова Марія Сергіївна        | individual    | Харків
3           | Коваленко Андрій Петрович      | company       | Київ  
4           | Мельник Ольга Іванівна         | individual    | Львів 
5           | Шевченко Віктор Олександрович  | company       | Харків
6           | Гриценко Наталія Володимирівна | individual    | Одеса 
7           | Білоус Дмитро Сергійович       | company       | Дніпро
8           | Сидоренко Тетяна Миколаївна    | individual    | Львів 
9           | Павленко Олексій Іванович      | individual    | Харків
10          | Федоренко Ірина Петрівна       | company       | Київ  
... показано 10 з 15 рядків
```

Усі 15 клієнтів мали замовлення у 2024 році (у даних є лише 2024 рік). Для демонстрації EXISTS / NOT EXISTS:

```sql
-- Варіант з EXISTS: клієнти, які мають відкриті замовлення (не delivered)
SELECT c.customer_id, c.contact_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.order_status <> 'delivered'
)
ORDER BY c.customer_id;
```

**Результат виконання:**
```
customer_id | contact_name                
------------+-----------------------------
1           | Петров Іван Миколайович     
7           | Білоус Дмитро Сергійович    
10          | Федоренко Ірина Петрівна    
13          | Лисенко Володимир Іванович  
15          | Морозенко Сергій Миколайович
(5 rows)
```

```sql
-- NOT EXISTS: товари, яких жодного разу не було в замовленнях
SELECT p.product_id, p.product_name, p.unit_price
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.product_id
)
ORDER BY p.product_id;
```

**Результат виконання:**
```
product_id | product_name                           | unit_price
-----------+----------------------------------------+-----------
22         | Бездротова клавіатура Logitech MX Keys | 3999.00   
23         | PlayStation 5 825GB                    | 17999.00  
24         | Xbox Series X 1TB                      | 16999.00  
25         | Nintendo Switch OLED 64GB              | 12999.00  
(4 rows)
```

**Завдання 3.3:** Товари з загальною кількістю продажів

```sql
SELECT p.product_id, p.product_name, p.unit_price,
       COALESCE((SELECT SUM(oi.quantity)
                 FROM order_items oi
                 WHERE oi.product_id = p.product_id), 0) AS total_sold
FROM products p
ORDER BY total_sold DESC, p.product_id;
```

**Результат виконання:**
```
product_id | product_name                          | unit_price | total_sold
-----------+---------------------------------------+------------+-----------
18         | iPad Air M2 11" 128GB Синій           | 23999.00   | 19        
6          | MacBook Air M2 13" 256GB Сріблястий   | 45999.00   | 15        
17         | Мікрохвильова піч Panasonic NN-ST45KW | 4999.00    | 13        
9          | Lenovo ThinkPad X1 Carbon Gen 11      | 48999.00   | 7         
7          | Dell XPS 13 Plus Intel i7 512GB       | 52999.00   | 6         
16         | Пральна машина LG F4V5VS6W 9кг        | 18999.00   | 6         
11         | Samsung QLED QE55Q80C 55" 4K          | 34999.00   | 5         
1          | iPhone 15 128GB Чорний                | 29999.00   | 4         
8          | ASUS ROG Strix G15 RTX 4060 1TB       | 42999.00   | 4         
2          | Samsung Galaxy S24 256GB Фіолетовий   | 27999.00   | 3         
... показано 10 з 25 рядків
```


### Рівень 2

#### 4. Складні з'єднання

**Завдання 4.1:** RIGHT JOIN - аналіз категорій та товарів

```sql
SELECT c.category_name,
       COUNT(p.product_id)               AS products_count,
       COALESCE(ROUND(AVG(p.unit_price), 2), 0) AS avg_price
FROM products p
RIGHT JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY products_count DESC, c.category_id;
```

**Результат виконання:**
```
category_name                | products_count | avg_price
-----------------------------+----------------+----------
Смартфони та телефони        | 5              | 23799.00 
Ноутбуки та комп'ютери       | 5              | 43999.00 
Телевізори та аудіо          | 4              | 46749.00 
Побутова техніка             | 3              | 17665.67 
Ігрові консолі та ігри       | 3              | 15999.00 
Аксесуари та комплектуючі    | 3              | 4565.67  
Планшети та електронні книги | 2              | 28499.00 
Фото та відео                | 0              | 0.00     
Розумний дім                 | 0              | 0.00     
Спортивні технології         | 0              | 0.00     
(10 rows)
```

RIGHT JOIN зберігає всі рядки правої таблиці `categories`; для трьох категорій без товарів `COUNT(p.product_id)` = 0 (а не 1, бо рахуються лише не-NULL значення), середня ціна замінена на 0 через `COALESCE`. FULL OUTER JOIN залишає невідповідні рядки з обох сторін; тут «осиротілими» є лише категорії:

```sql
-- FULL OUTER JOIN: категорії без товарів та товари без категорії в одному результаті
SELECT c.category_name, p.product_name
FROM categories c
FULL OUTER JOIN products p ON c.category_id = p.category_id
WHERE c.category_id IS NULL OR p.product_id IS NULL;
```

**Результат виконання:**
```
category_name        | product_name
---------------------+-------------
Фото та відео        | NULL        
Розумний дім         | NULL        
Спортивні технології | NULL        
(3 rows)
```

**Завдання 4.2:** Self-join - співробітники та керівники

```sql
SELECT e1.first_name || ' ' || e1.last_name AS employee,
       e1.title                              AS employee_title,
       e2.first_name || ' ' || e2.last_name AS manager,
       e2.title                              AS manager_title
FROM employees e1
LEFT JOIN employees e2 ON e1.reports_to = e2.employee_id
ORDER BY e1.employee_id;
```

**Результат виконання:**
```
employee           | employee_title         | manager            | manager_title        
-------------------+------------------------+--------------------+----------------------
Олександр Петренко | Генеральний директор   | NULL               | NULL                 
Марія Коваленко    | Менеджер з продажу     | Олександр Петренко | Генеральний директор 
Сергій Іваненко    | Менеджер з закупівель  | Олександр Петренко | Генеральний директор 
Ольга Шевченко     | Головний бухгалтер     | Олександр Петренко | Генеральний директор 
Андрій Мельник     | Менеджер з продажу     | Марія Коваленко    | Менеджер з продажу   
Наталія Гриценко   | Менеджер з продажу     | Марія Коваленко    | Менеджер з продажу   
Дмитро Білоус      | Менеджер зі складу     | Сергій Іваненко    | Менеджер з закупівель
Тетяна Сидоренко   | Спеціаліст з логістики | Сергій Іваненко    | Менеджер з закупівель
(8 rows)
```

Таблиця `employees` з'єднується сама із собою: `e1` — підлеглий, `e2` — його керівник (`e1.reports_to = e2.employee_id`). LEFT JOIN потрібен, щоб у результат потрапив генеральний директор, у якого `reports_to IS NULL`.

**Завдання 4.3:** З'єднання з додатковою умовою в ON

```sql
-- Умовне з'єднання: додаткова умова в ON зберігає всіх клієнтів
SELECT c.contact_name,
       COUNT(o.order_id) AS open_orders
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
                  AND o.order_status IN ('pending', 'processing', 'shipped')
GROUP BY c.customer_id, c.contact_name
ORDER BY open_orders DESC, c.customer_id;
```

**Результат виконання:**
```
contact_name                   | open_orders
-------------------------------+------------
Петров Іван Миколайович        | 1          
Білоус Дмитро Сергійович       | 1          
Федоренко Ірина Петрівна       | 1          
Лисенко Володимир Іванович     | 1          
Морозенко Сергій Миколайович   | 1          
Іванова Марія Сергіївна        | 0          
Коваленко Андрій Петрович      | 0          
Мельник Ольга Іванівна         | 0          
Шевченко Віктор Олександрович  | 0          
Гриценко Наталія Володимирівна | 0          
... показано 10 з 15 рядків
```

Умова `order_status IN (...)` винесена в `ON`, а не в `WHERE`, тому всі 15 клієнтів залишаються в результаті (5 мають по одному відкритому замовленню, 10 — 0). Якщо перенести цю умову в WHERE, клієнти без відкритих замовлень зникнуть і LEFT JOIN фактично перетвориться на INNER.


#### 5. Віконні функції

**Завдання 5.1:** Ранжування товарів за ціною в категоріях

```sql
SELECT p.product_name,
       c.category_name,
       p.unit_price,
       RANK()       OVER (PARTITION BY c.category_id ORDER BY p.unit_price DESC) AS price_rank,
       DENSE_RANK() OVER (PARTITION BY c.category_id ORDER BY p.unit_price DESC) AS price_dense_rank,
       ROW_NUMBER() OVER (PARTITION BY c.category_id ORDER BY p.unit_price DESC) AS row_num
FROM products p
JOIN categories c ON p.category_id = c.category_id
ORDER BY c.category_id, p.unit_price DESC;
```

**Результат виконання:**
```
product_name                         | category_name          | unit_price | price_rank | price_dense_rank | row_num
-------------------------------------+------------------------+------------+------------+------------------+--------
iPhone 15 128GB Чорний               | Смартфони та телефони  | 29999.00   | 1          | 1                | 1      
Samsung Galaxy S24 256GB Фіолетовий  | Смартфони та телефони  | 27999.00   | 2          | 2                | 2      
Google Pixel 8 128GB Білий           | Смартфони та телефони  | 24999.00   | 3          | 3                | 3      
OnePlus 12 256GB Зелений             | Смартфони та телефони  | 22999.00   | 4          | 4                | 4      
Xiaomi Redmi Note 13 Pro 128GB Синій | Смартфони та телефони  | 12999.00   | 5          | 5                | 5      
Dell XPS 13 Plus Intel i7 512GB      | Ноутбуки та комп'ютери | 52999.00   | 1          | 1                | 1      
Lenovo ThinkPad X1 Carbon Gen 11     | Ноутбуки та комп'ютери | 48999.00   | 2          | 2                | 2      
MacBook Air M2 13" 256GB Сріблястий  | Ноутбуки та комп'ютери | 45999.00   | 3          | 3                | 3      
ASUS ROG Strix G15 RTX 4060 1TB      | Ноутбуки та комп'ютери | 42999.00   | 4          | 4                | 4      
HP Pavilion 15 AMD Ryzen 7 512GB     | Ноутбуки та комп'ютери | 28999.00   | 5          | 5                | 5      
... показано 10 з 25 рядків
```

В межах однієї категорії повторюваних цін немає, тому три функції збігаються. Різницю показує ранжування по всьому каталогу, де є однакові ціни (29 999 — iPhone 15 і TCL C845; 28 999 — HP Pavilion і холодильник Samsung):

```sql
-- Ранжування по всьому каталогу: видно різницю RANK / DENSE_RANK на однакових цінах
SELECT product_name, unit_price,
       ROW_NUMBER() OVER (ORDER BY unit_price DESC, product_id) AS row_num,
       RANK()       OVER (ORDER BY unit_price DESC)             AS rank_num,
       DENSE_RANK() OVER (ORDER BY unit_price DESC)             AS dense_rank_num
FROM products
ORDER BY unit_price DESC, product_id
LIMIT 12;
```

**Результат виконання:**
```
product_name                        | unit_price | row_num | rank_num | dense_rank_num
------------------------------------+------------+---------+----------+---------------
LG OLED C3 65" 4K Smart TV          | 62999.00   | 1       | 1        | 1             
Sony Bravia XR A80L 55" OLED        | 58999.00   | 2       | 2        | 2             
Dell XPS 13 Plus Intel i7 512GB     | 52999.00   | 3       | 3        | 3             
Lenovo ThinkPad X1 Carbon Gen 11    | 48999.00   | 4       | 4        | 4             
MacBook Air M2 13" 256GB Сріблястий | 45999.00   | 5       | 5        | 5             
ASUS ROG Strix G15 RTX 4060 1TB     | 42999.00   | 6       | 6        | 6             
Samsung QLED QE55Q80C 55" 4K        | 34999.00   | 7       | 7        | 7             
Samsung Galaxy Tab S9+ 256GB        | 32999.00   | 8       | 8        | 8             
iPhone 15 128GB Чорний              | 29999.00   | 9       | 9        | 9             
TCL C845 Mini LED 65" 4K            | 29999.00   | 10      | 9        | 9             
HP Pavilion 15 AMD Ryzen 7 512GB    | 28999.00   | 11      | 11       | 10            
Холодильник Samsung RB38T7762SA/UA  | 28999.00   | 12      | 11       | 10            
(12 rows)
```

`ROW_NUMBER()` завжди дає унікальні номери (9, 10), `RANK()` дає однаковий ранг із пропуском наступного (9, 9, потім 11), `DENSE_RANK()` — однаковий ранг без пропусків (9, 9, потім 10).

**Завдання 5.2:** Порівняння замовлень з попередніми датами

```sql
WITH order_totals AS (
    SELECT o.order_id, o.customer_id, o.order_date,
           ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2) AS order_total
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY o.order_id, o.customer_id, o.order_date
)
SELECT customer_id, order_id, order_date, order_total,
       LAG(order_date)  OVER w AS prev_order_date,
       LEAD(order_date) OVER w AS next_order_date,
       order_date - LAG(order_date) OVER w AS days_since_prev,
       order_total - LAG(order_total) OVER w AS diff_vs_prev
FROM order_totals
WINDOW w AS (PARTITION BY customer_id ORDER BY order_date)
ORDER BY customer_id, order_date;
```

**Результат виконання:**
```
customer_id | order_id | order_date | order_total | prev_order_date | next_order_date | days_since_prev | diff_vs_prev
------------+----------+------------+-------------+-----------------+-----------------+-----------------+-------------
1           | 1        | 2024-01-15 | 13747.05    | NULL            | 2024-05-01      | NULL            | NULL        
1           | 16       | 2024-05-01 | 57959.08    | 2024-01-15      | 2024-08-10      | 107             | 44212.03    
1           | 28       | 2024-08-10 | 17098.10    | 2024-05-01      | NULL            | 101             | -40860.98   
2           | 3        | 2024-01-22 | 38998.00    | NULL            | 2024-05-10      | NULL            | NULL        
2           | 17       | 2024-05-10 | 25846.05    | 2024-01-22      | NULL            | 109             | -13151.95   
3           | 2        | 2024-01-20 | 90396.30    | NULL            | 2024-06-20      | NULL            | NULL        
3           | 22       | 2024-06-20 | 166596.60   | 2024-01-20      | NULL            | 152             | 76200.30    
4           | 4        | 2024-02-01 | 9698.00     | NULL            | 2024-05-18      | NULL            | NULL        
4           | 18       | 2024-05-18 | 3999.00     | 2024-02-01      | NULL            | 107             | -5699.00    
5           | 5        | 2024-02-10 | 88318.16    | NULL            | 2024-07-01      | NULL            | NULL        
5           | 23       | 2024-07-01 | 129595.56   | 2024-02-10      | NULL            | 142             | 41277.40    
6           | 6        | 2024-02-18 | 17999.00    | NULL            | 2024-05-25      | NULL            | NULL        
... показано 12 з 31 рядків
```

`PARTITION BY customer_id ORDER BY order_date` розбиває замовлення за клієнтами та впорядковує за датою. `LAG` повертає дані попереднього замовлення клієнта, `LEAD` — наступного; для першого/останнього замовлення результат NULL. Це дозволяє порахувати інтервал між покупками (`days_since_prev`) і зміну суми чека (`diff_vs_prev`). Усього 31 рядок — по одному на замовлення.

**Додатково — PARTITION BY з агрегатом (частка категорії у загальній виручці):**

```sql
SELECT c.category_name,
       ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2) AS category_revenue,
       ROUND(100.0 * SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) / SUM(SUM(oi.quantity * oi.unit_price * (1 - oi.discount))) OVER (), 2) AS pct_of_total,
       RANK() OVER (ORDER BY SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) DESC) AS revenue_rank
FROM order_items oi
JOIN products   p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY revenue_rank;
```

**Результат виконання:**
```
category_name                | category_revenue | pct_of_total | revenue_rank
-----------------------------+------------------+--------------+-------------
Ноутбуки та комп'ютери       | 1327311.75       | 58.42        | 1           
Телевізори та аудіо          | 467429.41        | 20.57        | 2           
Смартфони та телефони        | 266948.99        | 11.75        | 3           
Побутова техніка             | 94655.85         | 4.17         | 4           
Планшети та електронні книги | 85582.10         | 3.77         | 5           
Аксесуари та комплектуючі    | 29998.00         | 1.32         | 6           
(6 rows)
```


### Рівень 3

#### 6. Матеріалізовані представлення та рекурсивні запити

**Завдання 6.1:** Матеріалізоване представлення для аналізу продажів

```sql
CREATE MATERIALIZED VIEW mv_monthly_sales AS
SELECT
    CAST(EXTRACT(YEAR  FROM o.order_date) AS INTEGER) AS year,
    CAST(EXTRACT(MONTH FROM o.order_date) AS INTEGER) AS month,
    c.category_name,
    r.region_name,
    ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2)                                    AS total_revenue,
    COUNT(DISTINCT o.order_id)                              AS orders_count,
    ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) / COUNT(DISTINCT o.order_id), 2)       AS avg_order_value
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products    p  ON oi.product_id = p.product_id
JOIN categories  c  ON p.category_id = c.category_id
JOIN customers   cu ON o.customer_id = cu.customer_id
LEFT JOIN regions r ON cu.region_id = r.region_id
WHERE o.order_status = 'delivered'
GROUP BY 1, 2, c.category_name, r.region_name
WITH DATA;

-- Індекси для швидкого доступу до представлення
CREATE INDEX idx_mv_monthly_sales_date ON mv_monthly_sales(year, month);
-- Унікальний індекс потрібен для REFRESH ... CONCURRENTLY
CREATE UNIQUE INDEX uq_mv_monthly_sales
    ON mv_monthly_sales(year, month, category_name, region_name);

-- Оновлення даних (після появи нових замовлень)
REFRESH MATERIALIZED VIEW mv_monthly_sales;
-- або без блокування читачів:
-- REFRESH MATERIALIZED VIEW CONCURRENTLY mv_monthly_sales;
```

**Приклад використання та результат:**
```sql
SELECT * FROM mv_monthly_sales
ORDER BY year, month, total_revenue DESC;
```
```
year | month | category_name                | region_name              | total_revenue | orders_count | avg_order_value
-----+-------+------------------------------+--------------------------+---------------+--------------+----------------
2024 | 1     | Ноутбуки та комп'ютери       | м. Київ                  | 82798.20      | 1            | 82798.20       
2024 | 1     | Смартфони та телефони        | Харківська область       | 29999.00      | 1            | 29999.00       
2024 | 1     | Смартфони та телефони        | м. Київ                  | 12349.05      | 1            | 12349.05       
2024 | 1     | Побутова техніка             | Харківська область       | 8999.00       | 1            | 8999.00        
2024 | 1     | Планшети та електронні книги | м. Київ                  | 7598.10       | 1            | 7598.10        
2024 | 1     | Побутова техніка             | м. Київ                  | 1398.00       | 1            | 1398.00        
2024 | 2     | Ноутбуки та комп'ютери       | Дніпропетровська область | 129357.36     | 1            | 129357.36      
2024 | 2     | Ноутбуки та комп'ютери       | Харківська область       | 88318.16      | 1            | 88318.16       
2024 | 2     | Планшети та електронні книги | Одеська область          | 17999.00      | 1            | 17999.00       
2024 | 2     | Планшети та електронні книги | Дніпропетровська область | 10797.30      | 1            | 10797.30       
... показано 10 з 34 рядків
```

Загальна виручка за місяцями, отримана з представлення:
```sql
SELECT year, month, SUM(total_revenue) AS revenue
FROM mv_monthly_sales
GROUP BY year, month
ORDER BY year, month;
```
```
year | month | revenue  
-----+-------+----------
2024 | 1     | 143141.35
2024 | 2     | 256169.82
2024 | 3     | 145752.43
2024 | 4     | 415015.64
2024 | 5     | 104803.13
2024 | 6     | 198993.80
2024 | 7     | 208464.46
(7 rows)
```

**Пояснення:** Матеріалізоване представлення фізично зберігає результат складного запиту (5 з'єднань + агрегація) на диску, на відміну від звичайного VIEW, яке перераховується при кожному зверненні. Переваги: (1) звіти читаються миттєво — це звичайне сканування невеликої таблиці, а не повторне з'єднання й агрегація всіх замовлень; (2) на представлення можна створювати власні індекси (`idx_mv_monthly_sales_date`); (3) знижується навантаження на основні таблиці під час аналітики. Недолік — дані застарівають і оновлюються вручну командою `REFRESH MATERIALIZED VIEW` (з `CONCURRENTLY` — без блокування читачів, але потрібен унікальний індекс). У порівнянні з варіантом з методички поле `avg_order_value` рахується коректно як виручка / кількість замовлень (а не середнє по позиціях), а `year`/`month` приведені до INTEGER.

**Завдання 6.2:** Рекурсивний запит для ієрархії співробітників

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Базовий випадок: співробітники без керівника
    SELECT employee_id, first_name, last_name, title, reports_to,
           0 AS level,
           CAST(last_name || ' ' || first_name AS VARCHAR(1000)) AS hierarchy_path,
           LPAD(CAST(employee_id AS VARCHAR), 3, '0')            AS sort_key
    FROM employees
    WHERE reports_to IS NULL

    UNION ALL

    -- Рекурсивний випадок: підлеглі вже знайдених керівників
    SELECT e.employee_id, e.first_name, e.last_name, e.title, e.reports_to,
           eh.level + 1,
           CAST(eh.hierarchy_path || ' -> ' || e.last_name || ' ' || e.first_name AS VARCHAR(1000)),
           eh.sort_key || '.' || LPAD(CAST(e.employee_id AS VARCHAR), 3, '0')
    FROM employees e
    JOIN employee_hierarchy eh ON e.reports_to = eh.employee_id
    WHERE eh.level < 10        -- захист від циклів
)
SELECT employee_id,
       REPEAT('    ', level) || last_name || ' ' || first_name AS employee_tree,
       title, level, hierarchy_path
FROM employee_hierarchy
ORDER BY sort_key;
```

**Результат виконання:**
```
employee_id | employee_tree            | title                  | level | hierarchy_path                                           
------------+--------------------------+------------------------+-------+----------------------------------------------------------
1           | Петренко Олександр       | Генеральний директор   | 0     | Петренко Олександр                                       
2           |     Коваленко Марія      | Менеджер з продажу     | 1     | Петренко Олександр -> Коваленко Марія                    
5           |         Мельник Андрій   | Менеджер з продажу     | 2     | Петренко Олександр -> Коваленко Марія -> Мельник Андрій  
6           |         Гриценко Наталія | Менеджер з продажу     | 2     | Петренко Олександр -> Коваленко Марія -> Гриценко Наталія
3           |     Іваненко Сергій      | Менеджер з закупівель  | 1     | Петренко Олександр -> Іваненко Сергій                    
7           |         Білоус Дмитро    | Менеджер зі складу     | 2     | Петренко Олександр -> Іваненко Сергій -> Білоус Дмитро   
8           |         Сидоренко Тетяна | Спеціаліст з логістики | 2     | Петренко Олександр -> Іваненко Сергій -> Сидоренко Тетяна
4           |     Шевченко Ольга       | Головний бухгалтер     | 1     | Петренко Олександр -> Шевченко Ольга                     
(8 rows)
```

Anchor-частина вибирає співробітників без керівника (генеральний директор, `level = 0`), рекурсивна — підшукує підлеглих уже знайдених керівників і збільшує рівень. Дерево має 3 рівні: директор → 3 керівники відділів (Коваленко, Іваненко, Шевченко) → 4 виконавці. Умова `level < 10` захищає від зациклення. Для впорядкування «деревом» використано `sort_key` із ID, доповнених нулями.

**Завдання 6.3:** Збережена функція з динамічним SQL для параметризованої аналітики

```sql
CREATE OR REPLACE FUNCTION get_category_sales(
    p_year        INT     DEFAULT 2024,
    p_status      TEXT    DEFAULT 'delivered',
    p_min_revenue NUMERIC DEFAULT 0,
    p_order_by    TEXT    DEFAULT 'revenue'      -- 'revenue' | 'orders' | 'name'
)
RETURNS TABLE (category_name TEXT, revenue NUMERIC, orders_count BIGINT, items_sold BIGINT)
LANGUAGE plpgsql
AS $$
DECLARE
    v_order TEXT;
BEGIN
    -- білий список для ORDER BY: значення параметра не підставляється в SQL напряму
    v_order := CASE p_order_by
                   WHEN 'revenue' THEN 'revenue DESC'
                   WHEN 'orders'  THEN 'orders_count DESC'
                   WHEN 'name'    THEN 'category_name'
               END;
    IF v_order IS NULL THEN
        RAISE EXCEPTION 'Невідомий критерій сортування: %', p_order_by;
    END IF;

    RETURN QUERY EXECUTE format($q$
        SELECT c.category_name::text,
               ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2),
               COUNT(DISTINCT o.order_id),
               SUM(oi.quantity)::bigint
        FROM orders o
        JOIN order_items oi ON o.order_id    = oi.order_id
        JOIN products    p  ON oi.product_id = p.product_id
        JOIN categories  c  ON p.category_id = c.category_id
        WHERE o.order_status = $1
          AND o.order_date >= make_date($2, 1, 1)
          AND o.order_date <  make_date($2 + 1, 1, 1)
        GROUP BY c.category_id, c.category_name
        HAVING SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) >= $3
        ORDER BY %s
    $q$, v_order)
    USING p_status, p_year, p_min_revenue;
END;
$$;

-- Виклик:
SELECT * FROM get_category_sales(2024, 'delivered', 50000, 'revenue');
```

**Результат виконання виклику** `get_category_sales(2024, 'delivered', 50000, 'revenue')`:
```
category_name                | revenue   | orders_count | items_sold
-----------------------------+-----------+--------------+-----------
Ноутбуки та комп'ютери       | 822092.46 | 7            | 20        
Смартфони та телефони        | 266948.99 | 7            | 12        
Телевізори та аудіо          | 217354.53 | 5            | 6         
Побутова техніка             | 77557.75  | 9            | 18        
Планшети та електронні книги | 58388.90  | 5            | 12        
(5 rows)
```

**Пояснення:** Функція приймає рік, статус замовлення, мінімальну виручку та критерій сортування. Значення фільтрів передаються через `USING` як параметри `$1..$3` (захист від SQL-ін'єкцій), а `ORDER BY` підставляється через `format()` лише з білого списку (`revenue`, `orders`, `name`) — інші значення спричиняють помилку.


## Аналіз продуктивності

### Дослідження планів виконання

**Найповільніший запит** (найважчий за кількістю з'єднань: 5 таблиць + агрегація з `COUNT(DISTINCT)` — запит, що лежить в основі матеріалізованого представлення):
```sql
SELECT
    CAST(EXTRACT(YEAR  FROM o.order_date) AS INTEGER) AS year,
    CAST(EXTRACT(MONTH FROM o.order_date) AS INTEGER) AS month,
    c.category_name,
    r.region_name,
    ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2)                                    AS total_revenue,
    COUNT(DISTINCT o.order_id)                              AS orders_count,
    ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) / COUNT(DISTINCT o.order_id), 2)       AS avg_order_value
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products    p  ON oi.product_id = p.product_id
JOIN categories  c  ON p.category_id = c.category_id
JOIN customers   cu ON o.customer_id = cu.customer_id
LEFT JOIN regions r ON cu.region_id = r.region_id
WHERE o.order_status = 'delivered'
GROUP BY 1, 2, c.category_name, r.region_name;
```

**План виконання (EXPLAIN ANALYZE):**
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT
    CAST(EXTRACT(YEAR  FROM o.order_date) AS INTEGER) AS year,
    CAST(EXTRACT(MONTH FROM o.order_date) AS INTEGER) AS month,
    c.category_name,
    r.region_name,
    ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2)                                    AS total_revenue,
    COUNT(DISTINCT o.order_id)                              AS orders_count,
    ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) / COUNT(DISTINCT o.order_id), 2)       AS avg_order_value
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products    p  ON oi.product_id = p.product_id
JOIN categories  c  ON p.category_id = c.category_id
JOIN customers   cu ON o.customer_id = cu.customer_id
LEFT JOIN regions r ON cu.region_id = r.region_id
WHERE o.order_status = 'delivered'
GROUP BY 1, 2, c.category_name, r.region_name;
```
```
-- [ВСТАВТЕ результат EXPLAIN ANALYZE з Supabase SQL Editor / скріншот]
-- Planning Time: [__] ms
-- Execution Time: [__] ms
```

> На що звернути увагу в плані: тип сканування (`Seq Scan` / `Index Scan`), тип з'єднання (`Hash Join` / `Nested Loop`), вузол `HashAggregate`/`GroupAggregate` і `Sort` (для `COUNT(DISTINCT)`), різницю між `rows=` (оцінка) та `actual rows`.
> Оскільки таблиці дуже малі (25–47 рядків), планувальник PostgreSQL, імовірно, обиратиме `Seq Scan` + `Hash Join` навіть за наявності індексів — це нормально: послідовне читання кількох сторінок дешевше за індексний доступ. Ефект індексів стає помітним лише на більших обсягах (див. нижче тестову таблицю `products_big`).

**Запропоновані оптимізації:**
1. **Фільтр за статусом і датою за допомогою складеного індексу** `orders(order_status, order_date)` — замість окремих індексів `idx_orders_status` та `idx_orders_date`, які не можна комбінувати так само ефективно; запит фільтрує `order_status = 'delivered'`, а звіти — ще й за діапазоном дат.
2. **Матеріалізація результату** (`mv_monthly_sales`): важкий запит із 5 з'єднаннями виконується один раз при `REFRESH`, а звіти читають готову агреговану таблицю.
3. **Покриваючий індекс** `order_items(order_id) INCLUDE (product_id, quantity, unit_price, discount)` — дозволяє `Index Only Scan` без звернення до таблиці; а також вибірка лише потрібних стовпців замість `SELECT *` та використання діапазону дат замість функцій над датою у `WHERE` (щоб індекс міг застосовуватись).

### Створені індекси

**Індекс 1:**
```sql
CREATE INDEX idx_orders_status_date ON orders(order_status, order_date);
```
**Обґрунтування:** Запити до матеріалізованого представлення, функція `get_category_sales` та місячні звіти фільтрують замовлення за статусом (`= 'delivered'`) і діапазоном дат. Складений індекс із рівністю на першому стовпці та діапазоном на другому підходить для цього ідеально: індекс (A, B) працює для умов по A та по (A, B).

**Індекс 2:**
```sql
CREATE INDEX idx_products_category_price ON products(category_id, unit_price DESC);
```
**Обґрунтування:** Запити «в межах категорії» (середня ціна категорії, `RANK() OVER (PARTITION BY category_id ORDER BY unit_price DESC)`, корельований підзапит `p2.category_id = p.category_id AND p2.unit_price > p.unit_price`) одночасно групують за `category_id` та сортують/порівнюють за `unit_price`. Такий індекс віддає рядки вже в потрібному порядку і прискорює як віконні функції (немає окремого Sort), так і підзапит (діапазонне сканування індексу).

**Індекс 3 (додатковий, покриваючий):**
```sql
CREATE INDEX idx_order_items_covering ON order_items(order_id)
    INCLUDE (product_id, quantity, unit_price, discount);
```
**Обґрунтування:** майже кожен аналітичний запит читає позиції замовлень разом із кількістю, ціною та знижкою; покриваючий індекс дозволяє виконати Index Only Scan.

**Перевірка після створення індексів** (повторіть EXPLAIN ANALYZE і порівняйте з планом вище):
```
-- [ВСТАВТЕ новий план / Execution Time: [__] ms]
```


## Порівняльний аналіз

### Ефективність різних підходів

**Завдання:** Знайти топ-5 найдорожчих товарів у кожній категорії

**Підхід 1: Віконні функції**
```sql
SELECT category_name, product_name, unit_price, price_rank
FROM (
    SELECT c.category_id, c.category_name, p.product_name, p.unit_price,
           RANK() OVER (PARTITION BY p.category_id ORDER BY p.unit_price DESC) AS price_rank
    FROM products p
    JOIN categories c ON c.category_id = p.category_id
) t
WHERE price_rank <= 5
ORDER BY category_id, price_rank, product_name;
```

**Підхід 2: Корельований підзапит**
```sql
SELECT c.category_name, p.product_name, p.unit_price,
       (SELECT COUNT(*) + 1
        FROM products p2
        WHERE p2.category_id = p.category_id
          AND p2.unit_price > p.unit_price) AS price_rank
FROM products p
JOIN categories c ON c.category_id = p.category_id
WHERE (SELECT COUNT(*)
       FROM products p2
       WHERE p2.category_id = p.category_id
         AND p2.unit_price > p.unit_price) < 5
ORDER BY c.category_id, price_rank, p.product_name;
```

**Результат (обидва підходи повертають однаковий набір, 25 рядків — у кожній з 7 непорожніх категорій не більше 5 товарів):**
```
category_name          | product_name                         | unit_price | price_rank
-----------------------+--------------------------------------+------------+-----------
Смартфони та телефони  | iPhone 15 128GB Чорний               | 29999.00   | 1         
Смартфони та телефони  | Samsung Galaxy S24 256GB Фіолетовий  | 27999.00   | 2         
Смартфони та телефони  | Google Pixel 8 128GB Білий           | 24999.00   | 3         
Смартфони та телефони  | OnePlus 12 256GB Зелений             | 22999.00   | 4         
Смартфони та телефони  | Xiaomi Redmi Note 13 Pro 128GB Синій | 12999.00   | 5         
Ноутбуки та комп'ютери | Dell XPS 13 Plus Intel i7 512GB      | 52999.00   | 1         
Ноутбуки та комп'ютери | Lenovo ThinkPad X1 Carbon Gen 11     | 48999.00   | 2         
Ноутбуки та комп'ютери | MacBook Air M2 13" 256GB Сріблястий  | 45999.00   | 3         
Ноутбуки та комп'ютери | ASUS ROG Strix G15 RTX 4060 1TB      | 42999.00   | 4         
Ноутбуки та комп'ютери | HP Pavilion 15 AMD Ryzen 7 512GB     | 28999.00   | 5         
... показано 10 з 25 рядків
```

**Час виконання** (виміряти через `EXPLAIN (ANALYZE)` у Supabase):
- Віконні функції: [__] ms
- Корельований підзапит: [__] ms

**Порівняння на більшому обсязі** (на 25 товарах різниця майже непомітна, тому створюємо тестову таблицю на 20 000 рядків):
```sql
-- Тестова таблиця з 20 000 товарів (10 категорій) для порівняння підходів на більшому обсязі
DROP TABLE IF EXISTS products_big;
CREATE TABLE products_big AS
SELECT g AS product_id,
       (g % 10) + 1 AS category_id,
       'Товар ' || g AS product_name,
       ROUND((random() * 100000)::numeric, 2) AS unit_price
FROM generate_series(1, 20000) AS g;
ANALYZE products_big;

-- Віконні функції
EXPLAIN (ANALYZE, BUFFERS)
SELECT category_id, product_name, unit_price
FROM (
    SELECT category_id, product_name, unit_price,
           RANK() OVER (PARTITION BY category_id ORDER BY unit_price DESC) AS rnk
    FROM products_big
) t
WHERE rnk <= 5;

-- Корельований підзапит
EXPLAIN (ANALYZE, BUFFERS)
SELECT p.category_id, p.product_name, p.unit_price
FROM products_big p
WHERE (SELECT COUNT(*) FROM products_big p2
       WHERE p2.category_id = p.category_id
         AND p2.unit_price > p.unit_price) < 5;
```
- Віконні функції (20 000 рядків): [__] ms
- Корельований підзапит (20 000 рядків): [__] ms

**Висновок:** Віконна функція проходить таблицю один раз (WindowAgg після сортування/індексного сканування за `category_id, unit_price DESC`), тому її вартість зростає приблизно як N·log N. Корельований підзапит виконується окремо для кожного рядка зовнішньої таблиці (вузол SubPlan), тому без індексу його вартість зростає квадратично; індекс `(category_id, unit_price DESC)` суттєво зменшує навантаження, але віконні функції все одно залишаються ефективнішими та читабельнішими. На навчальних даних (25 товарів) різниця — соті частки мілісекунди і непомітна. Для еквівалентності результатів у підході 1 використано `RANK()` (а не `ROW_NUMBER()`), бо підзапит рахує кількість товарів з вищою ціною — як і RANK, він залишає всі товари з однаковою ціною.


## Висновки

**Самооцінка**: 5

**Обгрунтування**: У лабораторній роботі виконано завдання всіх трьох рівнів на базі даних «ТехноМарт». Рівень 1: написано запити з INNER/LEFT JOIN та множинним з'єднанням 7 таблиць, агрегатні запити з COUNT/SUM/AVG/MIN/MAX, GROUP BY і HAVING, підзапити (корельований, IN, EXISTS/NOT EXISTS, скалярний у SELECT). Рівень 2: використано RIGHT та FULL OUTER JOIN, self-join, з'єднання з умовою в ON, віконні функції RANK/DENSE_RANK/ROW_NUMBER, LAG/LEAD і агрегати з PARTITION BY. Рівень 3: створено матеріалізоване представлення з індексами, рекурсивний CTE для ієрархії співробітників та параметризовану функцію з динамічним SQL (захищеним білим списком і параметрами USING). Також досліджено плани виконання, створено складені та покриваючий індекси, порівняно віконні функції з корельованим підзапитом. Я розумію різницю між типами з'єднань, між WHERE та HAVING, між RANK і DENSE_RANK, принцип роботи корельованих підзапитів і причини, через які вони повільніші за віконні функції на великих обсягах.
