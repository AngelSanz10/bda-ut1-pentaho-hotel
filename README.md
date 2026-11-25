# Práctica integradora: el informe del hotel con Pentaho (`bda-ut1-pentaho-hotel`)

* **Autor:** Ángel Sanz
* **Módulo / Unidad:** Big Data Aplicado — UT1 (Cierre de unidad con Pentaho Data Integration)
* **Repositorio:** `https://github.com/AngelSanz10/bda-ut1-pentaho-hotel`
* **Commit de referencia:** `a42b8b6969e8b44801fcfdfb91f2b810c7e4ec60`
* **Código de paquete personal:** `A002`
* **Identificador del cierre:** `cierre_aula_01`
* **Fecha y hora de generación de los resultados:** `2026/11/25 23:15:47`

## 1. Objetivo del encargo y explicación para gerencia

Este proyecto resuelve el cierre del conjunto `cierre_aula_01` para un grupo hotelero de Cantabria mediante **Pentaho Data Integration (PDI)**, generando dos ficheros CSV diferenciados y validados automáticamente antes de publicarse:

1. **Informe principal (`salida/informe_hotel_canal.csv`):** muestra el número de reservas cobradas (`n_reservas`) y el importe total cobrado (`cobrado_total`, en **euros**) por combinación de **hotel** y **canal**, incluyendo únicamente reservas con cobro asociado y sumando el importe cobrado (no el precio de reserva).
2. **Control interno de reservas sin cobro (`salida/reservas_sin_cobro.csv`):** conserva las reservas sin cobro asociado en los archivos recibidos (`motivo`: `Sin cobro asociado en el archivo recibido`), indicando su `importe` (precio de la reserva) sin inventar importes ni mezclarlas con el informe de cobros.

### Explicación para gerencia

El conjunto `cierre_aula_01` corresponde a un cierre de aula ya acotado sobre los ficheros suministrados, sin fechas para filtrar por día concreto ni datos de inventario de habitaciones. En este cierre se contabilizan **15 reservas con cobro asociado** que suman un total de **3965.50 €** distribuidos en **13 combinaciones de hotel y canal**. Quedan fuera del informe principal **5 reservas sin cobro asociado** (identificadores `4`, `9`, `11`, `15` y `18`), que se entregan en un listado separado para revisión interna: que una reserva no tenga cobro registrado en estos archivos no demuestra por sí solo un impago, ya que puede deberse a pagos pendientes, plazos de cobro o registros aún no conciliados. Con estos datos **no se calcula ocupación hotelera**; para responder a esa pregunta habría que solicitar información adicional con el número de habitaciones disponibles y ocupadas por hotel y periodo.

## 2. Recorrido de los datos y diseño previo

```text
entrada/ (reservas_pdi.csv, cobros_pdi.csv, hoteles_pdi.csv)
          │
          ▼
Comprobación de existencia de entradas + lectura con separadores y tipos definidos
          │
          ├──► informe_cobros.ktr ──────► temporal/informe_hotel_canal.csv
          └──► reservas_sin_cobro.ktr ─► temporal/reservas_sin_cobro.csv
          │
          ▼
Comprobación de existencia de provisionales + validación automática
(validar_informe.ktr y validar_sin_cobro.ktr)
          │
          ├──► Si todo es válido ──► Copias a salida/ + registro de éxito
          └──► Si hay fallo ───────► Rama de error / aborto (sin publicar o avisando de posible estado parcial)
```

