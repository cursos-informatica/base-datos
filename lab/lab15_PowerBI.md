# Laboratorio 15: Power BI - Importación de Datos y Creación de Medidas

## Objetivos
- Importar múltiples archivos CSV desde una carpeta
- Combinar datos de diferentes períodos en un único modelo
- Crear medidas DAX para análisis de ventas
- Diseñar visualizaciones interactivas

## Requisitos Previos
- Power BI Desktop instalado
- Archivos CSV en la carpeta `data/`:
  - `setiembre - Sheet1.csv`
  - `octubre - Sheet1.csv`
  - `noviembre - Sheet1.csv`

## Estructura de los Datos

Los archivos contienen información de ventas con las siguientes columnas:

| Columna | Tipo | Descripción |
|---------|------|-------------|
| Fecha | Date | Fecha de la venta |
| ID_Producto | Text | Identificador único del producto |
| Producto | Text | Nombre del producto |
| Precio_Unitario | Decimal | Precio por unidad |
| Cantidad | Integer | Cantidad vendida |
| Precio_Total | Decimal | Total de la venta |
| Sucursal | Text | Ubicación de la sucursal |

---

## Parte 1: Importación de Datos

### 1.1 Importar desde Carpeta

1. Abrir **Power BI Desktop**
2. Ir a **Inicio** ’ **Obtener datos** ’ **Más...**
3. Seleccionar **Carpeta** y hacer clic en **Conectar**
4. Navegar hasta la carpeta `data/` del proyecto
5. Clic en **Aceptar**

### 1.2 Transformar y Combinar Archivos

1. En la ventana de vista previa, clic en **Transformar datos**
2. En el Editor de Power Query:
   - Filtrar solo archivos `.csv` (excluir el archivo `.zip`)
   - Clic en el botón **Combinar archivos** (icono de dos flechas)
   - Seleccionar el archivo de ejemplo y clic en **Aceptar**

3. **Limpiar datos combinados:**
   ```
   - Eliminar la columna "Source.Name" si no es necesaria
   - O renombrarla a "Mes_Origen" para identificar el archivo fuente
   ```

4. **Configurar tipos de datos:**
   - `Fecha` ’ Tipo **Fecha**
   - `ID_Producto` ’ Tipo **Texto**
   - `Producto` ’ Tipo **Texto**
   - `Precio_Unitario` ’ Tipo **Número decimal**
   - `Cantidad` ’ Tipo **Número entero**
   - `Precio_Total` ’ Tipo **Número decimal**
   - `Sucursal` ’ Tipo **Texto**

5. Clic en **Cerrar y aplicar**

### 1.3 Método Alternativo: Importar Archivos Individualmente

Si prefieres importar cada archivo por separado:

1. **Inicio** ’ **Obtener datos** ’ **Texto o CSV**
2. Seleccionar `setiembre - Sheet1.csv`
3. Repetir para `octubre` y `noviembre`
4. En Power Query, usar **Anexar consultas** para unirlas:
   - Seleccionar la primera tabla
   - **Inicio** ’ **Anexar consultas** ’ **Anexar consultas como nuevas**
   - Seleccionar las tres tablas
   - Renombrar la consulta resultante como `Ventas`

---

## Parte 2: Creación de Medidas DAX

### 2.1 Medidas Básicas de Ventas

Ir a la vista de **Modelo** o **Datos** y crear las siguientes medidas:

#### Total de Ventas
```dax
Total Ventas = SUM('Ventas'[Precio_Total])
```

#### Cantidad Total Vendida
```dax
Cantidad Total = SUM('Ventas'[Cantidad])
```

#### Número de Transacciones
```dax
Num Transacciones = COUNTROWS('Ventas')
```

#### Precio Promedio por Unidad
```dax
Precio Promedio = AVERAGE('Ventas'[Precio_Unitario])
```

### 2.2 Medidas de Análisis Avanzado

#### Ticket Promedio (Venta promedio por transacción)
```dax
Ticket Promedio = DIVIDE([Total Ventas], [Num Transacciones], 0)
```

