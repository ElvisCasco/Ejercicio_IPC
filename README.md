# Ejercicio_IPC

Este es solamente un ejercicio, no contiene datos reales. El objetivo del ejercicio es replicar metodologías de cálculo del IPC para Honduras.

## Acerca de `Fórmulas.qmd`

`Fórmulas.qmd` es el documento principal del repositorio. Se trata de un cuaderno Quarto en Python (kernel `jupyter: python3`) que combina código y explicación metodológica para mostrar, paso a paso, cómo se construye un Índice de Precios al Consumidor (IPC) a partir de datos brutos de precios. El archivo se genera por defecto en formato Word (`.docx`) usando la plantilla `custom-reference.docx`.

### Insumos

El cuaderno parte de dos archivos Excel ubicados en la raíz del repositorio:

- **`Ejercicio calculo IPC - Investigación.xlsx`** — una hoja por cada región (MDC, RUC, MSPS, RUN, ULA, UOri, UOcc, US), con precios y contenido (unidad de medida) por establecimiento, variedad y producto, en los períodos $t$ y $t-1$.
- **`Categorias.xlsx`** — contiene tres hojas:
  - `Regiones`: ponderaciones por producto y región;
  - `IPC_Calc`: ponderaciones del IPC por Estructura, Código, CCIF y Región;
  - `Zonas`: ponderaciones de cada región dentro del IPC general.

### Regiones consideradas

| Código | Descripción |
|---|---|
| MDC | Metropolitana Distrito Central |
| RUC | Resto Urbano Central |
| MSPS | Metropolitana San Pedro Sula |
| RUN | Resto Urbano Norte |
| ULA | Urbana Litoral Atlántico |
| UOri | Urbana Oriental |
| UOcc | Urbana Occidental |
| US | Urbana Sur |

### Pipeline de cálculo

El cuaderno ejecuta el siguiente flujo para cada región y luego consolida los resultados:

1. **Lectura y limpieza** del Excel regional: detección y reparación de cabeceras, filtrado a las filas válidas (Precio, Unidad de Medida, Contenido) y depuración de los códigos de establecimiento.
2. **Pivot a formato largo**: cada fila representa una combinación (Región, Código, Establecimiento, Variedad) con sus columnas `Precio_Unitario_{t,t-1}` y `Contenido_{t,t-1}`.
3. **Cálculo de precios base**: $\text{Precio}_t = \text{Precio Unitario}_t / \text{Contenido}_t$ (análogo para $t-1$).
4. **Índice elemental por establecimiento (Carli)**: $I_c = \text{Precio}_t / \text{Precio}_{t-1}$.
5. **Índice por variedad (Jevons)**: media geométrica de los $I_c$ del paso anterior.
6. **Índice por producto**: media geométrica de los índices por variedad correspondientes al mismo código CCIF de 8 dígitos.
7. **Índices de nivel superior (Young / Laspeyres)**: medias aritméticas ponderadas por niveles de la CCIF — Categoría (6 dígitos), SubClase (5), Clase (4), Grupo (3), División (2) — usando las ponderaciones `Pond_IPC`.
8. **Agregación regional y general**: combinación de los índices regionales con sus respectivas ponderaciones de Zona para obtener el IPC general.

El cuaderno presenta **dos implementaciones equivalentes** del mismo pipeline:

- Una basada en **DuckDB** (consultas SQL sobre archivos Parquet), bajo la sección `# Resultados usando DuckDB`.
- Otra basada en **Polars** (DataFrames en memoria), bajo la sección `# Resultados usando Polars`.

### Salidas

Al ejecutarse, el cuaderno escribe los siguientes artefactos:

- En `data/`: `precios_all.parquet/csv`, `indices_variedades.parquet/csv`, `ponderaciones.parquet`, `indices_producto.parquet/csv`, e `indices_g_{producto, Region, Division, Grupo}.parquet/csv`.
- En la raíz: `IPC.xlsx`, `IPC_Productos_Regiones.xlsx`, `IPC_Producto.xlsx`, `IPC_{Categoría, SubClase, Clase, Grupo, División}.xlsx`, `IPC_Regiones.xlsx`, `IPC_Regiones_Divisiones.xlsx`.
- En las carpetas por región (`MDC/`, `RUC/`, etc.): archivos por establecimiento, variedad, producto y por cada nivel de agrupación CCIF.
- En las carpetas por nivel CCIF (`Producto/`, `Categoría/`, `SubClase/`, `Clase/`, `Grupo/`, `División/`): el resultado de cada región para ese nivel.

### Fundamentos metodológicos incluidos en el documento

Después del código, el `.qmd` desarrolla la teoría con ejemplos numéricos:

- **Índices elementales**: definiciones y fórmulas de Carli, Dutot y Jevons.
- **Ejemplo de Arroz Clasificado** (CCIF `01111201`) en la región MDC, mostrando todas las tablas intermedias (precios, contenidos, precios unitarios, índices individuales, índices por variedad, índice por producto).
- **Índices de nivel superior**: derivación del índice de Young como promedio ponderado de índices elementales, con la fórmula $I^{0:t} = \sum w^b_i\,I^{0:t}_i$ con $\sum w^b_i = 1$.
- **Tablas finales** del IPC por productos y regiones, por productos, por cada nivel CCIF, por regiones, por regiones y divisiones, y el IPC general.

### Tecnologías

- **Python 3** vía kernel Jupyter
- **Polars** — manipulación de DataFrames
- **DuckDB** — analítica SQL embebida
- **NumPy** — cálculos numéricos auxiliares
- **great_tables** — formato de tablas en el documento renderizado
- **Quarto** — sistema de documentación

Las dependencias mínimas se listan en `requirements.txt`.

### Nota sobre configuración (`wd`)

Las primeras líneas de código contienen el directorio de trabajo `wd`, que está fijado a rutas locales del autor (`//srvfs02/IE/Investigaciones/2025/IPC_Calc/` y `C:/Trabajo/GitHub/Ejercicio_IPC-main/`). Para ejecutar el cuaderno en otro equipo, se debe **modificar `wd`** apuntándolo a la ruta local donde se haya clonado el repositorio.

### Cómo renderizar el documento

```bash
quarto render Fórmulas.qmd
```

Por defecto se generará `Fórmulas.docx` usando la plantilla `custom-reference.docx`. Si se desea una versión HTML, basta con descomentar el bloque `html:` en el encabezado YAML del `.qmd`.
