
---
### Primera Forma Normal (1FN) - Dataset 1: Netflix Movies and TV Shows

#### Estructura original 
La estructura original es una tabla plana que consta de 8,807 registros y 12 columnas. Dado nuestro ánalisis, se detectaron violaciones a la 1FN debido a que contiene campos que almacenan conjuntos de datos y múltiples valores concatenados.

Específicamente, las columnas `director`, `cast`, `country` y `listed_in` almacenan listas de valores separados por comas dentro de una misma celda, rompiendo la regla fundamental de que cada celda debe contener un único valor atómico. Además, la tabla carece de un diseño que prevenga la repetición de grupos de información.

#### Estructura resultante
Para alcanzar la 1FN sin generar una sola tabla con millones de filas redundantes, se crearon nuevas tablas dependientes y se definieron claves primarias (PK) compuestas. La estructura resultante es la siguiente:

* **Tabla `shows**`: `show_id` (PK), `type`, `title`, `release_year`, `rating`, `duration`, `description`, `date_added`.
* **Tabla `show_cast**`: `show_id` (PK), `actor_name` (PK).
* **Tabla `show_director**`: `show_id` (PK), `director_name` (PK).
* **Tabla `show_country**`: `show_id` (PK), `country_name` (PK).
* **Tabla `show_category**`: `show_id` (PK), `category_name` (PK).

#### Explicación de las transformaciones realizadas
El proceso consistió en identificar todas las columnas con listas separadas por comas y dividirlas en registros separados. Dado que intentar mantener esta división en la tabla original provocaría la repetición innecesaria de datos como el título o la descripción, se optó por crear tablas independientes para cada atributo multivaluado.

En estas nuevas tablas, se definió una clave primaria compuesta integrando el `show_id` (para mantener la relación con el título original) y el valor atómico extraído (por ejemplo, el nombre del actor). Esto garantiza que cada registro sea único y que cada celda contenga un valor indivisible, cumpliendo estrictamente con los requisitos de la 1FN.

#### Ejemplos de datos antes y después
**Antes (Tabla Desnormalizada con grupos repetitivos):** 
| show_id | title | cast | country | listed_in |
| --- | --- | --- | --- | --- |
| s2 | Blood & Water | Ama Qamata, Khosi Ngema | South Africa | TV Dramas, TV Mysteries |

**Después (Tablas Transformadas en 1FN):** 
*Tabla: `shows` (Datos atómicos únicos)*
| show_id (PK) | title |
| --- | --- |
| s2 | Blood & Water |

*Tabla: `show_cast` (Separación de registros múltiples)*
| show_id (PK) | actor_name (PK) |
| --- | --- |
| s2 | Ama Qamata |
| s2 | Khosi Ngema |

*Tabla: `show_category` (Separación de registros múltiples)*
| show_id (PK) | category_name (PK) |
| --- | --- |
| s2 | TV Dramas |
| s2 | TV Mysteries |

---

### Segunda Forma Normal (2FN) - Dataset 1: Netflix Movies and TV Shows

#### Estructura en 1FN 

Al finalizar la etapa anterior, contábamos con la siguiente estructura:
* **`shows`**: `show_id` (PK), `type`, `title`, `release_year`, `rating`, `duration`, `description`, `date_added`.
* **`show_cast`**: `show_id` (PK), `actor_name` (PK).
* **`show_director`**: `show_id` (PK), `director_name` (PK).
* **`show_country`**: `show_id` (PK), `country_name` (PK).
* **`show_category`**: `show_id` (PK), `category_name` (PK).

#### Identificación de dependencias parciales
La regla de la 2FN dicta que una tabla debe estar en 1FN y que todos los atributos no clave deben depender de *toda* la clave primaria compuesta, no solo de una parte de ella .

En nuestra estructura 1FN, las tablas dependientes (`show_cast`, `show_director`, etc.) usan cadenas de texto completas como parte de su clave primaria compuesta (ej. `actor_name`). Esto presenta una **dependencia parcial conceptual y de diseño relacional**:

