## 🙌 Challenge Técnico
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
- **Pagos:** Los clientes pueden realizar pagos parciales o totales para una factura específica. Cada pago registra la fecha, el monto, el método de pago (como tarjeta de crédito, transferencia bancaria), y el estado (pagado, fallido, pendiente).


## <br>🔧 Enfoque Metodológico 
- Las consultas están diseñadas para ejecutarse en **Google Cloud Platform > BigQuery**, considerando la sintaxis específica de este entorno.
- Se considera que todos los datasets proporcionados están almacenadas dentro de la base de datos `<dataset_input>` del proyecto denominado `<proyect_input>` .
- En algunos casos se ha utilizado la **base de datos pública Northwind** como referencia para ilustrar los resultados esperados, simulando situaciones en las que se cuenta con datos disponibles.
- Todas las soluciones se han implementado utilizando **SQL**.

## <br>💡 DATA MART 
- Los dataset estarán alojadas en la base de datos `<dataset>` dentro del proyecto `<data_mart>`.
- Se considero un ***esquema de copo de nieve*** para la arquitectura del Data Mart:

<br>

![Arquitectura de la base de datos](/images/data_mart_diagram.JPG)

<br>

Para crear el data mart, se deben ejecutar las siguientes query:

1) Crear la tabla ***dm_sale_order***
```
/*dm_sale_order*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_sale_order` as
SELECT
  order.order_id,
  order.user_id,
  order.order_date,
  order.status as order_status,
  order.total_amount,
  order.invoice_id
