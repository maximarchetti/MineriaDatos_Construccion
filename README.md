1. Descripción general del proyecto

Este proyecto implementa una arquitectura completa de procesamiento de datos en la nube utilizando Microsoft Azure para analizar indicadores oficiales del sector de la construcción en Argentina.

El objetivo es evaluar si:

es posible predecir el valor del índice en determinada fecha o provincia, y

existe correlación real entre las distintas fuentes INDEC, CAC y AFCP.

Para ello se utilizan servicios cloud para ingesta, almacenamiento, transformación y modelado.

2. Dataset utilizado

El dataset real utilizado es:

construccion.csv
Fuente oficial: Datos Argentina
Dataset: Indicadores de la construcción en valores anuales, trimestrales y mensuales
URL referencia: https://datos.gob.ar/

Formato original: CSV (codificación Windows-1252)
Cantidad de registros: ~14.000 filas

Principales columnas:

sector_id, sector_nombre

variable_id

actividad_producto_nombre

indicador

unidad_de_medida

fuente (INDEC, CAC, AFCP)

nombre_frecuencia

cobertura_nombre

alcance_tipo, alcance_id, alcance_nombre

indice_tiempo (fecha)

valor (numérico)

3. Arquitectura de la solución

La arquitectura del proyecto se implementó íntegramente en Azure.

1. Ingesta – Azure Data Factory (ADF)

Se creó un pipeline con:

Linked Service hacia Azure Blob Storage

Linked Service hacia Azure SQL Database

Dataset CSV (source)

Dataset SQL (sink)

Actividad Copy Data

Se cargaron 14.032 filas del CSV hacia Azure SQL Database.

2. Almacenamiento – Azure SQL Database

Tabla destino: dbo.Construccion

Ajustes realizados:

Corrección de tipos

valor convertido a FLOAT

indice_tiempo convertido a DATE

No se realizaron transformaciones en SQL; todo el preprocesamiento se hizo en Databricks.

3. Transformación y Modelado – Azure Databricks

Uso de un Notebook Python para:

Cargar datos vía JDBC

Limpieza:

eliminar columnas sin variación

renombrar campos

corregir codificaciones

eliminar nulos

agregar índice autoincremental

convertir fechas

Modelado (Hipótesis 1):

Regresión Lineal

Random Forest Regressor

Análisis (Hipótesis 2):

Matriz de correlación entre INDEC, CAC, AFCP

Heatmap

4. Hipótesis del proyecto
Hipótesis 1 – Predicción

"¿Es posible predecir el valor futuro del indicador usando ML supervisado?"

Modelos probados:

Regresión Lineal

Random Forest Regressor

Resultado:
Los modelos tuvieron baja capacidad predictiva: la variabilidad del índice es alta, los patrones no son estables y las fuentes no son consistentes entre sí.

Conclusión:
El indicador no es predecible con modelos supervisados básicos.

Hipótesis 2 – Correlación entre fuentes

"¿Existe relación estadística entre INDEC, CAC y AFCP?"

Procedimiento:

Agrupamiento por fecha, actividad y unidad de medida

Pivot para tener una columna por fuente

Cálculo de matriz de correlación

Visualización con heatmap

Resultado:
Las correlaciones son bajas o negativas.
No existe relación fuerte entre indicadores.

Conclusión:
No se debe depender de una única fuente para decisiones estratégicas.

5. Flujo ETL / ELT aplicado

Extracción (ADF)
Carga del CSV hacia SQL Database.

Carga (SQL Database)
Datos almacenados sin modificar.

Transformación (Databricks)
Limpieza completa:

renombrado

eliminación de columnas

tratamiento de nulos

tipificación

creación de índice

codificación con StringIndexer

Modelado (Databricks)

Supervisado: Lineal + Random Forest

Exploratorio: correlación entre fuentes

6. Resultados obtenidos
Modelo predictivo (Hipótesis 1)

Regresión Lineal: correlación débil

Random Forest: peor desempeño que la regresión

Conclusión técnica:
Los datos no contienen patrones predictivos fuertes debido a:

alta dispersión

diferencias entre fuentes

múltiples actividades/productos

estructura heterogénea

Correlación (Hipótesis 2)

Correlaciones bajas

Heatmap con predominancia de valores cercanos a 0

Muchas combinaciones incompletas entre las tres fuentes

Conclusión analítica:
No existe un índice “principal” u homogéneo entre organismos.

7. Gestión de costos del proyecto

Los costos reales del uso en Azure fueron:

Servicio	Uso real	Costo aproximado
Azure Data Factory	Copy Data	~0,70 USD
Azure SQL Database	Escalado automático	dentro del crédito
Azure Databricks	1 cluster pequeño	~0,52 USD
Otros cargos residuales	–	~0,20 USD
Total real consumido		~18 USD

Todo dentro de los 200 USD de crédito gratuito.

📉 Diferencia con un escenario de producción
Azure SQL Database

✔ Plan Básico 5 DTU → ~4.90 USD / mes
✔ Uso General (serverless 1 vCore) → ~0.30 USD / mes
✔ Facturación por segundo

Databricks

✔ Cluster de trabajo → se paga por hora
✔ Se sugiere usar auto-shutdown

ADF

✔ Se paga por DIUHours (muy bajo en este proyecto)

8. Cierre y desactivación de recursos

Para evitar cargos:

Eliminé todos los Resource Groups

Cancelé la suscripción de Azure

La tarjeta quedó pendiente de eliminación hasta el cierre del período

Confirmé que no quedaban servicios activos

9. Conclusiones finales

Se implementó una arquitectura Cloud escalable y reproducible.

Se procesó de punta a punta el dataset construccion.csv.

Se aplicaron dos enfoques distintos: predictivo y correlativo.

Los modelos no permitieron predecir el índice con precisión.

Las fuentes económicas no están correlacionadas entre sí.

Se construyó una base sólida para dashboards, análisis de tendencias o modelos avanzados.

El proyecto se completó con bajo costo gracias a la prueba gratuita