#### Cantidad Promedio por Transacción
```dax
Cantidad Promedio = DIVIDE([Cantidad Total], [Num Transacciones], 0)
```

#### Margen Estimado (asumiendo 30% de margen)
```dax
Margen Estimado = [Total Ventas] * 0.30
```

### 2.3 Medidas de Comparación Temporal

#### Ventas del Mes Anterior
```dax
Ventas Mes Anterior =
CALCULATE(
    [Total Ventas],
    DATEADD('Ventas'[Fecha], -1, MONTH)
)
```

#### Variación Mensual (%)
```dax
Variacion Mensual % =
VAR VentasActual = [Total Ventas]
VAR VentasAnterior = [Ventas Mes Anterior]
RETURN
DIVIDE(VentasActual - VentasAnterior, VentasAnterior, 0)
```

#### Ventas Acumuladas (YTD)
```dax
Ventas Acumuladas =
CALCULATE(
    [Total Ventas],
    DATESYTD('Ventas'[Fecha])
)
```

### 2.4 Medidas de Ranking y Análisis

#### Producto Más Vendido (por cantidad)
```dax
Ranking Producto =
RANKX(
    ALL('Ventas'[Producto]),
    [Cantidad Total],
    ,
    DESC,
    DENSE
)
```

#### Participación por Sucursal (%)
```dax
Participacion Sucursal % =
DIVIDE(
    [Total Ventas],
    CALCULATE([Total Ventas], ALL('Ventas'[Sucursal])),
    0
)
```

#### Top 5 Productos Flag
```dax
Es Top 5 = IF([Ranking Producto] <= 5, "Top 5", "Otros")
```

---

## Parte 3: Crear Tabla de Calendario

Para análisis temporal avanzado, crear una tabla de fechas:

### 3.1 Crear Tabla de Calendario con DAX

En la vista de **Modelo**, ir a **Modelado** ’ **Nueva tabla**:

```dax
Calendario =
VAR FechaMin = MIN('Ventas'[Fecha])
VAR FechaMax = MAX('Ventas'[Fecha])
RETURN
ADDCOLUMNS(
    CALENDAR(FechaMin, FechaMax),
    "Año", YEAR([Date]),
    "Mes", MONTH([Date]),
    "Nombre Mes", FORMAT([Date], "MMMM"),
    "Mes Corto", FORMAT([Date], "MMM"),
    "Día", DAY([Date]),
    "Día Semana", WEEKDAY([Date]),
    "Nombre Día", FORMAT([Date], "dddd"),
    "Semana", WEEKNUM([Date]),
    "Trimestre", "Q" & QUARTER([Date]),
    "Año-Mes", FORMAT([Date], "YYYY-MM")
)
```

### 3.2 Establecer Relación

1. Ir a la vista de **Modelo**
2. Arrastrar `Calendario[Date]` hacia `Ventas[Fecha]`
3. Verificar que la relación sea de tipo **Uno a Muchos** (1:*)

### 3.3 Marcar como Tabla de Fechas

1. Seleccionar la tabla `Calendario`
2. **Herramientas de tabla** ’ **Marcar como tabla de fechas**
3. Seleccionar la columna `Date`

---

## Parte 4: Visualizaciones Sugeridas

### 4.1 Dashboard de Ventas

Crear un informe con las siguientes visualizaciones:

| Visualización | Campos | Descripción |
|---------------|--------|-------------|
| **Tarjeta** | [Total Ventas] | KPI principal |
| **Tarjeta** | [Cantidad Total] | Unidades vendidas |
| **Tarjeta** | [Ticket Promedio] | Venta promedio |
| **Gráfico de barras** | Sucursal, [Total Ventas] | Ventas por sucursal |
| **Gráfico de líneas** | Fecha, [Total Ventas] | Tendencia temporal |
| **Tabla** | Producto, [Cantidad Total], [Total Ventas] | Detalle por producto |
| **Gráfico circular** | Sucursal, [Participacion Sucursal %] | Distribución |

### 4.2 Segmentadores Recomendados

Agregar los siguientes filtros interactivos:
- **Mes** (de la tabla Calendario)
- **Sucursal**
- **Producto**

