# 🚀 Proyecto de Ingeniería de Datos End-to-End con Databricks y dbt

Este repositorio contiene una implementación práctica de una arquitectura de ingeniería de datos moderna de extremo a extremo, utilizando **Databricks**, **dbt (Data Build Tool)** y siguiendo el patrón de la **Arquitectura Medallion (Bronze, Silver, Gold)**. El proyecto aborda desafíos comunes del mundo real, como la ingesta incremental, transformaciones complejas, control de cambios, construcción dinámica de modelos dimensionales y exposición de datos para análisis.

---

## 🌐 Arquitectura del Proyecto: Medallion al 100%

El pipeline de datos sigue estrictamente la Arquitectura Medallion, que es una de las mejores prácticas en entornos de Lakehouse.

**Flujo de Datos:**  
`CSV Files → Bronze Layer (Databricks Autoloader) → Silver Layer (DLT con CDC y validaciones) → Gold Layer (modelo dimensional) → dbt (modelos analíticos) → SQL Warehouse (BI y consumo)`

### Propósitos por capa:
- **Bronze:** Almacén inmutable de datos brutos.
- **Silver:** Datos limpios, enriquecidos y con CDC.
- **Gold:** Datos listos para análisis (modelo dimensional).

---

## 🛠️ Configuración y Prerrequisitos

1. **Cuenta de Databricks**: Se recomienda la Edición Gratuita.
2. **Unity Catalog**: Habilitado por defecto para gobernanza.
3. **Volumes**: Para las capas `raw`, `bronze`, `silver`, `gold`.
4. **dbt Cloud**: Para conexión y modelado analítico.

**Estructura del almacenamiento**  
Se usan esquemas como: `workspace.raw`, `workspace.bronze`, etc.  
Los archivos CSV (bookings, flights, customers, airports) se cargan en `workspace.raw.rawvolume.rawdata`.

---

## ⚙️ Capas del Pipeline de Datos

### 1. 🗄️ Capa Bronze – Ingesta Incremental Inteligente

- **Objetivo:** Ingestar archivos CSV dinámicamente.
- **Técnicas usadas:**
  - Databricks Autoloader + Structured Streaming.
  - Esquema evolutivo (`rescue` o `addNewColumns`).
  - Notebook parametrizable para cada fuente (`src`).
  - Orquestación con Databricks Workflows.
- **Salida:** Tablas Delta en `/Volumes/workspace/bronze/bronzevolume/{src}/data`.

---

### 2. ✨ Capa Silver – Limpieza, CDC y Validaciones (Lakeflow / DLT)

- **Objetivo:** Limpieza, transformaciones y control de cambios (SCD Type 1).
- **Técnicas usadas:**
  - Pipeline unificado DLT (una sola definición por restricciones CE).
  - `@dlt.expect_all_or_drop` para reglas de calidad.
  - `dlt.create_auto_cdc_flow` para manejar CDC.
  - Problemas de streaming resueltos con `ignore_changes`.
- **Salida:** Tablas limpias, enriquecidas y versionadas.

---

### 3. 🥇 Capa Gold – Esquema en Estrella con Constructores Dinámicos

- **Objetivo:** Modelo dimensional con tablas de hechos y dimensiones.
- **Constructores creados:**
  - **Dimensiones**: Claves subrogadas con `monotonically_increasing_id()`, manejo de carga incremental y full.
  - **Hechos**: Cálculo incremental, joins con dimensiones, UPSERT condicional vía `MERGE`.
- **Errores clave solucionados:**
  - `Cannot perform merges multiple source rows matched...` al incluir `booking_date` como clave compuesta.
- **Salida:** `DimPassengers`, `DimFlights`, `DimAirports`, `FactBookings`.

---

### 4. 📈 Conexión con dbt – Modelado Analítico y Exposición

- **Objetivo:** Modelado modular con gobernanza y versionado.
- **Pasos implementados:**
  - Conexión entre dbt Cloud y Databricks SQL Warehouse.
  - Creación de modelos dbt (`my_first_dbt_model`) usando SQL.
  - Materialización como `table` o `view` según necesidad.
  - Ejecución con `dbt run`.
- **Resultado:** Modelos analíticos gobernados listos para herramientas de BI.

---

## 🧠 Aprendizajes y Retos Clave Enfrentados

### Limitaciones y desafíos técnicos:
- Solo un DLT activo en Edición Gratuita → lógica consolidada.
- Tiempos de arranque de clusters.
- Errores por rutas incorrectas al crear tablas externas.
- Eliminación de `_rescued_data`, manejo de tipos y nulos.

### Dinamismo:
- Notebooks parametrizables para Bronze, Silver y Gold.
- Uso intensivo de f-strings, lógica condicional, y generación dinámica de SQL.

### MERGE en Hechos:
- Se resolvió el error de múltiples matches asegurando claves compuestas únicas.

### Depuración:
- Cada error fue una oportunidad para profundizar en el funcionamiento interno de Databricks y dbt.

---

## 🧰 Herramientas y Tecnologías Utilizadas

- **Databricks**:
  - Unity Catalog, Volumes, SQL Warehouse
  - Autoloader, Spark Structured Streaming
  - Lakeflow (DLT), Workflows

- **Apache Spark / Delta Lake**

- **dbt Cloud**

- **Lenguajes**:
  - Python / PySpark
  - SQL

---

## 🎯 ¿Por qué es importante este proyecto?

Este proyecto va más allá de un ejemplo:  
Es una **plantilla extensible y realista** para entornos empresariales. Refleja:

- Buenas prácticas de arquitectura Lakehouse.
- Automatización y versionado.
- Enfoque dinámico, escalable y profesional.

---

> _“La ingeniería de datos no se trata solo de mover datos. Se trata de construir sistemas resilientes, automatizados y gobernados que evolucionen con el negocio.”_

---

## 🤖 Generación del README

Este README fue creado con el apoyo de una IA generativa, guiada paso a paso para documentar el proyecto de forma clara y estructurada.

## 