* Un actor, un director o un país son entidades del mundo real que existen independientemente del `show`. Su identidad (el nombre) depende de la entidad "Actor", no de la combinación `(show_id, actor_name)`.


* Mantener cadenas de texto (`VARCHAR`) repetidas en claves compuestas genera ineficiencia en las consultas y anomalías si quisiéramos agregar más atributos propios del actor en el futuro (ej. fecha de nacimiento), ya que esos atributos dependerían solo del actor y no del `show_id`.

#### Estructura resultante (tablas en 2FN)

Para resolver esto y lograr un modelo relacional robusto, separamos los atributos multivaluados en sus propias tablas "catálogo" (maestras) con identificadores numéricos únicos (PK simples). Luego, resolvemos la relación de muchos-a-muchos (M:N) estableciendo tablas intermedias (puente) con claves foráneas (FK) .

* **Tabla Principal:**
* `shows` (`show_id` PK, `type`, `title`, `release_year`, `rating`, `duration`, `description`, `date_added`)

* **Tablas Catálogo (Entidades Independientes):**
* `actors` (`actor_id` PK, `full_name`)
* `directors` (`director_id` PK, `full_name`)
* `countries` (`country_id` PK, `country_name`)
* `categories` (`category_id` PK, `category_name`)

* **Tablas Puente (Relación M:N, PK Compuesta sin dependencias parciales):**
* `show_actor` (`show_id` PK/FK, `actor_id` PK/FK)
* `show_director` (`show_id` PK/FK, `director_id` PK/FK)
* `show_country` (`show_id` PK/FK, `country_id` PK/FK)
* `show_category` (`show_id` PK/FK, `category_id` PK/FK)


#### Ejemplos de datos transformados

**Antes (Estructura en 1FN con texto repetitivo en PK):**

*Tabla: `show_cast*`

| show_id (PK) | actor_name (PK) |
| --- | --- |
| s2 | Ama Qamata |
| s2 | Khosi Ngema |
| s5 | Khosi Ngema |

**Después (Estructura en 2FN con catálogos y tablas puente):**

*Tabla: `actors` (Entidad independiente)*

| actor_id (PK) | full_name |
| --- | --- |
| 1 | Ama Qamata |
| 2 | Khosi Ngema |

*Tabla: `show_actor` (Tabla puente - Dependencia total de la PK compuesta)*

| show_id (PK/FK) | actor_id (PK/FK) |
| --- | --- |
| s2 | 1 |
| s2 | 2 |
| s5 | 2 |

Aquí tienes la documentación final para culminar el proceso teórico del **Dataset 1 (Netflix)**, llevando la base de datos hasta la Tercera Forma Normal (3FN) con todos los elementos que solicita la rúbrica .

---

### Tercera Forma Normal (3FN) - Dataset 1: Netflix Movies and TV Shows

#### Estructura en 2FN 

Al iniciar esta fase, nuestra base de datos cuenta con catálogos independientes y tablas puente, eliminando las dependencias parciales y la redundancia de listas:

* **Tabla Principal:** `shows` (`show_id` PK, `type`, `title`, `release_year`, `rating`, `duration`, `description`, `date_added`)
* **Catálogos:** `actors`, `directors`, `countries`, `categories`.
* **Tablas Puente:** `show_actor`, `show_director`, `show_country`, `show_category`.

#### Identificación de dependencias transitivas

La 3FN exige que la base de datos esté en 2FN y que se eliminen las dependencias transitivas.

* **Problema:** En la tabla `shows`, los atributos `type` (Movie, TV Show) y `rating` (PG-13, TV-MA, R) son atributos categóricos que se repiten como texto a lo largo de miles de filas .

* **Anomalía:** Aunque parecen depender de `show_id`, en realidad representan dominios de datos independientes. Mantenerlos como texto libre permite errores tipográficos (ej. "Tv Show" vs "TV Show") y dificulta la actualización global de una categoría.

#### Estructura final normalizada (tablas en 3FN)
Para resolver esto, extraemos los atributos categóricos a sus propias tablas maestras y los referenciamos en la tabla principal mediante Claves Foráneas (FK) .

