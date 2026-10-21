<div align="center">

# 👥 Dashboard de RRHH en Power BI

### Modelamiento de datos · Power Query · Medidas DAX · KPIs

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-3%20medidas%20·%204%20columnas-0078D4?style=for-the-badge)
![Excel](https://img.shields.io/badge/Fuente-Excel-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white)
![Modelo](https://img.shields.io/badge/Modelo-3%20tablas%20·%202%20relaciones-6C5CE7?style=for-the-badge)

**Análisis de 194 colaboradores: composición, sueldos y evaluación de desempeño.**

<img src="images/05_Colaboradores.png" alt="Tablero de RRHH" width="900"/>

📘 *Guía técnica de `Reporte_DAX.pbix`. Las capturas son reales, tomadas de Power BI Desktop.*

[← Volver al README del proyecto](README_Dashboard_Financiero.md)

</div>

---

## 📑 Contenido

| # | Sección | Qué encontrarás |
|---|---|---|
| 1 | [🔄 Flujo de trabajo](#-flujo-de-trabajo) | De Excel al tablero |
| 2 | [🧹 Power Query](#-power-query) | Transformación de las 3 tablas |
| 3 | [🧩 Modelo de datos](#-modelo-de-datos) | Tablas y relaciones |
| 4 | [🧮 Fórmulas DAX](#-fórmulas-dax) | Medidas y columnas calculadas con captura |
| 5 | [📊 Tablero](#-tablero-páginas-y-kpis) | Las 3 páginas y sus KPIs |

---

## 🔄 Flujo de trabajo

<p align="center">
<img src="images/01_Flujo.svg" alt="Flujo de trabajo" width="900"/>
</p>

| Paso | Qué se hizo |
|---|---|
| 1️⃣ Obtener datos | Importar `Datos Empleados.xlsx` (3 hojas, 194 filas cada una) |
| 2️⃣ Transformar | Power Query: promover encabezados, tipos, dividir y combinar nombre, traducir género |
| 3️⃣ Modelar | 2 relaciones 1:1 por `ID Empleado` |
| 4️⃣ Calcular | 3 medidas y 4 columnas calculadas en DAX |
| 5️⃣ Visualizar | 3 páginas: Colaboradores, Sueldos y Evaluación |

---

## 🧹 Power Query

Las tres hojas se cargan como consultas independientes (`Consultas [3]`).

### 👥 Tabla Empleados

`Origen → Navegación → Encabezados promovidos → Tipo cambiado → Dividir columna por delimitador → Tipo cambiado1 → Columnas combinadas → Valor reemplazado → Valor reemplazado1`

| Transformación | Resultado |
|---|---|
| ✂️ *Dividir columna* + *Columnas combinadas* | `Spirea, Kelley` → `Kelley Spirea` en `Nombre_Empleado` |
| 🔤 *Valor reemplazado* | `Género`: `Male` → `Hombre`, `Female` → `Mujer` |

<p align="center">
<img src="images/02_PowerQuery_Empleados.png" alt="Power Query · Tabla Empleados" width="850"/>
</p>

### ⭐ Tabla Evaluacion y 💰 Tabla Sueldo

`Origen → Navegación → Encabezados promovidos → Tipo cambiado` (`ID Empleado` como entero; `Evaluación` como decimal; `Sueldo` como entero).

<table>
<tr>
<td width="50%" align="center"><img src="images/02b_PowerQuery_Evaluacion.png" alt="Power Query · Tabla Evaluacion"/><br/><sub>Tabla Evaluacion</sub></td>
<td width="50%" align="center"><img src="images/02c_PowerQuery_Sueldo.png" alt="Power Query · Tabla Sueldo"/><br/><sub>Tabla Sueldo</sub></td>
</tr>
</table>

---

## 🧩 Modelo de datos

**Tabla Empleados** es la tabla central. Se conecta con **Tabla Evaluacion** y **Tabla Sueldo** mediante relaciones **1 : 1** por `ID Empleado`, con filtro cruzado en **ambas direcciones** (icono ◄► en las líneas de relación).

<p align="center">
<img src="images/03_Modelo.png" alt="Vista de modelo" width="850"/>
</p>

<details>
<summary><b>📸 Vista de tabla (datos transformados y columnas calculadas)</b></summary>
<br/>
<p align="center"><img src="images/03b_Vista_Tabla.png" alt="Vista de tabla" width="850"/></p>
</details>

> El modelo tiene **3 tablas, 3 medidas y 2 relaciones**.

---

## 🧮 Fórmulas DAX

Todas las fórmulas son las reales del `.pbix`. Despliega cada una para ver su captura.

### 📏 Medidas (KPIs)

```dax
Total_empleados     = COUNTROWS ( 'Tabla Empleados' )
Evaluacion_promedio = AVERAGE ( 'Tabla Evaluacion'[Evaluación] )
Sueldo_promedio     = AVERAGE ( 'Tabla Sueldo'[Sueldo] )
```

<details>
<summary><b>📸 Capturas de las medidas</b></summary>
<br/>

![Total_empleados](images/04a_DAX_Total_empleados.png)
![Evaluacion_promedio](images/04d_DAX_Evaluacion_promedio.png)
![Sueldo_promedio](images/04f_DAX_Sueldo_promedio.png)

</details>

### 🧱 Columnas calculadas (segmentación)

**Edad**
```dax
Edad = INT ( YEARFRAC ( 'Tabla Empleados'[Nacimiento], TODAY () ) )
```

**Grupo edad**
```dax
Grupo edad =
IF ( 'Tabla Empleados'[Edad] <= 30, "20-30",
IF ( 'Tabla Empleados'[Edad] <= 40, "30-40",
IF ( 'Tabla Empleados'[Edad] <= 50, "40-50",
IF ( 'Tabla Empleados'[Edad] <= 60, "50-60",
IF ( 'Tabla Empleados'[Edad] <= 70, "60-70",
"70+" )))))
```

**Grupo_evaluacion**
```dax
Grupo_evaluacion =
IF ( 'Tabla Evaluacion'[Evaluación] <= 6, "0-6",
IF ( 'Tabla Evaluacion'[Evaluación] <= 7, "6-7",
IF ( 'Tabla Evaluacion'[Evaluación] <= 8, "7-8",
IF ( 'Tabla Evaluacion'[Evaluación] <= 9, "8-9",
"9 o más" ))))
```

**Grupo_sueldo**
```dax
Grupo_sueldo =
IF ( 'Tabla Sueldo'[Sueldo] <= 40000, "0-40 mil",
IF ( 'Tabla Sueldo'[Sueldo] <= 60000, "40-60 mil",
IF ( 'Tabla Sueldo'[Sueldo] <= 80000, "60-80 mil",
IF ( 'Tabla Sueldo'[Sueldo] <= 100000, "80-100 mil",
"100 mil o más" ))))
```

<details>
<summary><b>📸 Capturas de las columnas calculadas</b></summary>
<br/>

![Edad](images/04b_DAX_Edad.png)
![Grupo edad](images/04c_DAX_Grupo_edad.png)
![Grupo_evaluacion](images/04e_DAX_Grupo_evaluacion.png)
![Grupo_sueldo](images/04g_DAX_Grupo_sueldo.png)

</details>

### 🔗 Qué fórmula alimenta qué KPI

| KPI / visual | Fórmula | Funciones DAX |
|---|---|---|
| Total empleados (tarjeta, donut, mapa, columnas, barras) | `Total_empleados` | `COUNTROWS` |
| Evaluación promedio (tarjeta, columnas, barras) | `Evaluacion_promedio` | `AVERAGE` |
| Sueldo promedio (tarjeta, tabla) | `Sueldo_promedio` | `AVERAGE` |
| Promedio de edad (tarjeta) y dispersión Edad vs Sueldo | `Edad` | `INT`, `YEARFRAC`, `TODAY` |
| Empleados por rango de edad | `Grupo edad` | `IF` anidado |
| Empleados por rango de evaluación | `Grupo_evaluacion` | `IF` anidado |
| Empleados por rango de sueldo | `Grupo_sueldo` | `IF` anidado |

> Las tarjetas **Sueldo** (suma) y **Evaluación** (mínimo y máximo) no usan medidas DAX: son agregaciones de la columna hechas directamente en el visual.

---

## 📊 Tablero: páginas y KPIs

Las tres páginas comparten diseño: botones de navegación arriba, título «Informe Recursos Humanos», tarjetas KPI y visuales de detalle. Los visuales se filtran entre sí.

### 1️⃣ Total Colaboradores

<table>
<tr>
<td align="center"><h3>194</h3>Empleados</td>
<td align="center"><h3>7,56</h3>Evaluación promedio</td>
<td align="center"><h3>74,72 mil</h3>Sueldo promedio</td>
<td align="center"><h3>45,72</h3>Promedio de edad</td>
</tr>
</table>

Visuales: total por género (dona), por estado (mapa), por departamento (columnas), por `Grupo edad`, `Grupo_evaluacion` y `Grupo_sueldo` (barras) y tabla de detalle.

<p align="center"><img src="images/05_Colaboradores.png" alt="Total Colaboradores" width="900"/></p>

### 2️⃣ Análisis de Sueldos

<table>
<tr>
<td align="center"><h3>194</h3>Empleados</td>
<td align="center"><h3>74,72 mil</h3>Sueldo promedio</td>
<td align="center"><h3>14 mill.</h3>Suma de sueldo</td>
</tr>
</table>

Visuales: sueldo por género (dona), por estado (mapa), por departamento (columnas), dispersión *Sueldo y Edad por Empleado* y tabla de detalle.

<p align="center"><img src="images/06_Sueldos.png" alt="Análisis de Sueldos" width="900"/></p>

### 3️⃣ Análisis de Evaluación

<table>
<tr>
<td align="center"><h3>194</h3>Empleados</td>
<td align="center"><h3>7,56</h3>Evaluación promedio</td>
<td align="center"><h3>5,01</h3>Mínima</td>
<td align="center"><h3>9,93</h3>Máxima</td>
</tr>
</table>

Visuales: evaluación por jefe (barras), por género (columnas), por departamento (columnas), mapa por estado y tabla de detalle.

<p align="center"><img src="images/07_Evaluacion.png" alt="Análisis de Evaluación" width="900"/></p>

---

<div align="center">

**Herramientas:** Power BI Desktop · Power Query · DAX · Excel
**Autora:** [@Diana1295Dev](https://github.com/Diana1295Dev)

</div>
