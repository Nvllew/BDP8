# Documentación Técnica: Normalización de Bases de Datos

## Dataset 1: Netflix Movies and TV Shows

---
#### Estructura Original 
El dataset presenta un modelo completamente desnormalizado diseñado para un ánalisis rápido, pero ineficiente para objetivos mas avanzados.

* 
**Total de registros:** 8,807.

* 
**Total de columnas:** 12.

* 
**Tipos de datos presentes:** 

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


#### Identificación de Problemas de Normalización 

* 
**Violación de 1FN:** Múltiples columnas (`director`, `cast`, `country`, `listed_in`) contienen datos multivaluados separados por comas.


* 
**Redundancia de Datos:** Las cadenas de texto para `type` (ej. "Movie") y `rating` (ej. "TV-MA") se repiten en miles de registros.


* 
**Anomalías Identificadas:** 


* **Actualización:** Modificar el nombre de un actor requiere una búsqueda de subcadenas (`LIKE '%nombre%'`) en toda la tabla, lo cual es ineficiente y propenso a errores.
* **Inserción:** No es posible registrar a un actor o director en el sistema sin que esté asociado a un `show` específico.
* **Eliminación:** Si se elimina el único `show` de un director específico de la plataforma, el sistema pierde el registro de la existencia de dicho director.

---

## Dataset 2: E-commerce Sales Data

---

#### Estructura Original

Este conjunto de datos registra transacciones de ventas minoristas en línea. A diferencia del dataset de Netflix, este es un modelo transaccional clásico donde el problema principal no son las listas separadas por comas, sino la repetición masiva de datos maestros junto con datos de transacciones.

* 
**Total de registros:** ~500,000 registros.


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

#### Identificación de Problemas de Normalización

* **Violación de 2FN (Dependencias Parciales):** Para identificar una fila única de manera precisa, necesitamos una clave primaria compuesta por `(InvoiceNo, StockCode)`. Sin embargo, atributos como `Description` y `UnitPrice` dependen únicamente de `StockCode` (el producto), no de la factura. De igual manera, `InvoiceDate` depende solo de `InvoiceNo`.


* 
**Violación de 3FN (Dependencias Transitivas):** El atributo `Country` (País) no depende directamente de la factura ni del producto, sino que depende de `CustomerID` (el país pertenece al cliente, y el cliente realiza la factura).


* 
**Redundancia de Datos:** Los nombres de los productos (`Description`), los precios unitarios y los datos del cliente se repiten en cada línea de cada factura comprada.


* 
Anomalías Identificadas:


* **Anomalía de Inserción:** No podemos registrar un nuevo producto en la base de datos (con su código y descripción) hasta que un cliente lo compre y se genere un `InvoiceNo` (ya que la PK no puede ser nula).
* **Anomalía de Actualización:** Si el proveedor corrige el nombre de un producto (`Description`), habría que actualizar miles de registros de transacciones históricas para mantener la consistencia.
* **Anomalía de Eliminación:** Si un cliente cancela su única compra y eliminamos el registro de la factura, perderíamos también la información del cliente (`CustomerID`, `Country`).



---
```python
import pandas as pd
df_hospital = pd.read_csv('dataset.csv')
print("--- Hospital Dataset Info ---")
df_hospital.info()
print("\n--- Hospital Dataset Head ---")
print(df_hospital.head(5).to_string())



```