* Se crean las tablas `show_types` y `ratings`.
* La tabla `shows` se actualiza para almacenar únicamente `type_id` y `rating_id` en lugar de las cadenas de texto.

#### Modelo relacional completo con todas las tablas

El esquema relacional final queda compuesto por 12 tablas fuertemente tipadas y relacionadas:

1. `shows`
2. `show_types`
3. `ratings`
4. `actors`
5. `show_actor`
6. `directors`
7. `show_director`
8. `countries`
9. `show_country`
10. `categories`
11. `show_category`

<img width="2625" height="2754" alt="d11" src="https://github.com/user-attachments/assets/2beea8c4-df68-497b-9a43-800586242c3f" />
**Figura 1.1** 1F.
<img width="4000" height="1004" alt="d12" src="https://github.com/user-attachments/assets/ccdfec84-4c10-4625-95c9-4dd6495093d6" />
**Figura 1.2** 2F.
<img width="7344" height="1729" alt="d13" src="https://github.com/user-attachments/assets/34d3e54a-ec4b-4a25-8a94-c37da817fbe1" />
**Figura 1.3** 3F


#### Diccionario de datos de cada tabla

A continuación se detalla el esquema principal que refleja la 3FN. 
| Tabla | Columna | Tipo de Dato | Llaves | Descripción |
| --- | --- | --- | --- | --- |
| **shows** | `show_id` | VARCHAR(10) | PK | Identificador original de Netflix (ej. 's1'). |
|  | `type_id` | INT | FK | Referencia al tipo de contenido. |
|  | `title` | VARCHAR(255) |  | Título oficial de la obra. |
|  | `release_year` | INT |  | Año de lanzamiento. |
|  | `rating_id` | INT | FK | Referencia a la clasificación por edades. |
|  | `duration` | VARCHAR(50) |  | Duración (minutos o temporadas). |
|  | `description` | TEXT |  | Sinopsis del contenido. |
|  | `date_added` | DATE |  | Fecha en que se añadió a la plataforma. |
| **show_types** | `type_id` | INT | PK | Identificador numérico autoincremental. |
|  | `type_name` | VARCHAR(50) | UNIQUE | Formato (ej. 'Movie', 'TV Show'). |
| **ratings** | `rating_id` | INT | PK | Identificador numérico autoincremental. |
|  | `code` | VARCHAR(20) | UNIQUE | Código de clasificación (ej. 'PG-13'). |

#### Ejemplos de datos en el modelo normalizado

Así lucen los datos interactuando sin redundancia textual:

**Tabla: `show_types**`

| type_id (PK) | type_name |
| --- | --- |
| 1 | Movie |
| 2 | TV Show |

**Tabla: `ratings**`

| rating_id (PK) | code |
| --- | --- |
| 1 | PG-13 |
| 2 | TV-MA |

**Tabla Principal: `shows` (Completamente Normalizada)**

| show_id (PK) | type_id (FK) | title | release_year | rating_id (FK) |
| --- | --- | --- | --- | --- |
| s1 | 1 | Dick Johnson Is Dead | 2020 | 1 |
| s2 | 2 | Blood & Water | 2021 | 2 |
| s3 | 2 | Ganglands | 2021 | 2 |

---

### Primera Forma Normal (1FN) - Dataset 2: E-commerce Sales Data

#### Estructura original

La estructura original es una tabla transaccional que consta de aproximadamente 500,000 registros y 8 columnas. Dado nuestro análisis, aunque no contiene columnas con listas separadas por comas (sus valores son atómicos), presenta una violación fundamental a la 1FN: carece de una clave primaria definida que garantice la unicidad de cada registro.

Específicamente, un mismo número de factura (`InvoiceNo`) se repite en múltiples filas si el cliente compró varios productos diferentes, lo que significa que el `InvoiceNo` por sí solo no puede identificar de manera única una fila. La tabla carece de un diseño que prevenga la duplicidad de registros exactos.

#### Estructura resultante

Para alcanzar la 1FN cumpliendo la regla de que cada registro debe ser único, no fue necesario crear nuevas tablas, pero sí se definió una clave primaria (PK) compuesta. La estructura resultante es la siguiente:

* **Tabla `transactions**`: `InvoiceNo` (PK), `StockCode` (PK), `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country`.

#### Explicación de las transformaciones realizadas

El proceso consistió en analizar las dependencias para encontrar un identificador único. Se determinó que la combinación del número de factura (`InvoiceNo`) y el código del producto (`StockCode`) forma una clave candidata sólida.

Al establecer esta clave primaria compuesta, garantizamos que un mismo producto no aparezca listado dos veces en la misma factura como filas separadas, asegurando que cada registro sea único y verificable, cumpliendo estrictamente con los requisitos de la 1FN.

#### Ejemplos de datos antes y después

**Antes (Tabla Desnormalizada sin PK, riesgo de duplicados):** 
| InvoiceNo | StockCode | Description | Quantity | UnitPrice |
| --- | --- | --- | --- | --- |
| 536365 | 85123A | WHITE HANGING HEART T-LIGHT HOLDER | 6 | 2.55 |
| 536365 | 71053 | WHITE METAL LANTERN | 6 | 3.39 |

**Después (Tabla Transformada en 1FN con PK Compuesta):** *Tabla: `transactions` (Garantía de unicidad)*

| InvoiceNo (PK) | StockCode (PK) | Description | Quantity | UnitPrice |
| --- | --- | --- | --- | --- |
| 536365 | 85123A | WHITE HANGING HEART T-LIGHT HOLDER | 6 | 2.55 |
| 536365 | 71053 | WHITE METAL LANTERN | 6 | 3.39 |

---

### Segunda Forma Normal (2FN) - Dataset 2: E-commerce Sales Data

#### Estructura en 1FN

Al finalizar la etapa anterior, contábamos con la siguiente estructura:

* **`transactions`**: `InvoiceNo` (PK), `StockCode` (PK), `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country`.

#### 2. Identificación de dependencias parciales

La regla de la 2FN dicta que una tabla debe estar en 1FN y que todos los atributos no clave deben depender de *toda* la clave primaria compuesta, no solo de una parte de ella .

En nuestra estructura 1FN, varios atributos dependen solo de una fracción de la clave primaria compuesta `(InvoiceNo, StockCode)`. Esto presenta una **dependencia parcial**:

* Los atributos `Description` y `UnitPrice` describen al producto, por lo tanto, dependen exclusivamente de `StockCode`.
* Los atributos `InvoiceDate` y `CustomerID` describen la transacción general, dependiendo exclusivamente de `InvoiceNo`.
* Mantener estos atributos mezclados genera redundancia masiva. El nombre y precio de un producto se repiten cada vez que cualquier cliente lo compra.

#### Estructura resultante (tablas en 2FN)

Para resolver esto y lograr un modelo relacional robusto, separamos las entidades independientes extrayendo los datos de los productos y de las facturas generales en sus propias tablas maestras. Luego, mantenemos una tabla intermedia para el detalle de la cantidad comprada .

* **Tabla de Cabecera (Factura):**
* `invoices` (`InvoiceNo` PK, `InvoiceDate`, `CustomerID`, `Country`)
* **Tabla Catálogo (Productos):**
* `products` (`StockCode` PK, `Description`, `UnitPrice`)
* **Tabla Detalle (Relación M:N, PK Compuesta sin dependencias parciales):**
* `invoice_details` (`InvoiceNo` PK/FK, `StockCode` PK/FK, `Quantity`)


#### Ejemplos de datos transformados

**Antes (Estructura en 1FN con texto repetitivo):**

*Tabla: `transactions*`

| InvoiceNo (PK) | StockCode (PK) | Description | Quantity |
| --- | --- | --- | --- |
| 536365 | 85123A | WHITE HANGING HEART T-LIGHT HOLDER | 6 |
| 536366 | 85123A | WHITE HANGING HEART T-LIGHT HOLDER | 12 |

**Después (Estructura en 2FN con catálogos y tabla de detalle):**

*Tabla: `products` (Entidad independiente)*

| StockCode (PK) | Description | UnitPrice |
| --- | --- | --- |
| 85123A | WHITE HANGING HEART T-LIGHT HOLDER | 2.55 |