---

## Parte 5: Ejercicios Prácticos

### Ejercicio 1: Medidas Adicionales
Crear las siguientes medidas:

1. **Venta Máxima**: La transacción más alta
```dax
Venta Maxima = MAX('Ventas'[Precio_Total])
```

2. **Productos Distintos Vendidos**: Contar productos únicos
```dax
Productos Distintos = DISTINCTCOUNT('Ventas'[ID_Producto])
```

3. **Sucursales Activas**: Contar sucursales con ventas
```dax
Sucursales Activas = DISTINCTCOUNT('Ventas'[Sucursal])
```

### Ejercicio 2: Análisis Condicional
Crear una medida que clasifique las ventas:

```dax
Categoria Venta =
SWITCH(
    TRUE(),
    [Total Ventas] >= 1000, "Alta",
    [Total Ventas] >= 500, "Media",
    "Baja"
)
```

### Ejercicio 3: Comparación de Sucursales
Crear una medida que muestre la diferencia vs. el promedio general:

```dax
Diferencia vs Promedio =
VAR PromedioGeneral = CALCULATE([Total Ventas], ALL('Ventas'[Sucursal])) / DISTINCTCOUNT(ALL('Ventas'[Sucursal]))
RETURN
[Total Ventas] - PromedioGeneral
```

---

## Parte 6: Formato y Mejores Prácticas

### 6.1 Formatear Medidas

Para cada medida numérica, configurar el formato apropiado:

| Medida | Formato |
|--------|---------|
| Total Ventas | Moneda ($) |
| Cantidad Total | Número entero |
| Ticket Promedio | Moneda ($) |
| Variacion Mensual % | Porcentaje (%) |
| Participacion Sucursal % | Porcentaje (%) |

### 6.2 Organizar Medidas

1. Crear una **tabla de medidas** vacía:
```dax
_Medidas = ROW("Aux", BLANK())
```

2. Mover todas las medidas a esta tabla
3. Ocultar la columna `Aux`

### 6.3 Documentar Medidas

Agregar descripciones a cada medida:
1. Seleccionar la medida
2. En el panel de **Propiedades**, agregar una descripción

---

## Resumen de Medidas Creadas

| # | Medida | Fórmula DAX |
|---|--------|-------------|
| 1 | Total Ventas | `SUM('Ventas'[Precio_Total])` |
| 2 | Cantidad Total | `SUM('Ventas'[Cantidad])` |
| 3 | Num Transacciones | `COUNTROWS('Ventas')` |
| 4 | Precio Promedio | `AVERAGE('Ventas'[Precio_Unitario])` |
| 5 | Ticket Promedio | `DIVIDE([Total Ventas], [Num Transacciones], 0)` |
| 6 | Cantidad Promedio | `DIVIDE([Cantidad Total], [Num Transacciones], 0)` |
| 7 | Margen Estimado | `[Total Ventas] * 0.30` |
| 8 | Ventas Mes Anterior | `CALCULATE([Total Ventas], DATEADD(...))` |
| 9 | Variacion Mensual % | `DIVIDE(Actual - Anterior, Anterior, 0)` |
| 10 | Ventas Acumuladas | `CALCULATE([Total Ventas], DATESYTD(...))` |
| 11 | Ranking Producto | `RANKX(ALL(...), [Cantidad Total], ...)` |
| 12 | Participacion Sucursal % | `DIVIDE([Total Ventas], ALL(...), 0)` |

---

## Entregables

1. Archivo `.pbix` con:
   - Datos importados y combinados
   - Todas las medidas creadas
   - Tabla de calendario
   - Al menos 5 visualizaciones
   - Segmentadores interactivos

2. Capturas de pantalla de:
   - Vista de modelo con relaciones
   - Dashboard principal
   - Lista de medidas creadas

---

## Recursos Adicionales

- [Documentación DAX](https://docs.microsoft.com/es-es/dax/)
- [Power BI Desktop](https://powerbi.microsoft.com/desktop/)
- [Patrones DAX](https://www.daxpatterns.com/)
