# 📊 Dashboard de Control Logístico: Envíos y Costos 🚚

Un panel de control interactivo de Business Intelligence diseñado en **Power BI** para optimizar la toma de decisiones estratégicas, controlar el presupuesto de fletes y evaluar el rendimiento de la cadena de distribución física.

---

## 📈 Storytelling del Proyecto: De los Datos a la Estrategia

### 1. El Desafío de Negocio (La Situación Inicial)
Teníamos una operación con **220 envíos anuales** distribuidos en diferentes carriers y almacenes, pero el análisis de costos y la efectividad de las entregas estaba totalmente a ciegas. 

Necesitábamos unificar la información para responder de manera inmediata tres preguntas críticas: 
* ¿Cuánto gastamos realmente? 
* ¿Qué transportista rinde mejor y es más confiable?
* ¿Cómo fluctúa el costo logístico en el tiempo para adelantarnos a picos de demanda?

### 2. Los Descubrimientos Clave (Insights de Negocio)
* **Volumen vs. Gasto:** Cerramos el período analizado con un total de **220 envíos** y un costo de fletes acumulado de **$4.53 millones**.
* **Costo por Envío:** Identificamos que el costo promedio por despacho se ubica en **$20,570**. Este indicador es nuestro nuevo *baseline* para negociar tarifas de fletes.
* **Calidad del Servicio:** La tasa de entregas exitosas se sitúa en un **82.73%**, abriendo una oportunidad clave para auditar y exigir mejoras a los transportistas con peor desempeño.

---

## 🛠️ Arquitectura Técnica y Buenas Prácticas

Este proyecto no se diseñó de forma estática; se estructuró bajo estándares de desarrollo de software modernos:

* **Guardado en Formato de Proyecto (`.pbip`):** Almacenado como proyecto de Power BI, lo que permite separar el reporte, el dataset y las configuraciones en archivos de texto plano, facilitando el versionado con Git y la colaboración en equipos de datos.
* **Modelado Semántico con Lenguaje M:** Todo el proceso de ETL (Extracción, Transformación y Carga) se programó de manera optimizada en Power Query para garantizar la calidad del dato y la creación de un modelo estrella limpio.
* **Fórmulas DAX Robustas:** Uso de medidas DAX eficientes y seguras (por ejemplo, previniendo errores de división por cero mediante `DIVIDE`) para asegurar cálculos precisos y escalables.
* **Listo para Microsoft Fabric:** La arquitectura del modelo de datos está pensada para escalar y migrar directamente a un Lakehouse o Data Warehouse en Microsoft Fabric sin necesidad de rediseñar las consultas.

---

## 🧪 Código de Ingeniería de Datos (ETL)

A continuación, se detallan los scripts clave en **Lenguaje M** utilizados para construir y limpiar el modelo de datos:

### Creación de la Dimensión Calendario (`Dim_Calendario`)
Generación dinámica del calendario para el análisis temporal de la logística y cálculo de la clave de ordenación (`AñoMes_Key`) para evitar el orden alfabético de los meses:

```powerquery
let
    Origen = Fact_Shipments,
    FechasUnicas = List.Distinct(Origen[Shipment_Date]),
    FechaMinima = List.Min(FechasUnicas),
    FechaMaxima = List.Max(FechasUnicas),
    CantidadDias = Duration.Days(FechaMaxima - FechaMinima) + 1,
    ListaFechas = List.Dates(FechaMinima, CantidadDias, #duration(1, 0, 0, 0)),
    #"Convertido en tabla" = Table.FromList(ListaFechas, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Columnas renombradas" = Table.RenameColumns(#"Convertido en tabla",{{"Column1", "Date"}}),
    #"Tipo cambiado" = Table.TransformColumnTypes(#"Columnas renombradas",{{"Date", type date}}),
    #"Año Insertado" = Table.AddColumn(#"Tipo cambiado", "Año", each Date.Year([Date]), Int64.Type),
    #"Mes Insertado" = Table.AddColumn(#"Año Insertado", "Mes_Nro", each Date.Month([Date]), Int64.Type),
    #"Nombre de mes insertado" = Table.AddColumn(#"Mes Insertado", "Mes_Nombre", each Text.Title(Date.MonthName([Date])), type text),
    #"Clave AñoMes Añadida" = Table.AddColumn(#"Nombre de mes insertado", "AñoMes_Key", each [Año] * 100 + [Mes_Nro], Int64.Type)
in
    #"Clave AñoMes Añadida"