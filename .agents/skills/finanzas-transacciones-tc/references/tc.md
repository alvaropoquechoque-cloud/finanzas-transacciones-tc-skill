# Tipo de cambio a USD

## Propósito

Definir cómo convertir cada transacción a USD dentro del modelo financiero de Sommos.

La fuente viva es el Google Sheet:

- Spreadsheet: `Finanzas Sommos — Workflow y Control`
- Spreadsheet ID: `1RXy19WZMPQePflFaFeIIHnh09BpJbwOnk6Wumw8bW4E`

Pestañas relevantes:

- `Transacciones`
- `TC BCB`

## Columnas relevantes en Transacciones

Trabajar por nombre de encabezado, no por posición histórica:

- Fecha
- Moneda
- Monto original
- TC a USD
- Monto USD
- TC manual (otras)

Antes de escribir fórmulas o datos, leer los encabezados actuales.

## Reglas de conversión

### USD

`TC a USD = 1`

`Monto USD = Monto original`

---

### SOL

Se utiliza el TC fijo definido actualmente por el modelo:

`TC a USD = 0.28`

`Monto USD = Monto original × 0.28`

No cambiar esta regla sin instrucción explícita del usuario.

---

### BOB

Para bolivianos se utiliza el `TCO vigente` de la pestaña `TC BCB` correspondiente a la fecha de la transacción.

Conversión:

`Monto USD = Monto BOB / TCO vigente`

La búsqueda debe utilizar la fecha de la transacción.

Si el día no tiene publicación oficial por fin de semana, feriado u otra ausencia de cotización, usar el último TCO oficial disponible anterior o igual a esa fecha.

No utilizar automáticamente el TC del día actual para una transacción histórica.

## TC BCB

La pestaña `TC BCB` mantiene:

- Fecha
- TCO publicado BCB
- TCO vigente

`TCO vigente` arrastra el último valor oficial disponible para cubrir días sin publicación.

La tabla existente debe preservarse.

Antes de modificarla:

1. leer fórmulas actuales;
2. verificar el rango de fechas;
3. confirmar que no existan errores;
4. evitar sobrescribir datos oficiales o excepciones cargadas manualmente.

## Otras monedas

Para monedas distintas de:

- USD
- SOL
- BOB

utilizar el campo:

`TC manual (otras)`

No inventar un tipo de cambio.

Si falta el TC manual:

- mantener `TC a USD` sin valor;
- no estimar `Monto USD`;
- informar que falta el dato necesario.

## Fórmula conceptual de TC

```text
Si moneda = USD:
    TC = 1

Si moneda = SOL:
    TC = 0.28

Si moneda = BOB:
    TC = último TCO vigente <= fecha de transacción

En cualquier otra moneda:
    TC = TC manual (otras)
