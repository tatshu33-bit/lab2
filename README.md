-- Лабораторна робота 2. Кондратюк Дмитро Михайлович, ІПЗ-33
-- Усі запити для виконання в Supabase SQL Editor (PostgreSQL)

-- ===== Завдання 1.1 =====
SELECT p.product_name, c.category_name, s.company_name AS supplier, p.unit_price
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
INNER JOIN suppliers s  ON p.supplier_id = s.supplier_id
ORDER BY c.category_id, p.product_id;

-- ===== Завдання 1.2 =====
SELECT c.contact_name, c.customer_type, r.region_name,
       COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o  ON c.customer_id = o.customer_id
LEFT JOIN regions r ON c.region_id = r.region_id
GROUP BY c.customer_id, c.contact_name, c.customer_type, r.region_name
ORDER BY order_count DESC, c.customer_id;

-- ===== Завдання 1.2b =====
SELECT 'INNER JOIN' AS join_type, COUNT(DISTINCT c.category_id) AS categories_in_result
FROM categories c
INNER JOIN products p ON p.category_id = c.category_id
UNION ALL
SELECT 'LEFT JOIN', COUNT(DISTINCT c.category_id)
FROM categories c
LEFT JOIN products p ON p.category_id = c.category_id;

-- ===== Завдання 1.3 =====
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

-- ===== Завдання 2.1 =====
SELECT c.category_name,
       COUNT(p.product_id)          AS product_count,
       ROUND(AVG(p.unit_price), 2)  AS avg_price,
       MIN(p.unit_price)            AS min_price,
       MAX(p.unit_price)            AS max_price
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_id, c.category_name
ORDER BY product_count DESC, c.category_id;

-- ===== Завдання 2.2 =====
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

-- ===== Завдання 2.3 =====
SELECT s.company_name,
       COUNT(p.product_id)         AS product_count,
       ROUND(AVG(p.unit_price), 2) AS avg_price
FROM suppliers s
JOIN products p ON s.supplier_id = p.supplier_id
GROUP BY s.supplier_id, s.company_name
HAVING COUNT(p.product_id) > 2
ORDER BY product_count DESC, s.supplier_id;

-- ===== Завдання 3.1 =====
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

-- ===== Завдання 3.2 =====
SELECT c.customer_id, c.contact_name, c.customer_type, c.city
FROM customers c
WHERE c.customer_id IN (
    SELECT o.customer_id
    FROM orders o
    WHERE o.order_date >= '2024-01-01'
      AND o.order_date <  '2025-01-01'
)
ORDER BY c.customer_id;

-- ===== Завдання 3.2b =====
-- Варіант з EXISTS: клієнти, які мають відкриті замовлення (не delivered)
SELECT c.customer_id, c.contact_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.order_status <> 'delivered'
)
ORDER BY c.customer_id;

-- ===== Завдання 3.2c =====
-- NOT EXISTS: товари, яких жодного разу не було в замовленнях
SELECT p.product_id, p.product_name, p.unit_price
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.product_id
)
ORDER BY p.product_id;

-- ===== Завдання 3.3 =====
SELECT p.product_id, p.product_name, p.unit_price,
       COALESCE((SELECT SUM(oi.quantity)
                 FROM order_items oi
                 WHERE oi.product_id = p.product_id), 0) AS total_sold
FROM products p
ORDER BY total_sold DESC, p.product_id;

-- ===== Завдання 4.1 =====
SELECT c.category_name,
       COUNT(p.product_id)               AS products_count,
       COALESCE(ROUND(AVG(p.unit_price), 2), 0) AS avg_price
FROM products p
RIGHT JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY products_count DESC, c.category_id;

-- ===== Завдання 4.1b =====
-- FULL OUTER JOIN: категорії без товарів та товари без категорії в одному результаті
SELECT c.category_name, p.product_name
FROM categories c
FULL OUTER JOIN products p ON c.category_id = p.category_id
WHERE c.category_id IS NULL OR p.product_id IS NULL;

-- ===== Завдання 4.2 =====
SELECT e1.first_name || ' ' || e1.last_name AS employee,
       e1.title                              AS employee_title,
       e2.first_name || ' ' || e2.last_name AS manager,
       e2.title                              AS manager_title
FROM employees e1
LEFT JOIN employees e2 ON e1.reports_to = e2.employee_id
ORDER BY e1.employee_id;

-- ===== Завдання 4.3 =====
-- Умовне з'єднання: додаткова умова в ON зберігає всіх клієнтів
SELECT c.contact_name,
       COUNT(o.order_id) AS open_orders
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
                  AND o.order_status IN ('pending', 'processing', 'shipped')