*Tabla: `invoice_details` (Tabla de detalle transaccional)*

| InvoiceNo (PK/FK) | StockCode (PK/FK) | Quantity |
| --- | --- | --- |
| 536365 | 85123A | 6 |
| 536366 | 85123A | 12 |

---

### Tercera Forma Normal (3FN) - Dataset 2: E-commerce Sales Data

#### Estructura en 2FN

Al iniciar esta fase, nuestra base de datos cuenta con catálogos independientes y tablas de detalle, eliminando las dependencias parciales:

* **Cabecera:** `invoices` (`InvoiceNo` PK, `InvoiceDate`, `CustomerID`, `Country`)
* **Catálogos:** `products` (`StockCode` PK, `Description`, `UnitPrice`)
* **Detalle:** `invoice_details` (`InvoiceNo` PK/FK, `StockCode` PK/FK, `Quantity`)

#### Identificación de dependencias transitivas

La 3FN exige que la base de datos esté en 2FN y que se eliminen las dependencias transitivas .

* **Problema:** En la tabla `invoices`, el atributo `Country` (País) no describe directamente a la factura (`InvoiceNo`), sino que describe al cliente (`CustomerID`) .


* **Anomalía:** Esto es una cadena de dependencia (`InvoiceNo` $\rightarrow$ `CustomerID` $\rightarrow$ `Country`). Si un cliente cambia de país de residencia, tendríamos que buscar y actualizar el país en todo su historial de facturas pasadas para mantener la consistencia.

#### Estructura final normalizada (tablas en 3FN)

Para resolver esto, extraemos los atributos del cliente a su propia tabla maestra y los referenciamos en la tabla de facturas mediante una Clave Foránea (FK) .

* Se crea la tabla `customers`.
* La tabla `invoices` se actualiza para almacenar únicamente `CustomerID` como clave foránea, eliminando la columna `Country`.

#### 4. Modelo relacional completo con todas las tablas

El esquema relacional final queda compuesto por 4 tablas altamente eficientes y transaccionales:

1. `customers`
2. `products`
3. `invoices`
4. `invoice_details`

<img width="4850" height="579" alt="d21" src="https://github.com/user-attachments/assets/62238ef6-ce63-435d-a48f-4260bb3a63c0" />
**Figura 2.1** 1F.

<img width="6407" height="1308" alt="d22" src="https://github.com/user-attachments/assets/c9b06e45-28b3-4f85-ba67-a100dbb93911" />
**Figura 2.2** 2F.

<img width="6085" height="2033" alt="d23" src="https://github.com/user-attachments/assets/65ffd9e8-6fb6-437c-b797-00bc41e918d6" />
**Figura 2.3** 3F.


#### Diccionario de datos de cada tabla

A continuación se detalla el esquema principal que refleja la 3FN.

| Tabla | Columna | Tipo de Dato | Llaves | Descripción |
| --- | --- | --- | --- | --- |
| **customers** | `CustomerID` | INT | PK | Identificador único del cliente. |
|  | `Country` | VARCHAR(100) |  | País de residencia del cliente. |
| **products** | `StockCode` | VARCHAR(20) | PK | Código único del producto (SKU). |
|  | `Description` | VARCHAR(255) |  | Nombre comercial del producto. |
|  | `UnitPrice` | DECIMAL(10,2) |  | Precio unitario de venta. |
| **invoices** | `InvoiceNo` | VARCHAR(20) | PK | Número único de la factura. |
|  | `InvoiceDate` | DATETIME |  | Fecha y hora de emisión. |
|  | `CustomerID` | INT | FK | Referencia al cliente que realiza la compra. |
| **invoice_details** | `InvoiceNo` | VARCHAR(20) | PK, FK | Referencia a la cabecera de la factura. |
|  | `StockCode` | VARCHAR(20) | PK, FK | Referencia al producto comprado. |
|  | `Quantity` | INT |  | Número de unidades adquiridas. |

#### Ejemplos de datos en el modelo normalizado

Así lucen los datos interactuando sin redundancia transitiva:

**Tabla: `customers**`