```text
--- Hospital Dataset Info ---
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 91713 entries, 0 to 91712
Data columns (total 85 columns):
 #   Column                         Non-Null Count  Dtype  
---  ------                         --------------  -----  
 0   encounter_id                   91713 non-null  int64  
 1   patient_id                     91713 non-null  int64  
 2   hospital_id                    91713 non-null  int64  
 3   age                            87485 non-null  float64
 4   bmi                            88284 non-null  float64
 5   elective_surgery               91713 non-null  int64  
 6   ethnicity                      90318 non-null  object 
 7   gender                         91688 non-null  object 
 8   height                         90379 non-null  float64
 9   icu_admit_source               91601 non-null  object 
 10  icu_id                         91713 non-null  int64  
 11  icu_stay_type                  91713 non-null  object 
 12  icu_type                       91713 non-null  object 
 13  pre_icu_los_days               91713 non-null  float64
 14  weight                         88993 non-null  float64
 15  apache_2_diagnosis             90051 non-null  float64
 16  apache_3j_diagnosis            90612 non-null  float64
 17  apache_post_operative          91713 non-null  int64  
 18  arf_apache                     90998 non-null  float64
 19  gcs_eyes_apache                89812 non-null  float64
 20  gcs_motor_apache               89812 non-null  float64
 21  gcs_unable_apache              90676 non-null  float64
 22  gcs_verbal_apache              89812 non-null  float64
 23  heart_rate_apache              90835 non-null  float64
 24  intubated_apache               90998 non-null  float64
 25  map_apache                     90719 non-null  float64
 26  resprate_apache                90479 non-null  float64
 27  temp_apache                    87605 non-null  float64
 28  ventilated_apache              90998 non-null  float64
 29  d1_diasbp_max                  91548 non-null  float64
 30  d1_diasbp_min                  91548 non-null  float64
 31  d1_diasbp_noninvasive_max      90673 non-null  float64
 32  d1_diasbp_noninvasive_min      90673 non-null  float64
 33  d1_heartrate_max               91568 non-null  float64
 34  d1_heartrate_min               91568 non-null  float64
 35  d1_mbp_max                     91493 non-null  float64
 36  d1_mbp_min                     91493 non-null  float64
 37  d1_mbp_noninvasive_max         90234 non-null  float64
 38  d1_mbp_noninvasive_min         90234 non-null  float64
 39  d1_resprate_max                91328 non-null  float64
 40  d1_resprate_min                91328 non-null  float64
 41  d1_spo2_max                    91380 non-null  float64
 42  d1_spo2_min                    91380 non-null  float64
 43  d1_sysbp_max                   91554 non-null  float64
 44  d1_sysbp_min                   91554 non-null  float64
 45  d1_sysbp_noninvasive_max       90686 non-null  float64
 46  d1_sysbp_noninvasive_min       90686 non-null  float64
 47  d1_temp_max                    89389 non-null  float64
 48  d1_temp_min                    89389 non-null  float64
 49  h1_diasbp_max                  88094 non-null  float64
 50  h1_diasbp_min                  88094 non-null  float64
 51  h1_diasbp_noninvasive_max      84363 non-null  float64
 52  h1_diasbp_noninvasive_min      84363 non-null  float64
 53  h1_heartrate_max               88923 non-null  float64
 54  h1_heartrate_min               88923 non-null  float64
 55  h1_mbp_max                     87074 non-null  float64
 56  h1_mbp_min                     87074 non-null  float64
 57  h1_mbp_noninvasive_max         82629 non-null  float64
 58  h1_mbp_noninvasive_min         82629 non-null  float64
 59  h1_resprate_max                87356 non-null  float64
 60  h1_resprate_min                87356 non-null  float64
 61  h1_spo2_max                    87528 non-null  float64
 62  h1_spo2_min                    87528 non-null  float64
 63  h1_sysbp_max                   88102 non-null  float64
 64  h1_sysbp_min                   88102 non-null  float64
 65  h1_sysbp_noninvasive_max       84372 non-null  float64
 66  h1_sysbp_noninvasive_min       84372 non-null  float64
 67  d1_glucose_max                 85906 non-null  float64
 68  d1_glucose_min                 85906 non-null  float64
 69  d1_potassium_max               82128 non-null  float64
 70  d1_potassium_min               82128 non-null  float64
 71  apache_4a_hospital_death_prob  83766 non-null  float64
 72  apache_4a_icu_death_prob       83766 non-null  float64
 73  aids                           90998 non-null  float64
 74  cirrhosis                      90998 non-null  float64
 75  diabetes_mellitus              90998 non-null  float64
 76  hepatic_failure                90998 non-null  float64
 77  immunosuppression              90998 non-null  float64
 78  leukemia                       90998 non-null  float64
 79  lymphoma                       90998 non-null  float64
 80  solid_tumor_with_metastasis    90998 non-null  float64
 81  apache_3j_bodysystem           90051 non-null  object 
 82  apache_2_bodysystem            90051 non-null  object 
 83  Unnamed: 83                    0 non-null      float64
 84  hospital_death                 91713 non-null  int64  
dtypes: float64(71), int64(7), object(7)
memory usage: 59.5+ MB

--- Hospital Dataset Head ---
   encounter_id  patient_id  hospital_id   age    bmi  elective_surgery  ethnicity gender  height           icu_admit_source  icu_id icu_stay_type      icu_type  pre_icu_los_days  weight  apache_2_diagnosis  apache_3j_diagnosis  apache_post_operative  arf_apache  gcs_eyes_apache  gcs_motor_apache  gcs_unable_apache  gcs_verbal_apache  heart_rate_apache  intubated_apache  map_apache  resprate_apache  temp_apache  ventilated_apache  d1_diasbp_max  d1_diasbp_min  d1_diasbp_noninvasive_max  d1_diasbp_noninvasive_min  d1_heartrate_max  d1_heartrate_min  d1_mbp_max  d1_mbp_min  d1_mbp_noninvasive_max  d1_mbp_noninvasive_min  d1_resprate_max  d1_resprate_min  d1_spo2_max  d1_spo2_min  d1_sysbp_max  d1_sysbp_min  d1_sysbp_noninvasive_max  d1_sysbp_noninvasive_min  d1_temp_max  d1_temp_min  h1_diasbp_max  h1_diasbp_min  h1_diasbp_noninvasive_max  h1_diasbp_noninvasive_min  h1_heartrate_max  h1_heartrate_min  h1_mbp_max  h1_mbp_min  h1_mbp_noninvasive_max  h1_mbp_noninvasive_min  h1_resprate_max  h1_resprate_min  h1_spo2_max  h1_spo2_min  h1_sysbp_max  h1_sysbp_min  h1_sysbp_noninvasive_max  h1_sysbp_noninvasive_min  d1_glucose_max  d1_glucose_min  d1_potassium_max  d1_potassium_min  apache_4a_hospital_death_prob  apache_4a_icu_death_prob  aids  cirrhosis  diabetes_mellitus  hepatic_failure  immunosuppression  leukemia  lymphoma  solid_tumor_with_metastasis apache_3j_bodysystem apache_2_bodysystem  Unnamed: 83  hospital_death
0         66154       25312          118  68.0  22.73                 0  Caucasian      M   180.3                      Floor      92         admit         CTICU          0.541667    73.9               113.0               502.01                      0         0.0              3.0               6.0                0.0                4.0              118.0               0.0        40.0             36.0         39.3                0.0           68.0           37.0                       68.0                       37.0             119.0              72.0        89.0        46.0                    89.0                    46.0             34.0             10.0        100.0         74.0         131.0          73.0                     131.0                      73.0         39.9         37.2           68.0           63.0                       68.0                       63.0             119.0             108.0        86.0        85.0                    86.0                    85.0             26.0             18.0        100.0         74.0         131.0         115.0                     131.0                     115.0           168.0           109.0               4.0               3.4                           0.10                      0.05   0.0        0.0                1.0              0.0                0.0       0.0       0.0                          0.0               Sepsis      Cardiovascular          NaN               0
1        114252       59342           81  77.0  27.42                 0  Caucasian      F   160.0                      Floor      90         admit  Med-Surg ICU          0.927778    70.2               108.0               203.01                      0         0.0              1.0               3.0                0.0                1.0              120.0               0.0        46.0             33.0         35.1                1.0           95.0           31.0                       95.0                       31.0             118.0              72.0       120.0        38.0                   120.0                    38.0             32.0             12.0        100.0         70.0         159.0          67.0                     159.0                      67.0         36.3         35.1           61.0           48.0                       61.0                       48.0             114.0             100.0        85.0        57.0                    85.0                    57.0             31.0             28.0         95.0         70.0          95.0          71.0                      95.0                      71.0           145.0           128.0               4.2               3.8                           0.47                      0.29   0.0        0.0                1.0              0.0                0.0       0.0       0.0                          0.0          Respiratory         Respiratory          NaN               0
2        119783       50777          118  25.0  31.95                 0  Caucasian      F   172.7       Accident & Emergency      93         admit  Med-Surg ICU          0.000694    95.3               122.0               703.03                      0         0.0              3.0               6.0                0.0                5.0              102.0               0.0        68.0             37.0         36.7                0.0           88.0           48.0                       88.0                       48.0              96.0              68.0       102.0        68.0                   102.0                    68.0             21.0              8.0         98.0         91.0         148.0         105.0                     148.0                     105.0         37.0         36.7           88.0           58.0                       88.0                       58.0              96.0              78.0        91.0        83.0                    91.0                    83.0             20.0             16.0         98.0         91.0         148.0         124.0                     148.0                     124.0             NaN             NaN               NaN               NaN                           0.00                      0.00   0.0        0.0                0.0              0.0                0.0       0.0       0.0                          0.0            Metabolic           Metabolic          NaN               0
3         79267       46918          118  81.0  22.64                 1  Caucasian      F   165.1  Operating Room / Recovery      92         admit         CTICU          0.000694    61.7               203.0              1206.03                      1         0.0              4.0               6.0                0.0                5.0              114.0               1.0        60.0              4.0         34.8                1.0           48.0           42.0                       48.0                       42.0             116.0              92.0        84.0        84.0                    84.0                    84.0             23.0              7.0        100.0         95.0         158.0          84.0                     158.0                      84.0         38.0         34.8           62.0           44.0                        NaN                        NaN             100.0              96.0        92.0        71.0                     NaN                     NaN             12.0             11.0        100.0         99.0         136.0         106.0                       NaN                       NaN           185.0            88.0               5.0               3.5                           0.04                      0.03   0.0        0.0                0.0              0.0                0.0       0.0       0.0                          0.0       Cardiovascular      Cardiovascular          NaN               0
4         92056       34377           33  19.0    NaN                 0  Caucasian      M   188.0       Accident & Emergency      91         admit  Med-Surg ICU          0.073611     NaN               119.0               601.01                      0         0.0              NaN               NaN                NaN                NaN               60.0               0.0       103.0             16.0         36.7                0.0           99.0           57.0                       99.0                       57.0              89.0              60.0       104.0        90.0                   104.0                    90.0             18.0             16.0        100.0         96.0         147.0         120.0                     147.0                     120.0         37.2         36.7           99.0           68.0                       99.0                       68.0              89.0              76.0       104.0        92.0                   104.0                    92.0              NaN              NaN        100.0        100.0         130.0         120.0                     130.0                     120.0             NaN             NaN               NaN               NaN                            NaN                       NaN   0.0        0.0                0.0              0.0                0.0       0.0       0.0                          0.0               Trauma              Trauma          NaN               0


```