GROUP BY c.customer_id, c.contact_name
ORDER BY open_orders DESC, c.customer_id;

-- ===== Завдання 5.1 =====
SELECT p.product_name,
       c.category_name,
       p.unit_price,
       RANK()       OVER (PARTITION BY c.category_id ORDER BY p.unit_price DESC) AS price_rank,
       DENSE_RANK() OVER (PARTITION BY c.category_id ORDER BY p.unit_price DESC) AS price_dense_rank,
       ROW_NUMBER() OVER (PARTITION BY c.category_id ORDER BY p.unit_price DESC) AS row_num
FROM products p
JOIN categories c ON p.category_id = c.category_id
ORDER BY c.category_id, p.unit_price DESC;

-- ===== Завдання 5.1b =====
-- Ранжування по всьому каталогу: видно різницю RANK / DENSE_RANK на однакових цінах
SELECT product_name, unit_price,
       ROW_NUMBER() OVER (ORDER BY unit_price DESC, product_id) AS row_num,
       RANK()       OVER (ORDER BY unit_price DESC)             AS rank_num,
       DENSE_RANK() OVER (ORDER BY unit_price DESC)             AS dense_rank_num
FROM products
ORDER BY unit_price DESC, product_id
LIMIT 12;

-- ===== Завдання 5.2 =====
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

-- ===== Завдання 5.3 =====
SELECT c.category_name,
       ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2) AS category_revenue,
       ROUND(100.0 * SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) / SUM(SUM(oi.quantity * oi.unit_price * (1 - oi.discount))) OVER (), 2) AS pct_of_total,
       RANK() OVER (ORDER BY SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) DESC) AS revenue_rank
FROM order_items oi
JOIN products   p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY revenue_rank;

-- ===== 6.1 Матеріалізоване представлення =====
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

SELECT * FROM mv_monthly_sales
ORDER BY year, month, total_revenue DESC;

SELECT year, month, SUM(total_revenue) AS revenue
FROM mv_monthly_sales
GROUP BY year, month
ORDER BY year, month;

-- ===== 6.2 Рекурсивний CTE =====
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

-- ===== 6.3 Функція з динамічним SQL =====
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

-- ===== Індекси =====
-- Індекс 1: фільтрація за статусом і діапазоном дат (MV, звіти, функція get_category_sales)
CREATE INDEX idx_orders_status_date ON orders(order_status, order_date);

-- Індекс 2: рейтинги/підзапити "в межах категорії" (RANK, корельований підзапит)
CREATE INDEX idx_products_category_price ON products(category_id, unit_price DESC);

-- Індекс 3 (покриваючий): index-only scan по позиціях замовлення
CREATE INDEX idx_order_items_covering ON order_items(order_id) INCLUDE (product_id, quantity, unit_price, discount);

-- ===== Аналіз продуктивності =====
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

-- ===== Порівняльний аналіз =====
SELECT category_name, product_name, unit_price, price_rank
FROM (
    SELECT c.category_id, c.category_name, p.product_name, p.unit_price,
           RANK() OVER (PARTITION BY p.category_id ORDER BY p.unit_price DESC) AS price_rank
    FROM products p
    JOIN categories c ON c.category_id = p.category_id
) t
WHERE price_rank <= 5
ORDER BY category_id, price_rank, product_name;

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

-- Тестова таблиця з 20 000 товарів (10 категорій) для порівняння підходів на більшому обсязі
DROP TABLE IF EXISTS products_big;
CREATE TABLE products_big AS
SELECT g AS product_id,
       (g % 10) + 1 AS category_id,
       'Товар ' || g AS product_name,
       ROUND((random() * 100000)::numeric, 2) AS unit_price
FROM generate_series(1, 20000) AS g;
ANALYZE products_big;

EXPLAIN (ANALYZE, BUFFERS)
SELECT category_id, product_name, unit_price
FROM (
    SELECT category_id, product_name, unit_price,
           RANK() OVER (PARTITION BY category_id ORDER BY unit_price DESC) AS rnk
    FROM products_big
) t
WHERE rnk <= 5;;

EXPLAIN (ANALYZE, BUFFERS)
SELECT p.category_id, p.product_name, p.unit_price
FROM products_big p
WHERE (SELECT COUNT(*) FROM products_big p2
       WHERE p2.category_id = p.category_id
         AND p2.unit_price > p.unit_price) < 5;;