| CustomerID (PK) | Country |
| --- | --- |
| 17850 | United Kingdom |
| 13047 | United Kingdom |

**Tabla: `invoices` (Completamente Normalizada)**

| InvoiceNo (PK) | InvoiceDate | CustomerID (FK) |
| --- | --- | --- |
| 536365 | 2010-12-01 08:26:00 | 17850 |
| 536367 | 2010-12-01 08:34:00 | 13047 |

---

### Primera Forma Normal (1FN) - Dataset 3: Hospital Patient Records

#### Estructura original

La estructura original es una tabla clínica plana ("wide format") que consta de 91,713 registros y 85 columnas. Dado nuestro análisis, aunque no contiene columnas con listas separadas por comas (sus valores son atómicos), presenta una violación a los principios de la 1FN: carece de una clave primaria definida formalmente que garantice la unicidad de cada registro a nivel de base de datos.

Específicamente, la tabla mezcla identificadores transaccionales (`encounter_id`) con datos demográficos del paciente e infraestructura del hospital, lo que en un esquema sin restricciones formales permite la duplicidad accidental de filas clínicas.

#### Estructura resultante

Para alcanzar la 1FN cumpliendo la regla de que cada registro debe ser único, no fue necesario crear nuevas tablas en esta fase, pero sí se definió una clave primaria (PK). La estructura resultante es la siguiente:

* **Tabla `encounters_raw**`: `encounter_id` (PK), `patient_id`, `hospital_id`, `age`, `gender`, `bmi`, `icu_id`, `icu_type`, `icu_stay_type`, y demás columnas de signos vitales/diagnósticos.

#### Explicación de las transformaciones realizadas

El proceso consistió en auditar la tabla para confirmar que todos los valores (como las mediciones de `d1_heartrate_max`) fueran estrictamente atómicos (sin arrays ni JSONs anidados). Una vez confirmado, se estableció la columna `encounter_id` como la Clave Primaria (PK).

Al establecer esta clave, garantizamos que el sistema de base de datos rechace cualquier intento de insertar el mismo encuentro clínico dos veces, asegurando que cada registro sea único y verificable.

#### Ejemplos de datos antes y después

**Antes (Tabla Desnormalizada sin PK formal):** | encounter_id | patient_id | age | gender | d1_heartrate_max |
| --- | --- | --- | --- | --- |
| 66154 | 25312 | 68.0 | M | 119.0 |
| 114252 | 59342 | 77.0 | F | 118.0 |

**Después (Tabla Transformada en 1FN con PK definida):** *Tabla: `encounters_raw` (Garantía de unicidad)*

| encounter_id (PK) | patient_id | age | gender | d1_heartrate_max |
| --- | --- | --- | --- | --- |
| 66154 | 25312 | 68.0 | M | 119.0 |
| 114252 | 59342 | 77.0 | F | 118.0 |

---

### Segunda Forma Normal (2FN) - Dataset 3: Hospital Patient Records

#### Estructura en 1FN

Al finalizar la etapa anterior, contábamos con la siguiente estructura:

* **`encounters_raw`**: `encounter_id` (PK), `patient_id`, `hospital_id`, `icu_id`, `age`, `gender`, `ethnicity`, `height`, `icu_type`, y métricas clínicas.

#### Identificación de dependencias parciales

La regla de la 2FN dicta que una tabla debe estar en 1FN y resolver redundancias donde atributos dependan de entidades que no son el núcleo de la transacción .

En nuestra estructura 1FN de tabla ancha, el `encounter_id` identifica el ingreso, pero la tabla arrastra datos que no pertenecen al encuentro, sino al individuo. Esto genera una **dependencia problemática de diseño**:

* Los atributos `age`, `gender`, `ethnicity` y `height` describen exclusivamente al paciente, por lo tanto, dependen lógicamente de `patient_id`.
* Mantener estos atributos mezclados en cada encuentro genera redundancia masiva. Si un paciente ingresa 5 veces al hospital en un año, su fecha de nacimiento, género y etnia se repetirán 5 veces en la base de datos.

#### Estructura resultante (tablas en 2FN)

