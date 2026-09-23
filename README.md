# 📊 Data Engineering & Analytics End-to-End: Análisis de Exposición Cambiaria

Este proyecto presenta una solución integral de datos (End-to-End) diseñada para cuantificar el impacto de la volatilidad del tipo de cambio (USD/PEN) en los márgenes de ganancia de una empresa importadora. 

A través de una arquitectura Medallón construida en **Azure Databricks** y un modelado analítico en **Power BI**, el pipeline integra datos de ventas internas con la API pública del Banco Central de Reserva del Perú (BCRP), aislando el efecto cambiario del rendimiento comercial puro.

## 🛠️ Stack Tecnológico
* **Orquestación e Ingesta:** Azure Data Factory (ADF), BCRP API.
* **Procesamiento (ETL):** Apache Spark (PySpark), Databricks Serverless.
* **Almacenamiento:** Delta Lake (Arquitectura Medallón: Bronce, Plata, Oro).
* **Visualización & BI:** Power BI, DAX (Time Intelligence, Modelado en Estrella).
* **Control de Versiones:** Git / GitHub.

---

## 🏗️ Arquitectura de Datos (Medallion Architecture)

El procesamiento de datos masivos (~200k registros de ventas) se estructuró en tres capas dentro de Databricks para garantizar la calidad, trazabilidad y optimización del modelo analítico.

### 🥉 Capa Bronce (Raw Data)
* **Ingesta de Ventas:** Almacenamiento de registros transaccionales crudos.
* **Ingesta de Tipo de Cambio (BCRP):** Consumo del histórico de cotizaciones diarias (compra/venta) directamente desde la API oficial del estado peruano.

### 🥈 Capa Plata (Cleansed & Conformed)
* **Data Quality & Cuarentena:** Implementación de reglas de validación (precios nulos, cantidades negativas). Los registros limpios avanzan, mientras que las anomalías se derivan a una tabla de cuarentena (`cuarentena_ventas`) para su posterior auditoría.
* **Deduplicación:** Uso de funciones de ventana (`Window.partitionBy`) en PySpark para garantizar la unicidad de las transacciones.
* **Tratamiento de Series Temporales:** Resolución de "huecos" en los datos del BCRP (fines de semana y feriados) mediante la técnica de *Forward Fill* (`last(ignorenulls=True)`) sobre un calendario continuo generado dinámicamente.

### 🥇 Capa Oro (Business-Ready)
* **Modelado Dimensional (Star Schema):** Construcción de dimensiones optimizadas (`dim_tiempo`, `dim_cliente`, `dim_producto`) y una tabla de hechos (`fct_ventas`).
* **Congelamiento de Tipo de Cambio:** Cruce determinista (*Broadcast Join*) entre las ventas y el tipo de cambio del día exacto de la operación para fijar el costo histórico de importación.

---

## 📈 Capa Analítica y Dashboard (Power BI)

El modelo en estrella fue conectado mediante DirectQuery/Import a un **Databricks Serverless SQL Warehouse**. Se desarrollaron medidas DAX avanzadas para crear un escenario de simulación que compara el margen real obtenido frente al margen que se habría logrado si el dólar se hubiera mantenido estable.

### 1. Resumen de Exposición Cambiaria
Visualización de la "brecha" de pérdida de margen generada exclusivamente por el encarecimiento del dólar.
![Dashboard Resumen]([INSERTA_AQUI_TU_ENLACE_A_LA_IMAGEN_1])

### 2. Análisis Cambiario Mensual
Desglose del impacto mes a mes, correlacionando los picos del tipo de cambio con los meses de mayor pérdida, aislando el factor comercial mediante formato condicional dinámico.
![Dashboard Análisis]([INSERTA_AQUI_TU_ENLACE_A_LA_IMAGEN_2])

### 3. Detalle Comercial (Rendimiento por Canal y Categoría)
Análisis de rentabilidad que demuestra que las líneas de negocio con mayor costo en dólares (Cómputo) son las más expuestas, incluyendo un ranking de clientes por margen aportado.
![Dashboard Detalle]([INSERTA_AQUI_TU_ENLACE_A_LA_IMAGEN_3])

---

## 👨‍💻 Autor
**Jhon Vilcarana Tintaya**  
*Data Engineer & Data Analyst*  
Especializado en el diseño de arquitecturas cloud, modelado dimensional y desarrollo de pipelines escalables con PySpark, SQL y herramientas modernas de BI.
