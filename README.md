## Descripción Prueba Técnica
El desarrollo de una solución que permita extraer y procesar datos para alimentar un dashboard visual que refleje los indicadores clave planteados en el desafío. Se prioriza la optimización del rendimiento del sistema, evitando consultas en tiempo real sobre la base de datos operativa para garantizar una experiencia de usuario fluida y eficiente.
<br>
![Arquitectura de la base de datos](/images/arquitectura_bbdd.JPG)

## KPIs
Con la información descrita, se plantea obtener tablas para obteners los siguientes indicadores clave:
1) **Cantidad de Ventas y Productos Más Vendidos:** Visualizar el número total de
ventas realizadas y destacar los productos con mayor demanda.
2) **Monto Facturado y Estado de los Pagos:** Monitorear el monto total facturado,
junto con el desglose de los pagos y sus respectivos estados.
3) **Comparativa de Ventas vs. Pagos Recibidos:** Comparar el monto total de las
ventas con el monto efectivamente pagado por los clientes.
4) **Ranking de Usuarios por Ventas:** Determinar qué usuarios han generado la mayor
cantidad de ventas.
5) **Detalle de Ventas y Facturas:** Proveer un desglose detallado de cada venta y sus
correspondientes facturas.

## Enfoque Metodológico
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

![Arquitectura de la base de datos](/images/data_mart.JPG)

<br>
Para el data mart, se deben ejecutar las siguientes query:

1) Crear la tabla <em>dm_sale_order<em>
```
/*dm_sale_order*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_sale_order` () as
SELECT order_id, user_id, order_date, status, total_amount, invoice_id
FROM `<proyect_input>.<dataset_input>.SALE_ORDER`
```

2) Crear la tabla <em>dm_invoice<em>
```
/*dm_invoice*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_invoice` () as
SELECT invoice_id, invoice_date, total_amount, due_date, status_invoice, product_id, payment_id
FROM `<proyect_input>.<dataset_input>.INVOICE`
```

3) Crear la tabla <em>dm_payment<em>
```
/*dm_payment*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_payment` () as
SELECT invoice_id, amount, status_payment
FROM `<proyect_input>.<dataset_input>.PAYMENT`
```

4) Crear la tabla <em>dm_customer<em>
```
/*dm_customer*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_customer` () as
SELECT customer_id, name
FROM `<proyect_input>.<dataset_input>.CUSTOMER`
```

5) Crear la tabla <em>dm_product<em>
```
/*dm_product*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_product` () as
SELECT customer_id, name
FROM `<proyect_input>.<dataset_input>.PRODUCT`
```