¡Por supuesto! Para finalizar la primera fase de tu práctica, aquí tienes el análisis técnico estructurado para el **Dataset 3 (Hospital Patient Records)**.

Tal como lo solicitaste, este documento cubre únicamente hasta el segundo punto de la rúbrica y mantiene el mismo formato profesional que los anteriores para que puedas agregarlo directamente a tu archivo `.md`.

---

# Documentación Técnica: Normalización de Bases de Datos

## Dataset 3: Hospital Patient Records

---


#### Estructura Original

Este dataset es un registro clínico masivo en formato de "tabla plana" (wide format). Contiene información demográfica de pacientes combinada con métricas detalladas de signos vitales, diagnósticos y características de la Unidad de Cuidados Intensivos (UCI) por cada encuentro médico.

* **Total de registros:** 91,713.
* **Total de columnas:** 85.
* **Tipos de datos presentes:**
* `Numérico Flotante (float64)`: age, bmi, height, weight, d1_diasbp_max, etc. (71 columnas)
* `Numérico Entero (int64)`: encounter_id, patient_id, hospital_id, icu_id, elective_surgery, etc. (7 columnas)
* `Texto (object)`: ethnicity, gender, icu_admit_source, icu_stay_type, icu_type, etc. (7 columnas)



**Ejemplo de 5 registros representativos (columnas seleccionadas por legibilidad):**

