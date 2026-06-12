# Documentación Técnica: Normalización de Bases de Datos

## Dataset 1: Netflix Movies and TV Shows

---
<img width="1022" height="768" alt="Captura de pantalla 2026-06-11 a la(s) 5 36 47 p m" src="https://github.com/user-attachments/assets/979709f4-90e9-4f71-bb40-6493e9f383c5" />
**Figura 1.1.** CSV dataset 1.

#### Estructura Original 
El dataset presenta un modelo completamente desnormalizado diseñado para un ánalisis rápido, pero ineficiente para objetivos mas avanzados.

* **Total de registros:** 8,807.
* **Total de columnas:** 12.
* **Tipos de datos presentes:** 
* `Numérico (int64)`: release_year (1)
* `Texto (object/string)`: show_id, type, title, director, cast, country, date_added, rating, duration, listed_in, description (11)

**Ejemplo de 5 registros representativos:** 
| show_id | type | title | director | cast | country | release_year | rating | listed_in |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| s1 | Movie | Dick Johnson Is Dead | Kirsten Johnson | *NaN* | United States | 2020 | PG-13 | Documentaries |
| s2 | TV Show | Blood & Water | *NaN* | Ama Qamata, Khosi Ngema... | South Africa | 2021 | TV-MA | International TV Shows... |
| s3 | TV Show | Ganglands | Julien Leclercq | Sami Bouajila, Tracy Gotoas... | *NaN* | 2021 | TV-MA | Crime TV Shows... |
| s4 | TV Show | Jailbirds New Orleans | *NaN* | *NaN* | *NaN* | 2021 | TV-MA | Docuseries, Reality TV |
| s5 | TV Show | Kota Factory | *NaN* | Mayur More, Jitendra Kumar... | India | 2021 | TV-MA | International TV Shows... |

<img width="997" height="128" alt="Captura de pantalla 2026-06-11 a la(s) 5 35 55 p m" src="https://github.com/user-attachments/assets/65635e5f-ce32-46e9-acd1-510359f86e52" />
**Figura 1.2.** Datos representativos del dataset 1.

#### Identificación de Problemas de Normalización 

* **Violación de 1FN:** Múltiples columnas (`director`, `cast`, `country`, `listed_in`) contienen datos multivaluados separados por comas.
* **Redundancia de Datos:** Las cadenas de texto para `type` (ej. "Movie") y `rating` (ej. "TV-MA") se repiten en miles de registros.
* **Anomalías Identificadas:** 
* **Actualización:** Modificar el nombre de un actor requiere una búsqueda de subcadenas (`LIKE '%nombre%'`) en toda la tabla, lo cual es ineficiente y propenso a errores.
* **Inserción:** No es posible registrar a un actor o director en el sistema sin que esté asociado a un `show` específico.
* **Eliminación:** Si se elimina el único `show` de un director específico de la plataforma, el sistema pierde el registro de la existencia de dicho director.

### Diagrama de dependencias funcionales
<img width="1434" height="2592" alt="Netflix_DF_Academico" src="https://github.com/user-attachments/assets/7e5f16ef-bf83-43b4-a1de-6b158daad957" />
**Figura 1.3.** Diagrama de dependencias funcionales del dataset 1.

---

 
## Dataset 2: E-commerce Sales Data

---
<img width="1034" height="726" alt="Captura de pantalla 2026-06-11 a la(s) 5 40 16 p m" src="https://github.com/user-attachments/assets/5f0a2f1a-5d71-481d-b47e-9292dd620caa" />
**Figura 2.1.** CSV dataset 2.


#### Estructura Original
Este conjunto de datos registra transacciones de ventas minoristas en línea. A diferencia del dataset de Netflix, este es un modelo transaccional donde el problema principal no son las listas separadas por comas, sino la repetición masiva de datos  junto con datos de transacciones.

* **Total de registros:** ~500,000 registros.
* **Total de columnas:** 8.
* **Tipos de datos presentes:**
* `Numérico`: Quantity (int), UnitPrice (float), CustomerID (float/int)
* `Fecha/Hora`: InvoiceDate (datetime)
* `Texto (string)`: InvoiceNo, StockCode, Description, Country

**Ejemplo de 5 registros representativos:**
| InvoiceNo | StockCode | Description | Quantity | InvoiceDate | UnitPrice | CustomerID | Country |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 536365 | 85123A | WHITE HANGING HEART T-LIGHT HOLDER | 6 | 12/01/2010 08:26 | 2.55 | 17850 | United Kingdom |
| 536365 | 71053 | WHITE METAL LANTERN | 6 | 12/01/2010 08:26 | 3.39 | 17850 | United Kingdom |
| 536365 | 84406B | CREAM CUPID HEARTS COAT HANGER | 8 | 12/01/2010 08:26 | 2.75 | 17850 | United Kingdom |
| 536366 | 22633 | HAND WARMER UNION JACK | 6 | 12/01/2010 08:28 | 1.85 | 17850 | United Kingdom |
| 536367 | 84879 | ASSORTED COLOUR BIRD ORNAMENT | 32 | 12/01/2010 08:34 | 1.69 | 13047 | United Kingdom |

<img width="991" height="129" alt="Captura de pantalla 2026-06-11 a la(s) 5 40 31 p m" src="https://github.com/user-attachments/assets/88952b13-1b12-429e-935e-a7233ff75a8d" />
**Figura 2.2.** Datos representativos del dataset 2.