FROM `<proyect_input>.<dataset_input>.SALE_ORDER` AS order
```

2) Crear la tabla ***dm_invoice***
```
/*dm_invoice*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_invoice` as
SELECT
  invoice.invoice_id,
  invoice.invoice_date,
  invoice.total_amount,
  invoice.due_date,
  invoice.status_invoice,
  invoice.product_id,
  invoice.payment_id,
FROM `<proyect_input>.<dataset_input>.INVOICE` AS invoice
```

3) Crear la tabla ***dm_payment***
```
/*dm_payment*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_payment` as
SELECT
  payment.invoice_id,
  payment.amount_paid,
  payment.status_payment
FROM `<proyect_input>.<dataset_input>.PAYMENT` AS payment
```

4) Crear la tabla ***dm_customer***
```
/*dm_customer*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_customer` as
SELECT
  customer.customer_id,
  customer.name as customer_name,
FROM `<proyect_input>.<dataset_input>.CUSTOMER` AS customer
```

5) Crear la tabla ***dm_product***
```
/*dm_product*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_product` as
SELECT
  product.product_id,
  product.name as product_name
FROM `<proyect_input>.<dataset_input>.PRODUCT` AS product
```

## <br>📊 KPIs 
Los dataset estarán alojadas en la base de datos `<dataset>` dentro del proyecto `<data_kpis>`. Con la información descrita, se plantea obtener tablas para los siguientes indicadores clave:

1) **Cantidad de Ventas y Productos Más Vendidos:** Visualizar el número total de ventas realizadas y destacar los productos con mayor demanda.

```
/*sales_products*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.sales_products` as
SELECT 
    dmproduct.product_id,
    dmproduct.product_name,
    COUNT(dmsaleorder.order_id) AS total_sales,
    SUM(dmsaleorder.total_amount) AS total_sales_amount
FROM
  `<data_mart>.<dataset>.dm_sale_order` AS dmsaleorder
INNER JOIN
  `<data_mart>.<dataset>.dm_invoice` AS dminvoice ON dminvoice.order_id = dmsaleorder.order_id
INNER JOIN
  `<data_mart>.<dataset>.dm_product` AS dmproduct ON dmproduct.product_id = dminvoice.product_id
GROUP BY dmproduct.product_id
HAVING dmsaleorder.order_status = 'completada'
ORDER BY SUM(dmsaleorder.total_amount) desc, COUNT(dmsaleorder.order_id) desc
```
 Output esperado: <br>
 ![output_sales_products](/images/output_sales_products.JPG)


2) **Monto Facturado y Estado de los Pagos:** Monitorear el monto total facturado, junto con el desglose de los pagos y sus respectivos estados.

```
/*invoice_payments*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.invoice_payments` as
SELECT 
    dmsaleorder.order_id,
    dminvoice.invoice_id
    dmsaleorder.total_amount AS total_sales_amount,
    SUM(IFNULL(dmpayment.amount_paid, 0)) AS total_paid_amount,
    dmsaleorder.total_amount - SUM(IFNULL(dmpayment.amount, 0)) AS unpaid_amount,
    dminvoice.status_invoice AS invoice_status
FROM
  `<data_mart>.<dataset>.dm_sale_order` AS dmsaleorder
LEFT JOIN
  `<data_mart>.<dataset>.dm_invoice` AS dminvoice ON dmsaleorder.order_id = dminvoice.order_id
LEFT JOIN
  `<data_mart>.<dataset>.dm_payment` AS dmpayment ON dminvoice.invoice_id = dmpayment.invoice_id
GROUP BY dmsaleorder.order_id
```


3) **Comparativa de Ventas vs. Pagos Recibidos:** Comparar el monto total de las ventas con el monto efectivamente pagado por los clientes.

```
/*sales_payments*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.sales_payments` as
SELECT 
  dminvoice.invoice_id,
  dminvoice.status_invoice, 
  SUM(dminvoice.total_amount) AS total_invoice,
  SUM(IFNULL(dmpayment.amount_paid, 0)) AS total_paid
FROM 
    `<data_mart>.<dataset>.dm_invoice` AS dminvoice
LEFT JOIN 
    `<data_mart>.<dataset>.dm_payment` AS dmpayment ON dminvoice.invoice_id = dmpayment.invoice_id
GROUP BY 
    dminvoice.status_invoice;
```

4) **Ranking de Usuarios por Ventas:** Determinar qué usuarios han generado la mayor cantidad de ventas.

```
/*user_sales_ranking*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.user_sales_ranking` as
SELECT 
    dmcustoner.user_id,
    dmcustoner.customer_name,
    SUM(IFNULL(dmsaleorder.order_id, 0)) AS total_sales,
    SUM(IFNULL(dmsaleorder.total_amount, 0)) AS total_revenue
FROM
  `<data_mart>.<dataset>.dm_customer` AS dmcustoner
LEFT JOIN
  `<data_mart>.<dataset>.dm_sale_order` AS dmsaleorder ON dmcustoner.user_id = dmsaleorder.user_id
GROUP BY dmcustoner.user_id
ORDER BY SUM(IFNULL(dmsaleorder.total_amount, 0)) desc
```

Output esperado: <br>
 ![output_sales_products](/images/output_user_sales_ranking.JPG)


5) **Detalle de Ventas y Facturas:** Proveer un desglose detallado de cada venta y sus correspondientes facturas.
```
/*sales_invoices_detail*/
CREATE OR REPLACE TABLE `<data_kpis>.<dataset>.sales_invoices_detail` as
SELECT 
  dmsaleorder.order_id, 
  dmsaleorder.order_date,
  dmsaleorder.status as order_status 
  dmsaleorder.total_amount as order_total_amount, 
  dminvoice.invoice_id, 
  dminvoice.invoice_date,
  dminvoice.due_date,
  dminvoice.total_amount AS invoice_total_amount,
  dminvoice.status_invoice,
FROM 
    `<data_mart>.<dataset>.dm_sale_order` AS dmsaleorder
JOIN 
    `<data_mart>.<dataset>.dm_invoice` AS dminvoice ON dmsaleorder.order_id = dminvoice.order_id;
```

## Documentación

Todos los detalles de cada uno de los parámetros que devuelven las tablas para resolver los KPIs se encuentran la [documentación](https://github.com/mabustillo14/kpis_aden/blob/main/Documentacion.pdf)

