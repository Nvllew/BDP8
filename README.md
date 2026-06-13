<div align="center">
  <img src="https://images.seeklogo.com/logo-png/7/2/ipn-logo-png_seeklogo-73340.png" align="left" width="90" alt="Logo IPN" />
  <img src="https://www.escom.ipn.mx/x/ee2021/images/galeriaEE2021/escudo_ESCOM.png" align="right" width="120" alt="Logo ESCOM" />
  
  <h2>Instituto Politécnico Nacional</h2>
  <h2>Escuela Superior de Cómputo</h2>

  <br><br><br><br>

  <b>Práctica 4</b><br>
  <b>Normalización</b>

  <br><br><br>

  <b>Alumno:</b><br>
  Olate Tomás Kevin Saúl

  <br><br>

  <b>Profesor:</b><br>
  Gabriel Hurtado Avilés

  <br><br>

  <b>Unidad de Aprendizaje:</b><br>
  Bases de Datos

  <br><br>

  <b>Grupo:</b><br>
  3BV1

  <br><br>
</div>

---# Práctica de Normalización de Bases de Datos 

Este proyecto implementa un pipeline automatizado para la transformación y normalización de conjuntos de datos planos (CSV) hacia modelos relacionales estructurados en **1FN, 2FN y 3FN**. Todo el flujo de procesamiento, generación de esquemas (DDL) e inserción masiva (DML) se ejecuta de forma programática utilizando Python y contenedores Docker.

## Objetivo del Proyecto

Demostrar la capacidad de analizar dependencias funcionales y transitivas en bases de datos desnormalizadas, diseñando una arquitectura relacional óptima para garantizar la integridad referencial y evitar anomalías de inserción, actualización y borrado. 

El código extrae, transforma y carga (ETL) los datos originales, generando automáticamente los scripts SQL necesarios para reconstruir las bases de datos en un motor MySQL 8.0+.

## Tecnologías Utilizadas

* **Lenguaje:** Python 3.10
* **Análisis de Datos:** Pandas
* **ORM y Conexión:** SQLAlchemy, PyMySQL
* **Infraestructura:** Docker & Docker Compose
* **Motor de Base de Datos:** MySQL 8.0

## Datasets Procesados y Estrategia de Normalización

El pipeline procesa de forma secuencial tres escenarios distintos:

1. **Dataset 1: Catálogo de Entretenimiento (Netflix)**
   * **Problema:** Múltiples columnas multivaluadas (actores, directores, países, categorías) en una sola cadena de texto.
   * **Solución:** Aplicación estricta de 1FN mediante la separación atómica de valores, creación de catálogos en 3FN y generación de tablas puente (Relaciones M:N) para vincular entidades sin redundancia.

2. **Dataset 2: Transacciones de E-commerce (ERP/POS)**
   * **Problema:** Dependencias parciales y transitivas entre productos, clientes y recibos.
   * **Solución:** Separación del modelo en una arquitectura transaccional "Cabecera-Detalle" (Invoices e Invoice_Details) para optimizar el control de inventario y registro de ventas.

3. **Dataset 3: Registros Médicos (Hospital)**
   * **Problema:** Repetición demográfica masiva de pacientes y datos hospitalarios por cada encuentro clínico.
   * **Solución:** Extracción de datos demográficos y de infraestructura hacia catálogos maestros (2FN/3FN), dejando una tabla central de encuentros estrictamente transaccional.

## Estructura del Repositorio

El proyecto sigue una estructura modular orientada a la limpieza y escalabilidad:

```text
normalizacion-db/
├── data/
│   └── raw/                   # Datasets originales planos (.csv)
├── docs/                      # Documentación y diagramas ER
├── normalized/                # CSVs atómicos generados por el script
├── scripts/
│   ├── normalize_dataset1.py  # Lógica ETL Dataset 1
│   ├── normalize_dataset2.py  # Lógica ETL Dataset 2
│   ├── normalize_dataset3.py  # Lógica ETL Dataset 3
│   └── utils.py               # Funciones auxiliares y conexión DB
├── sql/
│   ├── ddl/                   # Scripts auto-generados de creación (CREATE TABLE)
│   └── dml/                   # Scripts auto-generados de inserción (INSERT INTO)
├── docker-compose.yml         # Orquestación de infraestructura
├── Dockerfile                 # Definición de la imagen de Python
└── requirements.txt           # Dependencias del proyecto

```

## Instrucciones de Ejecución

El proyecto está contenerizado, por lo que no es necesario instalar dependencias locales más allá de tener Docker Desktop ejecutándose en el equipo.

**1. Clonar el repositorio y acceder al directorio:**

```bash
git clone https://github.com/Nvllew/Database_Practices.git
cd normalizacion-db

```

**2. Construir e iniciar la infraestructura:**
Este comando levantará el motor de MySQL y ejecutará secuencialmente los tres scripts de normalización.

```bash
docker compose up --build -d

```

**3. Monitorear el proceso:**
Para visualizar en tiempo real el procesamiento de los datos y la exportación a la base de datos:

```bash
docker logs app_normalizacion -f

```

**4. Detener la infraestructura:**
Una vez generados los archivos, puedes apagar los contenedores de forma segura con:

```bash
docker compose down

```

## Resultados Generados

Al finalizar la ejecución, el pipeline habrá creado automáticamente:

1. **Modelos Físicos CSV:** Archivos atómicos normalizados en la carpeta `normalized/`.
2. **Esquemas DDL:** Archivos `_schema.sql` consolidados por dataset en la carpeta `sql/ddl/`.
3. **Poblado DML:** Archivos `_data.sql` con inserciones de ejemplo en la carpeta `sql/dml/`.
4. **Base de Datos Viva:** Una instancia de MySQL corriendo en el puerto `3307` con las tres bases de datos completamente estructuradas y pobladas.

---


