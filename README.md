# 📊 Dashboard de Seguimiento de Ventas - Power BI

<p align="center">
  <img src="https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black" />
  <img src="https://img.shields.io/badge/PostgreSQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white" />
</p>

## 📋 Descripción

Dashboard de Business Intelligence desarrollado para el seguimiento y control de ventas del área comercial de una empresa. El reporte permite monitorear las ventas cerradas en el período 2019–2022, compararlas con el presupuesto establecido (PPTO) y determinar superávits y déficits respecto a las metas del negocio.

> **Contexto**: El proyecto simula un escenario real donde la empresa realizaba sus reportes de forma completamente manual, descargando extensas hojas de datos desde distintos orígenes. El objetivo fue reemplazar ese flujo ineficiente por un modelo de datos centralizado y un dashboard interactivo que permita tomar decisiones en tiempo real.

> **Datos**: Los datos fueron proporcionados por [Biwiser](https://www.biwiser.com) en el marco de certificación en Power BI.

---

## 📊 Ver Dashboard

👉 **[Abrir en Power BI](https://app.powerbi.com/view?r=eyJrIjoiYmJkMmQ3NzctMDg5NS00ZTA1LWIwODYtMDc5OTJjOTQ2OWIxIiwidCI6IjM2YjZkNDEzLTNiNmYtNDgxYS1iYzlkLTY2ODliNTExY2FmYSIsImMiOjR9&pageName=4601d1f28000453dae40)** 👈

---

## 🎯 Objetivos del Dashboard

| Objetivo | Descripción |
|---|---|
| **Seguimiento de ventas** | Monitorear las ventas cerradas por el área comercial en el período 2019–2022 |
| **Control presupuestario** | Comparar ventas reales vs. presupuesto (PPTO) por vendedor, segmento y período |
| **Análisis de desviaciones** | Identificar superávits y déficits respecto a las metas establecidas |
| **Segmentación** | Analizar el comportamiento por producto, categoría, segmento y región |
| **Distribución de volumen** | Clasificar las ventas según intervalos de cantidad vendida |

---

## 🗃️ Fuentes de Datos

Datos extraídos desde una base de datos **PostgreSQL** con 7 tablas (2 de hechos transaccionales + 1 de presupuesto + 4 dimensiones):

| Tabla | Tipo | Descripción |
|---|---|---|
| `M2_Base_Ventas_2019_2020` | Hechos | Transacciones de ventas del período 2019–2020 |
| `M2_Base_Ventas_2021_2022` | Hechos | Transacciones de ventas del período 2021–2022 |
| `M2_Presupuesto_Ventas` | Hechos | Presupuesto mensual por vendedor y segmento |
| `M2_Maestro_Producto` | Dimensión | Categoría, subcategoría y nombre de producto |
| `M2_Maestro_Segmento` | Dimensión | Segmentos de venta |
| `M2_Maestro_Vendedor` | Dimensión | Información de vendedores (ID, nombre, cargo) |
| `M2_Tipo_de_cambio_USD` | Dimensión | Tipo de cambio USD por fecha |

---

## ⚙️ Proceso ETL en Power Query

La transformación y limpieza de datos fue una etapa crítica antes de construir el modelo. Los pasos aplicados fueron:

### Normalización y limpieza
- **Unpivot de M2_Presupuesto_Ventas** - La tabla de presupuesto tenía los meses como columnas (estructura matricial). Se normalizó a formato tabular (fila por mes) para poder relacionarla correctamente con el modelo de datos.
- **Append de tablas de ventas** - Las tablas `M2_Base_Ventas_2019_2020` y `M2_Base_Ventas_2021_2022`, con idéntica estructura, se anexaron en una sola tabla consolidada renombrada como **"Ventas Consolidado"**.
- **Eliminación de duplicados** - Se verificó la unicidad en las tres tablas de dimensión (Producto, Segmento y Vendedor) y se eliminaron registros duplicados para garantizar la integridad del modelo.
- **División de columna compuesta** - En `M2_Maestro_Vendedor`, la columna `DNI / Cargo` combinaba dos datos en uno. Se dividió en columnas separadas para permitir filtros y análisis por cargo de forma independiente.
- **Normalización de texto** - Todos los campos de texto del modelo se transformaron a **MAYÚSCULAS** para garantizar consistencia y evitar duplicados por diferencias tipográficas.

---

## 🗂️ Modelado de Datos

Se construyó un **modelo estrella (Star Schema)** con las tablas de hechos al centro y las dimensiones relacionadas en su perímetro.

```
                  ┌─────────────────┐
                  │  Dim Calendario │
                  └────────┬────────┘
                           │
┌──────────────┐    ┌──────┴──────────┐    ┌────────────────────┐
│ Dim Producto │────│ Ventas Consolidado│────│ Dim Tipo Cambio USD│
└──────────────┘    └──────┬──────────┘    └────────────────────┘
                           │
┌──────────────┐    ┌──────┴──────────┐    ┌────────────────────┐
│ Dim Segmento │────│  PPTO Ventas    │────│   Dim Vendedor     │
└──────────────┘    └─────────────────┘    └────────────────────┘
```

- **Tabla Calendario** - Creada con DAX, contiene campos de Fecha, Año, Mes y Nombre de Mes. Se relaciona con todas las tablas de hechos para habilitar análisis temporales consistentes.

---

## 📐 DAX: Medidas y Columnas Calculadas

### Columna calculada
- **`Intervalo Q venta`** - Clasifica cada transacción según el volumen vendido en cuatro rangos:

| Condición | Etiqueta |
|---|---|
| Cantidad entre 1 y 4 | `"1 a 4 Unid"` |
| Cantidad entre 5 y 8 | `"5 a 8 Unid"` |
| Cantidad entre 9 y 12 | `"9 a 12 Unid"` |
| Cantidad >= 13 | `"13 o Más Unid"` |

### Medidas calculadas
- **`Suma Ventas`** - Total de ventas en USD del período seleccionado
- **`Suma PPTO`** - Total del presupuesto en USD del período seleccionado
- **`Var Ventas/PPTO`** - Variación absoluta entre ventas reales y presupuesto
- **`Var % Ventas/PPTO`** - Variación porcentual entre ventas reales y presupuesto
- **`CALCULATE`** - Medida con contexto de filtro modificado para análisis comparativos avanzados

---

## 📈 Visualizaciones y Diseño

El dashboard cubre los cuatro ejes de análisis visual:

| Eje | Tipo de visual |
|---|---|
| **Composición** | Gráficos de torta / barras apiladas por categoría y segmento |
| **Comparación** | Gráficos de barras ventas vs. PPTO por vendedor y período |
| **Resumen** | Tarjetas KPI con totales de ventas, PPTO y variaciones |
| **Detalle** | Tablas con apertura por producto, vendedor y región |

### Funcionalidades de diseño implementadas
- **Tooltip personalizado** - Ventana emergente con información adicional al pasar el cursor sobre los visuals
- **Formato condicional** - Iconos y colores de fuente para identificar visualmente superávit (verde) y déficit (rojo) respecto al PPTO
- **Tema personalizado** - Diseño visual coherente descargado desde la comunidad de Power BI

---

## 🧹 Proceso Analítico: Del Dato Crudo al Dashboard

El proyecto siguió el flujo estándar de un proyecto BI real:

1. **Conectividad** → Conexión directa a base de datos PostgreSQL (no archivos planos)
2. **Power Query** → Limpieza, normalización y consolidación de las 7 tablas
3. **Modelado** → Construcción del modelo estrella y tabla calendario en DAX
4. **DAX** → Columnas y medidas calculadas para las métricas del negocio
5. **Visualizaciones** → Diseño de los 4 tipos de análisis requeridos
6. **Diseño** → Tooltips, formatos condicionales y tema visual

El desafío más relevante fue la **normalización del presupuesto**: la tabla original tenía 12 columnas (una por mes), lo que hacía imposible relacionarla correctamente con el modelo. Aplicar el unpivot en Power Query fue la decisión clave que desbloqueó el análisis ventas vs. PPTO.

---

## 🛠️ Herramientas

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white) ![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white) ![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft&logoColor=white)

---

## 📫 Contacto

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/cristobalmoyacantillana/)
