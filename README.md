## Descripción Prueba Técnica
El desarrollo de una solución que permita extraer y procesar datos para alimentar un dashboard visual que refleje los indicadores clave planteados en el desafío. Se prioriza la optimización del rendimiento del sistema, evitando consultas en tiempo real sobre la base de datos operativa para garantizar una experiencia de usuario fluida y eficiente.
<br>
![Arquitectura de la base de datos](/images/arquitectura_bbdd.JPG)

## Enfoque Metodológico
Consideraciones previas para la resolución:
- Las consultas están diseñadas para ejecutarse en Google Cloud Platform, considerando la sintaxis específica de este entorno.
- Todas los datasets proporcionados están almacenadas dentro de la base de datos `<dataset_input>` del proyecto denominado `<proyect_input>` .
- Se ha utilizado la base de datos pública Northwind como referencia para ilustrar los resultados esperados, simulando situaciones en las que se cuenta con datos disponibles.
- Todas las soluciones se han implementado utilizando SQL.

# <br>DATA MART ⚙️
Para poder inicializar la arquitectura del Data Mart para resolver, se consideraron los siguientes aspectos:
- Los dataset estarán alojadas en la base de datos `<dataset>` dentro del proyecto `<data_mart>`.
- Se considero un _esquema de estrella_ para el Data Mart: 

<br>
Para construirlo, se deben ejecutar las siguientes query:

1) Crear la tabla <em>dm_sale_order<em>
```
/*dm_sale_order*/
CREATE OR REPLACE TABLE `<data_mart>.<dataset>.dm_sale_order` () as
SELECT order_id, user_id, order_date, status, total_amount, invoice_id
FROM `<proyect_input>.<dataset_input>.dm_sale_order`
```