* **Qué necesita dirección y qué representa cada fila:** Dirección necesita conocer cuánto se ha cobrado (`cobrado_total` en euros) y cuántas reservas cobradas hay (`n_reservas`) por `hotel` y `canal` en `cierre_aula_01` (una fila por combinación de hotel y canal), junto con un control separado de las reservas sin cobro (una fila por reserva sin cobro asociado). En las entradas, `reservas_pdi.csv` representa una reserva (`id_reserva` único), `cobros_pdi.csv` un cobro asociado a una reserva (`id_reserva`, como máximo uno por reserva en este conjunto) y `hoteles_pdi.csv` un hotel (`id_hotel` único).
* **Separación de carpetas:** Los ficheros originales se conservan sin modificar en `entrada/` (junto con `asignacion.json` en la raíz); los resultados provisionales se escriben en `temporal/` (ignorado en Git mediante `/temporal/`); y los informes validados y publicados se entregan en `salida/`.
* **Inicio y proceso por lotes:** La lectura se inicia mediante el job `cierre_hotel.kjb` (por ejemplo, ejecutado con `Kitchen.bat` o `Pan.bat`). Un proceso por lotes encaja con un cierre porque procesa un conjunto cerrado de ficheros de forma repetible, ordenada y desatendida. En un entorno real podría lanzarse de madrugada mediante el programador de tareas del sistema operativo (ejecutar `Kitchen.bat` manualmente no programa una hora automática por sí solo).
* **Comprobaciones, repetición y falta de archivos:** Antes de entregar se comprueban los conteos, claves, totales, validaciones automáticas y una ejecución limpia desde otra carpeta. Repetir la ejecución sobre las mismas entradas sobrescribe las salidas sin acumular duplicados. Si falta una entrada requerida o falla la preparación/validación, el job aborta antes de publicar; si falla una copia durante la publicación, el job termina con error y advierte de un posible estado parcial en `salida/`, sin anunciar éxito.
* **Formato CSV y alternativa analítica:** CSV es adecuado para esta entrega por su sencillez, legibilidad, bajo peso y compatibilidad directa con herramientas de hoja de cálculo y procesos ETL sobre un volumen pequeño. Para un histórico grande consultado por columnas, se estudiaría un formato columnar como **Parquet** (o **ORC**), que permite lectura selectiva de columnas y mejor compresión.

## 3. Entorno técnico y estructura del repositorio

* **Pentaho Data Integration:** Community Edition **9.4.0.0-343** (`Spoon.bat`, `Pan.bat` y `Kitchen.bat`).
* **Entorno Java:** Java 8 configurado para el entorno de aula en Windows.
* **Requisitos adicionales:** No requiere base de datos, controladores JDBC, servicios de pago ni plugins extra.

```text
bda-ut1-pentaho-hotel/
├── README.md
├── .gitignore
├── asignacion.json
├── entrada/
│   ├── reservas_pdi.csv
│   ├── cobros_pdi.csv
│   └── hoteles_pdi.csv
├── procesos/
│   ├── informe_cobros.ktr
│   ├── reservas_sin_cobro.ktr
│   ├── validar_informe.ktr
│   ├── validar_sin_cobro.ktr
│   └── cierre_hotel.kjb
├── temporal/          # Generada en local; ignorada con /temporal/ en .gitignore
├── salida/            # Resultados finales publicados (crear antes de ejecutar si no existe)
└── evidencias/        # Logs, tabla esperado/obtenido/conclusión y registros de sesión
```

*(Cada transformación y job incluye una nota en el lienzo con el nombre del autor y la pregunta o función que resuelve).*

## 4. Procedencia, separadores y tipos de los datos de entrada

Los ficheros de entrada proceden del paquete personal asignado (`asignacion.json`, código `A002`) y deben leerse en **UTF-8**, con **cabecera activada**, **punto (`.`) como símbolo decimal** en importes, **sin separador de miles** y con *Lazy conversion* desactivada al comprobar la lectura:

| Fichero | Separador | Filas esperadas (sin cabecera) | Campos `Integer` | Campos `String` | Campos `BigNumber` |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `entrada/reservas_pdi.csv` | `,` (coma) | 20 | `id_reserva`, `id_hotel`, `noches` | `canal` | `importe` |
| `entrada/cobros_pdi.csv` | `;` (punto y coma) | 15 | `id_reserva` | `medio` | `cobrado` |
| `entrada/hoteles_pdi.csv` | `;` (punto y coma) | 5 | `id_hotel`, `id_cadena` | `hotel`, `comarca` | *(ninguno)* |

## 5. Preparación y ejecución desde una copia recién descargada

Git no conserva carpetas vacías. Para ejecutar el proyecto desde una copia recién clonada o descargada en otra carpeta sin modificar rutas internas:

1. Clona o descarga el repositorio en la carpeta elegida y sitúate en su raíz.
2. Crea las carpetas `temporal` y `salida` si no existen:

   ```powershell
   New-Item -ItemType Directory -Path ".\temporal", ".\salida" -Force
   ```

3. Ejecuta el job `cierre_hotel.kjb` con `Kitchen.bat` (ajustando la ruta de instalación de PDI a tu equipo):

   ```powershell
   & "C:\ruta\a\Pentaho\data-integration\Kitchen.bat" /file:".\procesos\cierre_hotel.kjb" /level:Basic
   ```

4. Los resultados finales aparecerán en:
   * `salida/informe_hotel_canal.csv`
   * `salida/reservas_sin_cobro.csv`

## 6. Decisiones de diseño

