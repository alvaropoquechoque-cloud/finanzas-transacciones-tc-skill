# Finanzas Sommos — Tipo de cambio a USD

## Propósito

Definir cómo convertir movimientos financieros a USD dentro del workflow financiero de Sommos.

Esta referencia pertenece a:

`finanzas-transacciones-tc`

La fuente viva es el Google Sheet:

- Spreadsheet: `Finanzas Sommos — Workflow y Control`
- Spreadsheet ID: `1RXy19WZMPQePflFaFeIIHnh09BpJbwOnk6Wumw8bW4E`

Pestañas relevantes:

- `Transacciones`
- `TC BCB`
- `Config`

La estructura viva del Google Sheet prevalece sobre este documento.

---

# Principio general

El tipo de cambio debe permitir mantener simultáneamente:

- moneda original;
- monto original;
- TC aplicado;
- monto equivalente en USD;
- trazabilidad de la fuente del TC.

Nunca reemplazar el monto original del extracto por su equivalente USD.

---

# Campos relevantes

En `Transacciones` revisar conceptualmente:

- Fecha
- Moneda
- Monto original
- TC a USD
- Monto USD
- campos manuales de TC si existen

Trabajar siempre por nombre de encabezado.

No asumir posiciones históricas de columnas.

---

# USD

Para movimientos originalmente en USD:

`TC a USD = 1`

`Monto USD = Monto original`

No aplicar conversiones adicionales.

---

# BOB

Para bolivianos utilizar el tipo de cambio oficial almacenado en:

`TC BCB`

La regla es:

usar el último TC oficial disponible tal que:

`Fecha TC <= fecha aplicable`

Nunca utilizar un tipo de cambio posterior a la fecha del movimiento.

Si la fecha corresponde a:

- fin de semana;
- feriado;
- día sin publicación;

utilizar el último TC oficial anterior disponible.

Conversión conceptual:

`Monto USD = Monto BOB / TC`

---

# TC BCB

`TC BCB` contiene el histórico utilizado por el modelo.

Puede incluir conceptualmente:

- Fecha
- TCO publicado
- TCO vigente

`TCO vigente` puede arrastrar el último valor publicado para cubrir días sin cotización.

Antes de modificar esta pestaña:

1. leer la estructura actual;
2. comprobar fórmulas;
3. revisar el rango de fechas;
4. identificar datos oficiales vs excepciones manuales;
5. evitar sobrescribir históricos correctos.

No inventar un TC para llenar un vacío.

---

# PEN / SOL

Mientras la política viva del modelo mantenga:

`TC a USD = 0.28`

usar:

`Monto USD = Monto original × 0.28`

No cambiar esta convención automáticamente por consultar una cotización externa más reciente.

Si Sommos cambia oficialmente esta política:

actualizar primero la configuración correspondiente y luego el modelo.

---

# Otras monedas

Para monedas diferentes de las expresamente soportadas:

utilizar el mecanismo manual definido en el Sheet.

No inventar TC.

Si falta el tipo de cambio necesario:

- mantener el dato pendiente;
- no estimar el Monto USD;
- informar qué valor falta.

---

# Fecha aplicable

Como regla general, para movimientos bancarios:

usar la fecha efectiva del movimiento.

No utilizar automáticamente:

- fecha actual;
- fecha de carga del archivo;
- fecha de generación del extracto.

Para obligaciones documentales con una metodología específica, respetar la lógica aprobada del modelo.

---

# Precisión

Mantener suficiente precisión en el TC y en Monto USD.

No redondear cálculos internos prematuramente.

La presentación visual puede mostrar:

- 0 decimales;
- 2 decimales;

según la pestaña.

Pero las fórmulas fuente deben preservar la precisión necesaria para:

- CxC;
- CxP;
- Bancos;
- Cash Flow;
- Balance Sheet.

---

# Pagos agrupados y facturas

Si un pago consolidado cubre varias facturas:

el TC documental de cada componente puede no ser idéntico al TC implícito del movimiento bancario total.

No forzar un único TC cuando el Control/documentación demuestra una lógica por factura.

Caso conocido:

PPO tuvo un pago bancario consolidado correspondiente a varias facturas mensuales.

El split debe preservar:

`SUMA partes = total bancario`

y mantener la trazabilidad documental.

---

# Diferencias cambiarias

No utilizar `Exchange rate differences` simplemente porque una transacción esté en moneda extranjera.

Una diferencia de cambio existe cuando hay un efecto económico entre:

- valor contable;
- valor de liquidación;
- tipos de cambio aplicables.

La mera conversión de BOB o PEN a USD no constituye automáticamente una diferencia cambiaria.

---

# QA del TC

Después de cargar o modificar movimientos:

comprobar:

- moneda original;
- monto original;
- TC;
- Monto USD;
- fecha usada para TC;
- ausencia de TC futuros;
- precisión;
- consistencia con `TC BCB`.

Buscar además:

- `#REF!`
- `#VALUE!`
- `#N/A`
- `#DIV/0!`
- `#ERROR!`

---

# Regla de seguridad

Nunca modificar un TC únicamente para:

- hacer cuadrar Bancos;
- hacer cuadrar Cash Flow;
- eliminar una diferencia del Balance;
- reproducir manualmente un valor esperado.

Primero determinar cuál es el TC correcto según la fuente y metodología del modelo.

---

# Principio final

El objetivo del TC no es conseguir un número conveniente.

El objetivo es mantener una conversión:

- documentada;
- consistente;
- reproducible;
- auditable.

Si moneda = BOB:
    TC = último TCO vigente <= fecha de transacción

En cualquier otra moneda:
    TC = TC manual (otras)
