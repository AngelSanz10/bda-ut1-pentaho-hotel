# Práctica integradora: informe del hotel con Pentaho (`bda-ut1-pentaho-hotel`)

* **Autor:** Ángel
* **Módulo / Unidad:** Big Data Aplicado — UT1 (Pentaho Data Integration)
* **Código de paquete personal:** `[A001–A015 — revisar en asignacion.json]`
* **Identificador del cierre:** `cierre_aula_01`

## 1. Objetivo del encargo y explicación para gerencia

Este proyecto procesa los ficheros de reservas, cobros y hoteles del conjunto `cierre_aula_01` mediante **Pentaho Data Integration (PDI)** para generar dos salidas diferenciadas:

1. **`salida/informe_hotel_canal.csv`**: informe de cierre con el número de reservas cobradas (`n_reservas`) y el importe cobrado (`cobrado_total`, en **euros**) por combinación de **hotel** y **canal**.
2. **`salida/reservas_sin_cobro.csv`**: listado de control interno con las reservas que no tienen un cobro asociado en los archivos recibidos (`motivo`: *Sin cobro asociado en el archivo recibido*).

### Nota para gerencia

El conjunto `cierre_aula_01` corresponde a un cierre de aula ya acotado a partir de los archivos suministrados (no incluye marcas temporales para filtrar por día ni datos de habitaciones disponibles u ocupadas por periodo). En este cierre se contabilizan **15 reservas con cobro asociado** que suman un total de **3965.50 €** repartidos en **13 combinaciones de hotel y canal**. Quedan fuera del informe principal **5 reservas sin cobro asociado** (identificadores `4`, `9`, `11`, `15` y `18`), que se entregan en un listado aparte para revisión interna: la ausencia de cobro en estos archivos no demuestra por sí sola un impago, ya que puede deberse a pagos pendientes, plazos de cobro o registros no conciliados aún. Con los datos disponibles **no se calcula ocupación hotelera**; para responder a esa pregunta habría que solicitar el inventario de habitaciones disponibles y ocupadas por hotel y periodo.

* **Fecha y hora de generación de los resultados:** `[AAAA-MM-DD HH:MM — indicar hora de generación final, sin presentarla como fecha de cobro]`

## 2. Recorrido de los datos

```text
entrada/ (reservas_pdi.csv, cobros_pdi.csv, hoteles_pdi.csv)
          │
          ▼
Lectura con tipos y separadores definidos + comprobación de existencia en el job
          │
          ├──► informe_cobros.ktr ──────► temporal/informe_hotel_canal.csv
          └──► reservas_sin_cobro.ktr ─► temporal/reservas_sin_cobro.csv
          │
          ▼
Validación automática (validar_informe.ktr y validar_sin_cobro.ktr)
          │
          ├──► Si todo es válido ──► Copias a salida/ + registro de éxito
          └──► Si hay fallo ───────► Rama de error / aborto (sin publicar o avisando de estado parcial)