| encounter_id | patient_id | hospital_id | age | gender | bmi | icu_id | icu_type | apache_2_diagnosis | hospital_death |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 66154 | 25312 | 118 | 68.0 | M | 22.73 | 92 | CTICU | 113.0 | 0 |
| 114252 | 59342 | 81 | 77.0 | F | 27.42 | 90 | Med-Surg ICU | 108.0 | 0 |
| 119783 | 50777 | 118 | 25.0 | F | 31.95 | 93 | Med-Surg ICU | 122.0 | 0 |
| 79267 | 46918 | 118 | 81.0 | F | 22.64 | 92 | CTICU | 203.0 | 0 |
| 92056 | 34377 | 33 | 19.0 | M | *NaN* | 91 | Med-Surg ICU | 119.0 | 0 |

#### Identificación de Problemas de Normalización

A diferencia de Netflix, este dataset no tiene múltiples valores separados por comas, pero sufre de una severa redundancia estructural debido a su diseño de tabla ancha ("wide table") y mezcla de entidades.

* **Violación de 2FN (Dependencias Parciales):** Si consideramos que la tabla registra "Encuentros Médicos" (`encounter_id`), observamos que los datos del paciente (`age`, `gender`, `ethnicity`, `height`) dependen de `patient_id` y no del encuentro en sí. De igual forma, el tipo de unidad (`icu_type`) depende de `icu_id`, no del paciente ni del encuentro.
* **Violación de 3FN (Dependencias Transitivas):** Existe información del hospital acoplada con el encuentro. Por ejemplo, `hospital_id` determina lógicamente la ubicación o características del hospital, información que es independiente del `encounter_id`.
* **Redundancia de Datos Masiva:**
* **Datos demográficos:** Si un paciente (`patient_id`) tiene 5 encuentros (5 `encounter_id` distintos), su edad, género y origen étnico se repetirán 5 veces en la base de datos.
* **Datos de infraestructura:** Los nombres y tipos de las UCIs (`icu_type`, `icu_stay_type`) se repiten en texto plano por cada paciente ingresado.


* **Anomalías Identificadas:**
* **Anomalía de Actualización:** Si se corrige la estatura (`height`) o fecha de nacimiento/edad de un paciente, el sistema obliga a buscar y actualizar todos los encuentros históricos de ese `patient_id` para evitar discrepancias.
* **Anomalía de Inserción:** No se puede dar de alta a un nuevo paciente en el sistema hasta que tenga un encuentro en la UCI (`encounter_id`), ya que este último actúa como identificador de la fila transaccional.



---