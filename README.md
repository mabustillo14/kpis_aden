## Descripción Prueba Técnica
El desarrollo de una solución que permita extraer y procesar datos para alimentar un dashboard visual que refleje los indicadores clave planteados en el desafío. Se prioriza la optimización del rendimiento del sistema, evitando consultas en tiempo real sobre la base de datos operativa para garantizar una experiencia de usuario fluida y eficiente.
![Arquitectura de la base de datos](/images/arquitectura_bbdd.JPG)

## Enfoque Metodológico
Consideraciones previas para la resolución:
- Las consultas están diseñadas para ejecutarse en Google Cloud Platform, considerando la sintaxis específica de este entorno.
- Todas las tablas necesarias están almacenadas en el proyecto denominado `<proyect_input>` dentro de la base de datos `<dataset_input>`.
- Se ha utilizado la base de datos pública Northwind como referencia para ilustrar los resultados esperados, simulando situaciones en las que se cuenta con datos disponibles.
- Todas las soluciones se han implementado utilizando SQL.