#### Identificación de Problemas de Normalización
* **Violación de 2FN (Dependencias Parciales):** Para identificar una fila única de manera precisa, necesitamos una clave primaria compuesta por `(InvoiceNo, StockCode)`. Sin embargo, atributos como `Description` y `UnitPrice` dependen únicamente de `StockCode` (el producto), no de la factura. De igual manera, `InvoiceDate` depende solo de `InvoiceNo`.
* **Violación de 3FN (Dependencias Transitivas):** El atributo `Country` (País) no depende directamente de la factura ni del producto, sino que depende de `CustomerID` (el país pertenece al cliente, y el cliente realiza la factura).
* **Redundancia de Datos:** Los nombres de los productos (`Description`), los precios unitarios y los datos del cliente se repiten en cada línea de cada factura comprada.

* **Anomalías Identificadas:**

* **Anomalía de Inserción:** No podemos registrar un nuevo producto en la base de datos (con su código y descripción) hasta que un cliente lo compre y se genere un `InvoiceNo` (ya que la PK no puede ser nula).
* **Anomalía de Actualización:** Si el proveedor corrige el nombre de un producto (`Description`), habría que actualizar miles de registros de transacciones históricas para mantener la consistencia.
* **Anomalía de Eliminación:** Si un cliente cancela su única compra y eliminamos el registro de la factura, perderíamos también la información del cliente (`CustomerID`, `Country`).

### Diagrama de dependencias funcionales
<img width="2942" height="1271" alt="Ecommerce_DF_Academico" src="https://github.com/user-attachments/assets/b98ab602-df07-49b2-a507-e668c7d59c20" />
**Figura 2.3.** Diagrama de dependencias funcionales del dataset 2.


## Dataset 3: Hospital Patient Records

---
<img width="1022" height="768" alt="Captura de pantalla 2026-06-11 a la(s) 5 40 51 p m" src="https://github.com/user-attachments/assets/e41d18bb-e103-44df-a0cc-98c24ac845f3" />
**Figura 3.1.** CSV dataset 3.

#### Estructura Original
Este dataset es un registro clínico masivo en formato de "tabla plana". Contiene información demográfica de pacientes combinada con métricas detalladas de signos vitales, diagnósticos y características de la Unidad de Cuidados Intensivos (UCI) por cada encuentro médico.

* **Total de registros:** 91,713.
* **Total de columnas:** 85.
* **Tipos de datos presentes:**
* `Numérico Flotante (float64)`: age, bmi, height, weight, d1_diasbp_max, etc. (71 columnas)
* `Numérico Entero (int64)`: encounter_id, patient_id, hospital_id, icu_id, elective_surgery, etc. (7 columnas)
* `Texto (object)`: ethnicity, gender, icu_admit_source, icu_stay_type, icu_type, etc. (7 columnas)

**Ejemplo de 5 registros representativos:**

| encounter_id | patient_id | hospital_id | age | gender | bmi | icu_id | icu_type | apache_2_diagnosis | hospital_death |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 66154 | 25312 | 118 | 68.0 | M | 22.73 | 92 | CTICU | 113.0 | 0 |
| 114252 | 59342 | 81 | 77.0 | F | 27.42 | 90 | Med-Surg ICU | 108.0 | 0 |
| 119783 | 50777 | 118 | 25.0 | F | 31.95 | 93 | Med-Surg ICU | 122.0 | 0 |
| 79267 | 46918 | 118 | 81.0 | F | 22.64 | 92 | CTICU | 203.0 | 0 |
| 92056 | 34377 | 33 | 19.0 | M | *NaN* | 91 | Med-Surg ICU | 119.0 | 0 |

<img width="969" height="134" alt="Captura de pantalla 2026-06-11 a la(s) 5 40 59 p m" src="https://github.com/user-attachments/assets/0cbd75ff-873e-44eb-b250-96d34afe1602" />
**Figura 3.2.** Datos representativos del dataset 3.

#### Identificación de Problemas de Normalización
A diferencia de Netflix, este dataset no tiene múltiples valores separados por comas, pero sufre de una severa redundancia estructural debido a su diseño de tabla ancha y mezcla de entidades.

* **Violación de 2FN (Dependencias Parciales):** Si consideramos que la tabla registra "Encuentros Médicos" (`encounter_id`), observamos que los datos del paciente (`age`, `gender`, `ethnicity`, `height`) dependen de `patient_id` y no del encuentro en sí. De igual forma, el tipo de unidad (`icu_type`) depende de `icu_id`, no del paciente ni del encuentro.
* **Violación de 3FN (Dependencias Transitivas):** Existe información del hospital acoplada con el encuentro. Por ejemplo, `hospital_id` determina lógicamente la ubicación o características del hospital, información que es independiente del `encounter_id`.
* **Redundancia de Datos Masiva:**
* **Datos demográficos:** Si un paciente (`patient_id`) tiene 5 encuentros (5 `encounter_id` distintos), su edad, género y origen étnico se repetirán 5 veces en la base de datos.
* **Datos de infraestructura:** Los nombres y tipos de las UCIs (`icu_type`, `icu_stay_type`) se repiten en texto plano por cada paciente ingresado.

* **Anomalías Identificadas:**
* **Anomalía de Actualización:** Si se corrige la estatura (`height`) o fecha de nacimiento/edad de un paciente, el sistema obliga a buscar y actualizar todos los encuentros históricos de ese `patient_id` para evitar discrepancias.
* **Anomalía de Inserción:** No se puede dar de alta a un nuevo paciente en el sistema hasta que tenga un encuentro en la UCI (`encounter_id`), ya que este último actúa como identificador de la fila transaccional.

### Diagrama de dependencias funcionales
<img width="2883" height="2142" alt="Hospital_DF_Academico" src="https://github.com/user-attachments/assets/ef8619d8-ad53-4b92-b889-7f38da154537" />
**Figura 3.3.** Diagrama de dependencias funcionales del dataset 3.

---
