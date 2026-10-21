# 👥 Dashboard de Recursos Humanos · Modelado de Datos y DAX en Power BI

Reporte analítico de RRHH construido en **Power BI** a partir de un Excel con tres tablas (empleados, evaluaciones y sueldos). El proyecto muestra el flujo completo: **Excel → Power Query → modelo relacional → medidas y columnas DAX → reporte de 3 páginas**.

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/Lenguaje-DAX-0078D4)
![Excel](https://img.shields.io/badge/Fuente-Excel-217346?logo=microsoftexcel&logoColor=white)

---

## 📁 Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Datos Empleados.xlsx` | Fuente de datos (3 hojas) |
| `Reporte_DAX.pbix` | Modelo y reporte de Power BI |
| `README.md` | Documentación del proyecto |

---

## 1. Fuente de datos (Excel)

El libro `Datos Empleados.xlsx` tiene **3 hojas con 194 registros cada una**. Todas se relacionan por `ID Empleado`, que es único y está presente en las tres hojas (100 % de coincidencia).

| Hoja | Columnas | Contenido |
|---|---|---|
| **Tabla Empleados** | ID Empleado, Nombre Empleado, Estado, Nacimiento, Género, Departamento, Posición, Nombre Jefe | Datos demográficos y organizacionales |
| **Tabla Evaluacion** | ID Empleado, Evaluación | Puntaje de desempeño (5.01 – 9.93, promedio ≈ 7.56) |
| **Tabla Sueldo** | ID Empleado, Sueldo | Sueldo anual (26,536 – 144,000, promedio ≈ 74,722) |

**Perfil de los datos:** 108 mujeres / 86 hombres · 6 departamentos (Production, Information Technology, Admin Offices, Sales, Software Engineering, Executive Office) · 8 posiciones (de Administrative I a Director) · empleados concentrados en Texas (77), California (53) y Massachusetts (51).

---

## 2. Proceso de modelado de datos

### 2.1 Transformación (Power Query)
- Carga de las 3 hojas como tablas independientes.
- Tipificación de columnas: `Nacimiento` → fecha, `Evaluación` y `Sueldo` → numérico, `ID Empleado` → texto (clave).
- **Limpieza de texto:** se detectaron espacios sobrantes en `Departamento` (p. ej. `"Production       "`, `"Software Engineering     "`) que generan categorías duplicadas en los visuales. Se resuelve con *Recortar* (`Trim`) y *Limpiar* (`Clean`).
- Estandarización del campo `Estado` (`Estado, Estados Unidos`) para que el mapa geocodifique correctamente.

### 2.2 Modelo relacional (esquema en estrella)

```
                ┌──────────────────┐
                │ Tabla Evaluacion │
                │  ID Empleado (1) │
                └────────┬─────────┘
                         │ 1 : 1
┌────────────────────────┴─────────┐ 1 : 1 ┌──────────────────┐
│        Tabla Empleados           ├───────┤   Tabla Sueldo   │
│  ID Empleado (PK) · Departamento │       │  ID Empleado (1) │
│  Estado · Género · Posición · …  │       └──────────────────┘
└──────────────────────────────────┘
```

- **Tabla Empleados** actúa como tabla central (dimensión) y contiene los atributos de segmentación.
- Relaciones **uno a uno** por `ID Empleado` hacia `Tabla Evaluacion` y `Tabla Sueldo`.
- Filtro cruzado en una sola dirección desde la tabla de empleados.

### 2.3 Capa DAX
Sobre el modelo se crearon **columnas calculadas** (para segmentar) y **medidas** (para los KPI). Ver sección 3.

---

## 3. Fórmulas DAX

> ⚠️ Las fórmulas siguientes corresponden a los nombres de medidas y columnas que usa el reporte. Si alguna difiere de la de tu `.pbix`, ajústala en esta sección.

### Medidas

```dax
Total_empleados =
DISTINCTCOUNT ( 'Tabla Empleados'[ID Empleado] )
```

```dax
Evaluacion_promedio =
AVERAGE ( 'Tabla Evaluacion'[Evaluación] )
```

```dax
Sueldo_promedio =
AVERAGE ( 'Tabla Sueldo'[Sueldo] )
```

### Columnas calculadas

```dax
Edad =
DATEDIFF ( 'Tabla Empleados'[Nacimiento], TODAY (), YEAR )
```

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

```dax
Grupo_evaluacion =
SWITCH (
    TRUE (),
    'Tabla Evaluacion'[Evaluación] < 6, "Bajo (< 6)",
    'Tabla Evaluacion'[Evaluación] < 8, "Medio (6 - 8)",
    "Alto (≥ 8)"
)
```

### Funciones DAX utilizadas
| Función | Uso |
|---|---|
| `DISTINCTCOUNT` | Conteo único de colaboradores |
| `AVERAGE` | Promedios de sueldo y evaluación |
| `DATEDIFF` + `TODAY` | Cálculo de edad desde la fecha de nacimiento |
| `SWITCH ( TRUE (), … )` | Segmentación en rangos (edad, sueldo, evaluación) |

---

## 4. Reporte en Power BI (3 páginas)

Cada página comparte el mismo diseño: menú de navegación (botones), tarjetas KPI arriba, y visuales de detalle. La página activa se filtra de forma cruzada con los demás visuales.

### 📊 Página 1 · Total Colaboradores
Pregunta que responde: *¿Cuántos somos y cómo estamos compuestos?*

| Visual | Campos |
|---|---|
| Tarjetas KPI | `Total_empleados`, `Evaluacion_promedio`, `Sueldo_promedio`, `Edad` |
| Columnas | `Departamento` × `Total_empleados` |
| Mapa | `Estado` × `Total_empleados` |
| Dona | `Género` × `Total_empleados` |
| Barras | `Grupo edad`, `Grupo_sueldo`, `Grupo_evaluacion` × `Total_empleados` |
| Tabla de detalle | Nombre, Posición, Estado, Edad, Evaluación, Sueldo |

### 💰 Página 2 · Análisis de Sueldos
Pregunta que responde: *¿Cómo se distribuye la compensación?*

| Visual | Campos |
|---|---|
| Tarjetas KPI | `Total_empleados`, `Sueldo`, `Sueldo_promedio` |
| Columnas | `Departamento` × `Sueldo` |
| Mapa | `Estado` × `Sueldo` |
| Dona | `Género` × `Sueldo` (brecha de género) |
| Dispersión | `Edad` vs `Sueldo` por `Nombre_Empleado` |
| Tabla de detalle | Nombre, Posición, Estado, Edad, Evaluación, Sueldo promedio |

### ⭐ Página 3 · Análisis de Evaluación
Pregunta que responde: *¿Cómo es el desempeño y dónde se concentra?*

| Visual | Campos |
|---|---|
| Tarjetas KPI | `Total_empleados`, `Evaluacion_promedio`, `Evaluación` |
| Columnas | `Departamento` × `Evaluacion_promedio` |
| Barras | `Nombre Jefe` × `Evaluacion_promedio` (desempeño por líder) |
| Columnas | `Género` × `Evaluacion_promedio` |
| Mapa de relleno | `Estado` × `Evaluacion_promedio` / `Total_empleados` |
| Tabla de detalle | Nombre, Posición, Estado, Edad, Evaluación, Sueldo |

---

## 5. Match Excel ↔ Modelo ↔ Reporte

| Dato en Excel | Tabla / Campo en el modelo | Cálculo DAX | Dónde se ve en el reporte |
|---|---|---|---|
| `ID Empleado` | `Tabla Empleados[ID Empleado]` | `Total_empleados` | Tarjetas de las 3 páginas |
| `Nacimiento` | `Tabla Empleados[Nacimiento]` | `Edad`, `Grupo edad` | Tarjeta Edad, barras por rango, dispersión |
| `Departamento` | `Tabla Empleados[Departamento]` | — (eje) | Columnas en las 3 páginas |
| `Estado` | `Tabla Empleados[Estado]` | — (ubicación) | Mapas en las 3 páginas |
| `Género` | `Tabla Empleados[Género]` | — (leyenda) | Donas y columnas por género |
| `Nombre Jefe` | `Tabla Empleados[Nombre Jefe]` | — (eje) | Barras de evaluación por jefe |
| `Posición`, `Nombre Empleado` | `Tabla Empleados` | — | Tabla de detalle |
| `Evaluación` | `Tabla Evaluacion[Evaluación]` | `Evaluacion_promedio`, `Grupo_evaluacion` | Página Evaluación + barras por rango |
| `Sueldo` | `Tabla Sueldo[Sueldo]` | `Sueldo_promedio`, `Grupo_sueldo` | Página Sueldos + barras por rango |

---

## 6. Hallazgos de calidad de datos
- ✅ `ID Empleado` único y presente en las 3 tablas (194/194).
- ⚠️ `Departamento` con espacios sobrantes → corregido con `Trim`.
- ℹ️ 3 estados concentran ~93 % de los empleados; el resto tiene 1 persona cada uno.

---

## 7. Cómo usar el proyecto
1. Clona el repositorio.
2. Abre `Reporte_DAX.pbix` con **Power BI Desktop**.
3. Si Power BI pide la ruta de origen: *Inicio → Transformar datos → Configuración del origen de datos* y apunta a `Datos Empleados.xlsx`.
4. Actualiza el modelo y navega con los botones del menú lateral.

---

## 🛠️ Herramientas
Power BI Desktop · Power Query · DAX · Microsoft Excel

## 👩‍💻 Autora
**Diana Carolina Gonzalez** · [@Diana1295Dev](https://github.com/Diana1295Dev)