Para resolver esto y lograr un modelo relacional robusto, separamos la entidad independiente extrayendo los datos demográficos en su propia tabla maestra de pacientes . Luego, mantenemos la tabla de encuentros apuntando a ese paciente.

* **Tabla Catálogo (Pacientes):**
* `patients` (`patient_id` PK, `age`, `gender`, `ethnicity`, `height`)
* **Tabla Principal Transaccional (Encuentros):**
* `encounters` (`encounter_id` PK, `patient_id` FK, `hospital_id`, `icu_id`, `icu_type`, `icu_stay_type`, `bmi`, `weight`, métricas clínicas)

#### Ejemplos de datos transformados

**Antes (Estructura en 1FN con demografía repetitiva):**

*Tabla: `encounters_raw*`

| encounter_id (PK) | patient_id | age | gender | d1_heartrate_max |
| --- | --- | --- | --- | --- |
| 66154 | 25312 | 68.0 | M | 119.0 |
| 99999 | 25312 | 68.0 | M | 125.0 |

**Después (Estructura en 2FN con catálogo de pacientes):**

*Tabla: `patients` (Entidad independiente)*

| patient_id (PK) | age | gender |
| --- | --- | --- |
| 25312 | 68.0 | M |

*Tabla: `encounters` (Tabla transaccional)*

| encounter_id (PK) | patient_id (FK) | d1_heartrate_max |
| --- | --- | --- |
| 66154 | 25312 | 119.0 |
| 99999 | 25312 | 125.0 |

---

### Tercera Forma Normal (3FN) - Dataset 3: Hospital Patient Records

#### Estructura en 2FN

Al iniciar esta fase, nuestra base de datos cuenta con el catálogo de pacientes separado de las transacciones clínicas:

* **Pacientes:** `patients` (`patient_id` PK, `age`, `gender`, `ethnicity`, `height`)
* **Encuentros:** `encounters` (`encounter_id` PK, `patient_id` FK, `hospital_id`, `icu_id`, `icu_type`, `icu_stay_type`, métricas clínicas)

#### Identificación de dependencias transitivas

La 3FN exige que la base de datos esté en 2FN y que se eliminen las dependencias transitivas .

* **Problema:** En la tabla `encounters`, los atributos `icu_type` (tipo de UCI) y `icu_stay_type` no describen directamente al encuentro clínico (`encounter_id`), sino que describen a la Unidad de Cuidados Intensivos (`icu_id`) .

* **Anomalía:** Esto es una cadena de dependencia (`encounter_id` $\rightarrow$ `icu_id` $\rightarrow$ `icu_type`). Si el hospital reclasifica una UCI de "Med-Surg" a "CTICU", se tendrían que actualizar miles de registros históricos de encuentros para reflejar el cambio en esa sala específica.

#### Estructura final normalizada (tablas en 3FN)

Para resolver esto, extraemos los atributos de la infraestructura hospitalaria a sus propias tablas maestras y las referenciamos en la tabla de encuentros mediante Claves Foráneas (FK) .

* Se crean las tablas `hospitals` e `icus`.
* La tabla `encounters` se actualiza para almacenar únicamente `hospital_id` e `icu_id` como claves foráneas, eliminando las columnas descriptivas de texto.

#### Modelo relacional completo con todas las tablas

El esquema relacional final queda compuesto por 4 tablas transaccionales y de catálogo:

1. `patients`
2. `hospitals`
3. `icus`
4. `encounters`
   
<img width="5431" height="579" alt="d31" src="https://github.com/user-attachments/assets/9c6e1532-5af4-4d29-86ed-71300bdccbfb" />
**Figura 3.1** 1F

<img width="4441" height="1308" alt="d32" src="https://github.com/user-attachments/assets/f7b40201-713d-4ce1-944a-0b3976b9bf92" />
**Figura 3.2** 2F

<img width="6558" height="1308" alt="d33" src="https://github.com/user-attachments/assets/81f5dd4d-2d40-49ba-9385-a3e4b6881bfa" />
**Figura 3.3** 3F

#### Diccionario de datos de cada tabla

A continuación se detalla el esquema principal que refleja la 3FN.