* **Uniones y agrupación en `informe_cobros.ktr`:** Se ordenan `reservas_pdi.csv` y `cobros_pdi.csv` por `id_reserva` (mismo tipo `Integer`) y se aplica un cruce `INNER` para conservar únicamente las 15 reservas con cobro asociado. Después se ordena por `id_hotel` y se une mediante cruce `INNER` con `hoteles_pdi.csv` para añadir `hotel` (manteniendo las 15 filas). Finalmente se ordena por `hotel` y `canal` y se agrupa sumando `cobrado` en `cobrado_total` (euros, punto decimal y dos decimales, sin separador de miles) y contando `id_reserva` en `n_reservas`, escribiendo `temporal/informe_hotel_canal.csv` en UTF-8 con separador `;` y sin añadir a ejecuciones anteriores. En este conjunto hay como máximo un cobro por reserva; si una reserva tuviera varios cobros parciales, habría que distinguir entre sumar importes cobrados, contar cobros y contar reservas únicas (por ejemplo, deduplicando `id_reserva` para contar reservas).
* **Control de reservas sin cobro en `reservas_sin_cobro.ktr`:** Se renombra `id_reserva` como `id_reserva_cobro` en la rama de cobros para distinguir ambas claves, se ordena cada rama por su clave y se realiza un cruce `LEFT OUTER` con reservas a la izquierda y cobros a la derecha (conservando las 20 reservas). Se filtra donde `id_reserva_cobro` es nulo (sin usar `cobrado = 0`, ya que un cobro de cero y la ausencia de cobro son situaciones distintas), se incorpora `hotel` mediante cruce por `id_hotel`, se seleccionan `id_reserva`, `hotel`, `canal` e `importe` (precio de la reserva, no dinero cobrado), se añade `motivo` con el valor `Sin cobro asociado en el archivo recibido`, se ordena por `id_reserva` y se sobrescribe `temporal/reservas_sin_cobro.csv`.
* **Validaciones y coordinación en `cierre_hotel.kjb`:** El job comprueba la existencia de los tres ficheros de entrada, ejecuta `informe_cobros.ktr` y `reservas_sin_cobro.ktr`, comprueba que existen ambos CSV provisionales y ejecuta `validar_informe.ktr` y `validar_sin_cobro.ktr` (con agregación sin claves, *Always give back a result row*, recuento con *Number of rows* sin campo y *Abort* con umbral `0` ante cabecera vacía, duplicados o valores fuera de control). Solo si ambas validaciones pasan se copian ambos CSV a `salida/` sobrescribiendo los anteriores y se registra el éxito al final; si falta una entrada o falla preparación/validación se detiene antes de publicar, y si falla una copia se sigue una rama de error que marca el fallo y advierte de un posible estado parcial sin anunciar éxito.

## 7. Repetición, comprobaciones y pruebas

* **Repetición:** Ejecutar `cierre_hotel.kjb` varias veces sobre las mismas entradas regenera y sobrescribe los ficheros manteniendo los mismos resultados (**13 grupos**, **15 reservas**, **3965.50 €**, y **5 reservas sin cobro**: `4`, `9`, `11`, `15` y `18`), sin acumular filas duplicadas.
* **Comprobaciones y evidencias en `evidencias/`:**
  * Tabla de validación manual (`esperado`, `obtenido`, `conclusión`) y revisión externa del CSV.
  * Pruebas documentadas del apartado 8 y evidencias adicionales:
    1. **Primera ejecución** (éxito con salidas generadas).
    2. **Repetición** (mismos grupos, filas y totales sin duplicados).
    3. **Falta una entrada** (ruta de cobros inexistente; aborto sin publicar).
    4. **Salida incompleta** (solo cabecera en provisional y ejecución directa de `validar_informe.ktr` con error).
    5. **Total incorrecto** (importe alterado en provisional y ejecución directa de `validar_informe.ktr` con error).
    6. **Recuperación** (restauración de entradas y ejecución completa del job con éxito).
  * Evidencias de bloqueo de publicación por fallo de preparación/validación y de error durante las copias con advertencia de posible estado parcial, además de los registros de sesión (`sesion-01.md`, etc.).

## 8. Fuentes y ayudas utilizadas

* Material de clase, talleres de PDI (UT1, incluyendo talleres 0, 1.6, 1.7 y 1.8) y enunciado de la práctica integradora.
* Apoyo de asistente de IA para revisar configuraciones, redactar documentación y organizar comprobaciones; todo el diseño, ejecución, validación y verificación en PDI y consola ha sido comprobado de forma personal.
