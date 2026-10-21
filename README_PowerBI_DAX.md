# 📊 Power BI · Modelamiento de Datos y DAX para KPIs de RRHH

Guía técnica del tablero `Reporte_DAX.pbix`: cómo se modeló la data, qué fórmulas DAX alimentan cada KPI y cómo se ve el resultado en el reporte.

> Documentación general del proyecto: [README.md](README.md)

---

## Índice
1. [Flujo de trabajo](#1-flujo-de-trabajo)
2. [Modelamiento de datos](#2-modelamiento-de-datos)
3. [Fórmulas DAX para los KPIs](#3-fórmulas-dax-para-los-kpis)
4. [Tablero: páginas y KPIs](#4-tablero-páginas-y-kpis)
5. [Buenas prácticas aplicadas](#5-buenas-prácticas-aplicadas)

---

## 1. Flujo de trabajo

```
 Excel (3 hojas)  →  Power Query  →  Modelo relacional  →  DAX  →  Reporte (3 páginas)
 Datos Empleados     limpieza y       1 : 1 por           medidas y   KPIs, mapas,
                     tipos            ID Empleado         columnas    gráficos
```

| Paso | Qué se hizo |
|---|---|
| 1. Obtener datos | Importar `Datos Empleados.xlsx` (Tabla Empleados, Tabla Evaluacion, Tabla Sueldo) |
| 2. Transformar | Tipos de datos, recorte de espacios en `Departamento`, estandarizar `Estado` |
| 3. Modelar | Relaciones por `ID Empleado` |
| 4. Calcular | Medidas y columnas calculadas en DAX |
| 5. Visualizar | Tarjetas KPI, columnas, barras, dona, dispersión y mapas |

📸 **Captura del flujo**

![Flujo de trabajo](images/01_flujo.png)

---

## 2. Modelamiento de datos

### 2.1 Origen de datos
| Tabla | Filas | Campos |
|---|---|---|
| Tabla Empleados | 194 | ID Empleado, Nombre Empleado, Estado, Nacimiento, Género, Departamento, Posición, Nombre Jefe |
| Tabla Evaluacion | 194 | ID Empleado, Evaluación |
| Tabla Sueldo | 194 | ID Empleado, Sueldo |

### 2.2 Transformaciones en Power Query
- `ID Empleado` como **texto** (es una clave, no se suma).
- `Nacimiento` como **fecha**; `Evaluación` y `Sueldo` como **número**.
- `Departamento`: **Recortar** y **Limpiar**. Había valores como `"Production       "` y `"Software Engineering     "` que partían una misma categoría en dos.
- `Estado` con formato `Estado, Estados Unidos` para que el mapa geocodifique bien.

📸 **Captura de Power Query**

![Power Query](images/02_power_query.png)

### 2.3 Relaciones

| Desde | Hacia | Cardinalidad | Dirección del filtro |
|---|---|---|---|
| Tabla Empleados[ID Empleado] | Tabla Evaluacion[ID Empleado] | 1 : 1 | Ambas / única desde Empleados |
| Tabla Empleados[ID Empleado] | Tabla Sueldo[ID Empleado] | 1 : 1 | Ambas / única desde Empleados |

`ID Empleado` es único y existe en las tres tablas (194 de 194), por lo que no hay registros huérfanos.

📸 **Captura de la vista de modelo**

![Modelo de datos](images/03_modelo.png)

---

## 3. Fórmulas DAX para los KPIs

> ⚠️ Revisa que estas fórmulas coincidan con las de tu `.pbix` (los nombres de medidas sí son los del reporte; los rangos de los grupos son definiciones de referencia).

### 3.1 Medidas (KPIs)

**Total de empleados**
```dax
Total_empleados =
DISTINCTCOUNT ( 'Tabla Empleados'[ID Empleado] )
```

**Evaluación promedio**
```dax
Evaluacion_promedio =
AVERAGE ( 'Tabla Evaluacion'[Evaluación] )
```

**Sueldo promedio**
```dax
Sueldo_promedio =
AVERAGE ( 'Tabla Sueldo'[Sueldo] )
```

### 3.2 Columnas calculadas (segmentación)

**Edad**
```dax
Edad =
DATEDIFF ( 'Tabla Empleados'[Nacimiento], TODAY (), YEAR )
```

**Grupo de edad**
```dax
Grupo edad =
SWITCH (
    TRUE (),
    'Tabla Empleados'[Edad] < 30, "Menos de 30",
    'Tabla Empleados'[Edad] < 40, "30 - 39",
    'Tabla Empleados'[Edad] < 50, "40 - 49",
    "50 o más"
)
```

**Grupo de sueldo**
```dax
Grupo_sueldo =
SWITCH (
    TRUE (),
    'Tabla Sueldo'[Sueldo] < 50000,  "Menos de 50 mil",
    'Tabla Sueldo'[Sueldo] < 80000,  "50 - 80 mil",
    'Tabla Sueldo'[Sueldo] < 110000, "80 - 110 mil",
    "Más de 110 mil"
)
```

**Grupo de evaluación**
```dax
Grupo_evaluacion =
SWITCH (
    TRUE (),
    'Tabla Evaluacion'[Evaluación] < 6, "Bajo (< 6)",
    'Tabla Evaluacion'[Evaluación] < 8, "Medio (6 - 8)",
    "Alto (≥ 8)"
)
```

### 3.3 Resumen: qué fórmula alimenta qué KPI

| KPI / visual | Fórmula | Funciones DAX |
|---|---|---|
| Total de colaboradores | `Total_empleados` | `DISTINCTCOUNT` |
| Evaluación promedio | `Evaluacion_promedio` | `AVERAGE` |
| Sueldo promedio | `Sueldo_promedio` | `AVERAGE` |
| Edad (tarjeta y dispersión) | `Edad` | `DATEDIFF`, `TODAY` |
| Empleados por rango de edad | `Grupo edad` | `SWITCH`, `TRUE` |
| Empleados por rango de sueldo | `Grupo_sueldo` | `SWITCH`, `TRUE` |
| Empleados por rango de evaluación | `Grupo_evaluacion` | `SWITCH`, `TRUE` |

📸 **Captura de las medidas DAX**

![Medidas DAX](images/04_medidas_dax.png)

---

## 4. Tablero: páginas y KPIs

Las tres páginas comparten el mismo diseño: menú de navegación con botones, tarjetas KPI arriba y visuales de detalle abajo. Todos los visuales se filtran entre sí.

### 4.1 Total Colaboradores
KPIs: `Total_empleados`, `Evaluacion_promedio`, `Sueldo_promedio`, `Edad`.
Visuales: colaboradores por departamento, por estado (mapa), por género (dona), por rango de edad, de sueldo y de evaluación, y tabla de detalle.

![Total Colaboradores](images/05_total_colaboradores.png)

### 4.2 Análisis de Sueldos
KPIs: `Total_empleados`, `Sueldo`, `Sueldo_promedio`.
Visuales: sueldo por departamento, por estado (mapa), por género (dona), dispersión Edad vs Sueldo y tabla de detalle.

![Análisis de Sueldos](images/06_analisis_sueldos.png)

### 4.3 Análisis de Evaluación
KPIs: `Total_empleados`, `Evaluacion_promedio`, `Evaluación`.
Visuales: evaluación por departamento, por jefe, por género, mapa por estado y tabla de detalle.

![Análisis de Evaluación](images/07_analisis_evaluacion.png)

---

## 5. Buenas prácticas aplicadas
- **Medidas en lugar de columnas** para los KPIs: se recalculan según el filtro activo.
- **`DISTINCTCOUNT`** sobre la clave para evitar duplicados.
- **Columnas calculadas solo para segmentar** (rangos de edad, sueldo y evaluación).
- **Limpieza en Power Query**, no en DAX, para que el modelo llegue ya normalizado.
- **Nombres consistentes** en medidas (`Total_empleados`, `Sueldo_promedio`, …).

---

**Herramientas:** Power BI Desktop · Power Query · DAX · Excel
**Autora:** [@Diana1295Dev](https://github.com/Diana1295Dev)
