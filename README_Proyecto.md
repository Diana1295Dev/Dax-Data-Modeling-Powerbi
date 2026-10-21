# 👥 Dashboard de Recursos Humanos · Power BI

Reporte analítico de RRHH construido en **Power BI** a partir de un Excel con tres tablas (empleados, evaluaciones y sueldos). Flujo completo: **Excel → Power Query → modelo relacional → DAX → reporte de 3 páginas**.

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/Lenguaje-DAX-0078D4)
![Excel](https://img.shields.io/badge/Fuente-Excel-217346?logo=microsoftexcel&logoColor=white)

📘 **Detalle técnico (Power Query, modelo, fórmulas DAX y capturas):** [README_Modelamiento.md](README_Modelamiento.md)

![Tablero · Total Colaboradores](images/05_Colaboradores.png)

---

## 📁 Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Datos Empleados.xlsx` | Fuente de datos (3 hojas) |
| `Reporte_DAX.pbix` | Modelo y reporte de RRHH |
| `Modelamiento_y_DAX.pbix` | Reporte adicional de finanzas (ingresos, gastos, utilidad) |
| `README_Modelamiento.md` | Modelamiento y DAX con capturas |
| `images/` | Capturas reales de Power BI y diagrama de flujo |

---

## 1. Fuente de datos (Excel)

`Datos Empleados.xlsx` tiene **3 hojas con 194 registros cada una**, relacionadas por `ID Empleado` (único y presente en las tres).

| Hoja | Columnas | Contenido |
|---|---|---|
| **Tabla Empleados** | ID Empleado, Nombre Empleado, Estado, Nacimiento, Género, Departamento, Posición, Nombre Jefe | Datos demográficos y organizacionales |
| **Tabla Evaluacion** | ID Empleado, Evaluación | Puntaje de desempeño (5,01 – 9,93; promedio ≈ 7,56) |
| **Tabla Sueldo** | ID Empleado, Sueldo | Sueldo (26.536 – 144.000; promedio ≈ 74.722) |

**Perfil:** 108 mujeres y 86 hombres · 6 departamentos · 8 posiciones · empleados concentrados en Texas (77), California (53) y Massachusetts (51).

---

## 2. Proceso de modelado

1. **Power Query:** promover encabezados y cambiar tipos en las 3 tablas; en *Tabla Empleados* además se divide y recombina el nombre (`Apellido, Nombre` → `Nombre Apellido`) y se traduce `Género` (Male → Hombre, Female → Mujer).
2. **Modelo:** *Tabla Empleados* se relaciona 1:1 con *Tabla Evaluacion* y *Tabla Sueldo* por `ID Empleado`, con filtro en ambas direcciones.
3. **DAX:** 3 medidas (`Total_empleados`, `Evaluacion_promedio`, `Sueldo_promedio`) y 4 columnas calculadas (`Edad`, `Grupo edad`, `Grupo_evaluacion`, `Grupo_sueldo`).
4. **Reporte:** 3 páginas con tarjetas KPI, mapas, gráficos y tabla de detalle.

![Modelo de datos](images/03_Modelo.png)

Fórmulas DAX principales:

```dax
Total_empleados     = COUNTROWS ( 'Tabla Empleados' )
Evaluacion_promedio = AVERAGE ( 'Tabla Evaluacion'[Evaluación] )
Sueldo_promedio     = AVERAGE ( 'Tabla Sueldo'[Sueldo] )
Edad                = INT ( YEARFRAC ( 'Tabla Empleados'[Nacimiento], TODAY () ) )
```

Las columnas de agrupación (`Grupo edad`, `Grupo_evaluacion`, `Grupo_sueldo`) están completas en [README_Modelamiento.md](README_Modelamiento.md).

---

## 3. Reporte (3 páginas)

| Página | Pregunta que responde | Visuales principales |
|---|---|---|
| **Total Colaboradores** | ¿Cuántos somos y cómo estamos compuestos? | Dona por género, mapa por estado, columnas por departamento, barras por rango de edad / evaluación / sueldo |
| **Análisis de Sueldos** | ¿Cómo se distribuye la compensación? | Dona por género, mapa, columnas por departamento, dispersión Edad vs Sueldo |
| **Análisis de Evaluación** | ¿Cómo es el desempeño y dónde se concentra? | Barras por jefe, columnas por género y por departamento, mapa por estado |

Las tres incluyen una tabla de detalle (Nombre_Empleado, Posición, Estado, Edad, Evaluación, Sueldo).

| Análisis de Sueldos | Análisis de Evaluación |
|---|---|
| ![Sueldos](images/06_Sueldos.png) | ![Evaluación](images/07_Evaluacion.png) |

---

## 4. Match Excel ↔ Modelo ↔ Reporte

| Dato en Excel | Campo en el modelo | Cálculo DAX | Dónde se ve |
|---|---|---|---|
| `ID Empleado` | `Tabla Empleados[ID Empleado]` | `Total_empleados` (`COUNTROWS`) | Tarjeta de las 3 páginas |
| `Nacimiento` | `Tabla Empleados[Nacimiento]` | `Edad`, `Grupo edad` | Tarjeta de edad, barras por rango, dispersión |
| `Departamento`, `Estado`, `Género`, `Nombre Jefe` | `Tabla Empleados` | — (ejes y leyendas) | Columnas, mapas, donas, barras por jefe |
| `Evaluación` | `Tabla Evaluacion[Evaluación]` | `Evaluacion_promedio`, `Grupo_evaluacion` | Página Evaluación y barras por rango |
| `Sueldo` | `Tabla Sueldo[Sueldo]` | `Sueldo_promedio`, `Grupo_sueldo` | Página Sueldos y barras por rango |

---

## 5. Cómo usar el proyecto
1. Clona el repositorio.
2. Abre `Reporte_DAX.pbix` con **Power BI Desktop**.
3. Si pide la ruta de origen: *Transformar datos → Configuración del origen de datos* y apunta a `Datos Empleados.xlsx`.
4. Navega con los botones superiores.

**Herramientas:** Power BI Desktop · Power Query · DAX · Excel
**Autora:** [@Diana1295Dev](https://github.com/Diana1295Dev)