| Tabla | Columna | Tipo de Dato | Llaves | Descripción |
| --- | --- | --- | --- | --- |
| **patients** | `patient_id` | INT | PK | Identificador único del paciente. |
|  | `age` | FLOAT |  | Edad biológica del paciente. |
|  | `gender` | VARCHAR(1) |  | Género (M/F). |
| **hospitals** | `hospital_id` | INT | PK | Identificador único del centro médico. |
| **icus** | `icu_id` | INT | PK | Identificador de la Unidad de Cuidados Intensivos. |
|  | `icu_type` | VARCHAR(50) |  | Clasificación de la unidad (ej. CTICU). |
|  | `icu_stay_type` | VARCHAR(50) |  | Tipo de admisión (ej. admit, transfer). |
| **encounters** | `encounter_id` | INT | PK | Identificador único del ingreso clínico. |
|  | `patient_id` | INT | FK | Referencia al paciente ingresado. |
|  | `hospital_id` | INT | FK | Referencia al hospital donde ocurre el ingreso. |
|  | `icu_id` | INT | FK | Referencia a la UCI asignada. |
|  | `d1_heartrate_max` | FLOAT |  | Frecuencia cardíaca máxima en el día 1. |
|  | `hospital_death` | INT |  | Indicador binario de mortalidad (0=Vivo, 1=Fallecido). |

#### Ejemplos de datos en el modelo normalizado

Así lucen los datos interactuando sin redundancia de infraestructura:

**Tabla: `icus**`

| icu_id (PK) | icu_type | icu_stay_type |
| --- | --- | --- |
| 92 | CTICU | admit |
| 90 | Med-Surg ICU | admit |

**Tabla: `encounters` (Completamente Normalizada)**

| encounter_id (PK) | patient_id (FK) | icu_id (FK) | d1_heartrate_max | hospital_death |
| --- | --- | --- | --- | --- |
| 66154 | 25312 | 92 | 119.0 | 0 |
| 114252 | 59342 | 90 | 118.0 | 0 |

---

### Cuadros Comparativos de Normalización

A continuación, se presenta la evolución de la complejidad estructural, la redundancia y la integridad de los datos para cada uno de los datasets trabajados.

#### Dataset 1: Netflix Movies and TV Shows

| Aspecto | Original | 1FN | 2FN | 3FN |
| --- | --- | --- | --- | --- |
| **Número de tablas** | 1 | 5 | 9 | 11 |
| **Total de columnas** | 12 | 16 | 24 | 26 |
| **Redundancia estimada (%)** | > 85% | > 95% | ~ 30% | < 5% |
| **Anomalías identificadas** | Ins. / Act. / Elim. | Ins. / Act. / Elim. | Act. / Elim. | Ninguna |
| **Integridad referencial** | Nula | Nula (solo PKs) | Parcial | Completa (FKs estrictas) |


#### Dataset 2: E-commerce Sales Data

| Aspecto | Original | 1FN | 2FN | 3FN |
| --- | --- | --- | --- | --- |
| **Número de tablas** | 1 | 1 | 3 | 4 |
| **Total de columnas** | 8 | 8 | 10 | 11 |
| **Redundancia estimada (%)** | > 75% | > 75% | ~ 20% | < 5% |
| **Anomalías identificadas** | Ins. / Act. / Elim. | Ins. / Act. / Elim. | Act. / Elim. | Ninguna |
| **Integridad referencial** | Nula | Nula (solo PK) | Parcial | Completa (FKs estrictas) |


#### Dataset 3: Hospital Patient Records

| Aspecto | Original | 1FN | 2FN | 3FN |
| --- | --- | --- | --- | --- |
| **Número de tablas** | 1 | 1 | 2 | 4 |
| **Total de columnas** | 85 | 85 | 86 | 89 |
| **Redundancia estimada (%)** | > 60% | > 60% | ~ 25% | < 5% |
| **Anomalías identificadas** | Ins. / Act. / Elim. | Ins. / Act. / Elim. | Act. / Elim. | Ninguna |
| **Integridad referencial** | Nula | Nula (solo PK) | Parcial | Completa (FKs estrictas) |

