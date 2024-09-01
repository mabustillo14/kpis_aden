## Descripción Prueba Técnica
El desarrollo de una solución que permita extraer y procesar datos para alimentar un dashboard visual que refleje los indicadores clave planteados en el desafío. Se prioriza la optimización del rendimiento del sistema, evitando consultas en tiempo real sobre la base de datos operativa para garantizar una experiencia de usuario fluida y eficiente.
<br>
![Arquitectura de la base de datos](/images/arquitectura_bbdd.JPG)

A continuación, se describen las principales entidades y sus relaciones:

- **Clientes:** Se registran los datos básicos de cada cliente, como nombre, teléfono, dirección y país.
- **Usuarios:** Los empleados que interactúan con el sistema están registrados como usuarios. Cada usuario tiene un nombre, correo electrónico y un rol específico (por ejemplo, vendedor o administrador).
- **Órdenes de Venta:** Los usuarios pueden generar órdenes de venta en el sistema. Cada orden de venta incluye la fecha de creación, el estado (pendiente, completada, cancelada) y el monto total.
- **Productos:** El sistema permite gestionar los productos, cada uno con un nombre, un precio y una asignación a una categoría específica.
- **Categorías de Productos:** Los productos están organizados en categorías. Cada categoría posee un nombre y una descripción.
- **Facturación:** Por cada orden de venta se genera una factura, la cual incluye la fecha de emisión, el monto total, la fecha de vencimiento y el estado de la factura (pagada, pendiente, vencida).


## <br>Enfoque Metodológico
Consideraciones previas para la resolución:
- Las consultas están diseñadas para ejecutarse en Google Cloud Platform, considerando la sintaxis específica de este entorno.
- Todas los datasets proporcionados están almacenadas dentro de la base de datos `<dataset_input>` del proyecto denominado `<proyect_input>` .
- Se ha utilizado la base de datos pública Northwind como referencia para ilustrar los resultados esperados, simulando situaciones en las que se cuenta con datos disponibles.
- Todas las soluciones se han implementado utilizando SQL.

## <br>DATA MART ⚙️
Para poder inicializar la arquitectura del Data Mart para responder los KPIs planteados, se consideraron los siguientes aspectos:
- Los dataset estarán alojadas en la base de datos `<dataset>` dentro del proyecto `<data_mart>`.
- Se considero un _esquema de copo de nieve_ para el Data Mart:

<br>

![Arquitectura de la base de datos](/images/data_mart_diagram.JPG)

<br>
### Querys para el Data Mart
Para crear el data mart, se deben ejecutar las siguientes query:

1) Crear la tabla <em>dm_sale_order<em>
```
/*dm_sale_order*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_sale_order` as
SELECT
  `order_id`,
  `user_id`,
  `order_date`,
  `status`,
  `total_amount`,
  `invoice_id`,
FROM `<proyect_input>.<dataset_input>.SALE_ORDER`
```

2) Crear la tabla <em>dm_invoice<em>
```
/*dm_invoice*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_invoice` as
SELECT
  `invoice_id`,
  `invoice_date`,
  `total_amount`,
  `due_date`,
  `status_invoice`,
  `product_id`,
  `payment_id`,
FROM `<proyect_input>.<dataset_input>.INVOICE`
```

3) Crear la tabla <em>dm_payment<em>
```
/*dm_payment*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_payment` as
SELECT
  `invoice_id`,
  `amount`,
  `status_payment`
FROM `<proyect_input>.<dataset_input>.PAYMENT`
```

4) Crear la tabla <em>dm_customer<em>
```
/*dm_customer*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_customer` as
SELECT
  `customer_id`,
  `name`,
FROM `<proyect_input>.<dataset_input>.CUSTOMER`
```

5) Crear la tabla <em>dm_product<em>
```
/*dm_product*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_product` as
SELECT customer_id, name
FROM `<proyect_input>.<dataset_input>.PRODUCT`
```

## <br> KPIs 
Los dataset estarán alojadas en la base de datos `<dataset>` dentro del proyecto `<data_kpis>`. Con la información descrita, se plantea obtener tablas para los siguientes indicadores clave:

1) **Cantidad de Ventas y Productos Más Vendidos:** Visualizar el número total de ventas realizadas y destacar los productos con mayor demanda.

```
/*sales_products*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.sales_products` as
SELECT 
    p.product_id,
    p.name AS product_name,
    COUNT(so.order_id) AS total_sales,
    SUM(so.total_amount) AS total_sales_amount
FROM `<data_mart>.<dataset>.dm_sale_order` so
INNER JOIN `<data_mart>.<dataset>.dm_invoice` i ON i.order_id = so.order_id
INNER JOIN `<data_mart>.<dataset>.dm_product` p ON p.product_id = i.product_id
GROUP BY p.product_id
HAVING so.status = 'completada'
ORDER BY total_sales_amount desc, total_sales desc
```

2) **Monto Facturado y Estado de los Pagos:** Monitorear el monto total facturado, junto con el desglose de los pagos y sus respectivos estados.

```
/*invoice_payments*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.invoice_payments` as
SELECT 
  i.invoice_id,
  i.status_invoice, 
  SUM(i.total_amount) AS total_invoice,
  SUM(IFNULL(p.amount, 0)) AS total_paid
FROM 
    `<data_mart>.<dataset>.dm_invoice` i
LEFT JOIN 
    `<data_mart>.<dataset>.dm_payment` p 
ON 
    i.invoice_id = p.invoice_id
GROUP BY 
    i.status_invoice;
```


3) **Comparativa de Ventas vs. Pagos Recibidos:** Comparar el monto total de las ventas con el monto efectivamente pagado por los clientes.



4) **Ranking de Usuarios por Ventas:** Determinar qué usuarios han generado la mayor cantidad de ventas.

```
/*user_sales_ranking*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.user_sales_ranking` as
SELECT 
    c.user_id,
    c.name AS user_name,
    COALESCE(COUNT(so.order_id),0) AS total_sales,
    COALESCE(SUM(so.total_amount),0) AS total_revenue
FROM
  `<data_mart>.<dataset>.dm_customer` c
LEFT JOIN
  sale_order so ON c.user_id = so.user_id
GROUP BY c.user_id
ORDER BY COALESCE(SUM(so.total_amount),0) desc
```


5) **Detalle de Ventas y Facturas:** Proveer un desglose detallado de cada venta y sus correspondientes facturas.
```
/*sales_invoices_detail*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.sales_invoices_detail` as
SELECT 
  so.order_id, 
  so.order_date,
  so.status as order_status 
  so.total_amount as order_total_amount, 
  i.invoice_id, 
  i.invoice_date,
  i.due_date,
  i.total_amount AS invoice_total_amount,
  i.status_invoice,
FROM 
    `<data_mart>.<dataset>.dm_sale_order` so
JOIN 
    `<data_mart>.<dataset>.dm_invoice` i 
ON 
    so.order_id = i.order_id;
```

