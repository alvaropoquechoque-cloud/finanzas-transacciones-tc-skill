---
name: finanzas-transacciones-tc
description: Importa, normaliza, registra y audita movimientos financieros de Sommos; aplica tipos de cambio a USD, evita duplicados, gestiona transferencias y prepara las transacciones para conciliación bancaria sin confundir cash con devengo contable.
---

# Finanzas Sommos — Transacciones y TC

## Propósito

Operar la capa transaccional y bancaria del workflow financiero de Sommos.

Esta skill se encarga principalmente de:

- importar extractos bancarios;
- registrar y actualizar movimientos en `Transacciones`;
- normalizar ingresos, egresos y transferencias internas;
- prevenir duplicados;
- preservar trazabilidad con el extracto original;
- aplicar tipo de cambio a USD;
- respetar la categorización definida por el sistema;
- registrar fechas reales de pago/cobro;
- preparar los movimientos para conciliación bancaria;
- mantener consistencia entre cash real y las vistas financieras dependientes.

## Principio fundamental

`Transacciones` es la fuente de verdad para:

- movimientos bancarios;
- cash realizado;
- pagos;
- cobros;
- transferencias;
- banco/cuenta utilizada;
- fecha efectiva del movimiento;
- moneda;
- monto;
- conciliación.

`Transacciones` **no es la fuente única del devengo contable**.

El modelo distingue:

`Operative incomes`
→ devengo de ingresos / monto a facturar

`Real S&A`
→ devengo de gastos / monto a pagar

`Sueldos 2026`
→ devengo y planificación de nómina

`Transacciones`
→ cobros y pagos reales

Por lo tanto:

**cash ≠ devengo**

y una transacción bancaria no debe crear o mover automáticamente un ingreso/gasto de periodo solamente porque el dinero se haya recibido o pagado.

---

# Archivo principal

Google Sheet:

`Finanzas Sommos — Workflow y Control`

Spreadsheet ID:

`1RXy19WZMPQePflFaFeIIHnh09BpJbwOnk6Wumw8bW4E`

URL:

`https://docs.google.com/spreadsheets/d/1RXy19WZMPQePflFaFeIIHnh09BpJbwOnk6Wumw8bW4E/edit`

Antes de escribir:

1. leer la estructura viva;
2. leer encabezados actuales;
3. revisar filas relacionadas;
4. revisar fórmulas y validaciones;
5. buscar duplicados;
6. identificar el banco/cuenta correspondiente.

Nunca asumir posiciones históricas de columnas.

---

# Pestañas principales

Esta skill opera principalmente sobre:

- `Transacciones`
- `TC BCB`

Puede utilizar como staging/auditoría:

- `Importación extractos`

`Importación extractos` es una pestaña técnica y actualmente puede permanecer oculta.

No es una fuente contable independiente.

También puede consultar:

- `Config`
- `Reglas categorización`
- `Bancos`
- `CxC Mensual`
- `CxP Mensual`
- `CxP Sueldos`
- `Operative incomes`
- `Real S&A`
- `Sueldos 2026`
- `Real P&L`
- `Cash Flow`
- `Balance Sheet`
- `Runway Mensual`
- `Dashboard`

No escribir manualmente en estados financieros desde esta skill salvo que el usuario solicite expresamente reparar una dependencia causada por una modificación transaccional.

---

# Estructura actual de Transacciones

La estructura conocida actualmente contiene campos conceptuales como:

- Mes
- Banco / cuenta
- Fecha
- Tipo
- País
- Categoría
- Descripción
- Moneda
- Monto original
- TC a USD
- Monto USD
- campos de referencia / conciliación
- Estado pago
- Fecha vencimiento
- Responsable / Proyecto
- identificadores
- información de transferencias
- Detalle / soporte
- Fecha pago / cobro

La hoja actualmente se extiende aproximadamente de A:V.

Esta información es referencial.

**Leer siempre los encabezados vivos antes de escribir.**

---

# Flujo de importación

Usar conceptualmente:

`Extracto`
→ `extracción`
→ `normalización`
→ `deduplicación`
→ `Transacciones`
→ `TC`
→ `categorización`
→ `conciliación`
→ `Bancos`
→ vistas dependientes

---

# 1. Recibir el extracto

Formatos preferidos:

1. XLSX / XLSM / CSV estructurado
2. PDF con texto seleccionable
3. imagen o PDF escaneado como último recurso

Cuando el extracto contenga:

- saldo inicial;
- saldo final;
- total créditos;
- total débitos;
- moneda;
- periodo;

usar esos valores como controles de conciliación.

No importar parcialmente un archivo sin identificar primero:

- cuenta;
- periodo;
- moneda;
- cobertura completa o parcial.

---

# 2. Extraer movimientos

Para cada movimiento identificar, cuando exista:

- fecha;
- banco/cuenta;
- descripción original;
- contraparte;
- referencia bancaria;
- identificador;
- moneda;
- monto;
- naturaleza entrada/salida;
- saldo posterior;
- soporte adicional.

Preservar suficiente texto original para permitir auditoría posterior.

No reemplazar una descripción bancaria útil por una descripción demasiado resumida.

---

# 3. Normalizar

## Importes

La convención actual de `Transacciones` utiliza normalmente:

`Monto original = positivo`

y la dirección económica se expresa mediante:

`Tipo`

Tipos conocidos:

- `Ingreso`
- `Egreso`
- `Transferencia interna`

No convertir egresos en números negativos si la estructura viva utiliza monto positivo + Tipo.

---

# 4. Deduplicación

Nunca cargar un movimiento sin buscar primero si ya existe.

Prioridad de identificación:

1. identificador bancario único;
2. banco/cuenta + fecha + monto + referencia;
3. banco/cuenta + fecha + monto + descripción;
4. contraparte + monto + fecha;
5. revisión manual.

No considerar automáticamente duplicado solamente porque:

- tenga el mismo importe;
- tenga la misma fecha;
- tenga descripción similar.

Pueden existir transacciones legítimas repetidas.

---

# Preexistencia de obligaciones

Antes de crear una nueva fila por un movimiento bancario:

1. buscar si existe una fila pendiente que represente la obligación/cobro;
2. revisar monto;
3. revisar moneda;
4. revisar descripción;
5. revisar vencimiento;
6. revisar contraparte;
7. revisar soporte.

Cuando un movimiento bancario liquide claramente una obligación ya registrada:

preferir actualizar la obligación existente con:

- Banco/cuenta;
- Estado pago;
- Fecha pago/cobro;
- información de conciliación;

en lugar de crear un duplicado.

No unir filas cuando no exista evidencia suficiente de que representan el mismo hecho económico.

---

# Fecha

## Movimiento bancario nuevo

Para un movimiento directamente importado de un extracto:

`Fecha = fecha efectiva del banco`

## Obligación previamente registrada

Cuando ya existe una obligación documental:

- conservar su fecha original;
- conservar `Fecha vencimiento`;
- registrar el movimiento efectivo mediante `Fecha pago / cobro`.

Nunca cambiar la fecha documental para hacer que un gasto o ingreso aparezca en otro mes.

---

# Fecha pago / cobro

`Fecha pago / cobro` representa cuándo ocurrió efectivamente el cash.

Se utiliza para:

- asignar cobros/pagos al mes correcto;
- conciliación;
- CxC/CxP;
- Cash Flow;
- análisis de caja.

No reemplaza:

- fecha de factura;
- fecha de devengo;
- fecha de vencimiento.

---

# Estado de pago

Estados conocidos incluyen:

- `Pendiente`
- `Pagado/Cobrado`

La estructura viva de `Config` prevalece.

## Regla importante

`Pendiente` no implica automáticamente que el monto deba convertirse en devengo contable.

Una fila pendiente puede utilizarse para seguimiento operacional.

El devengo oficial se determina principalmente desde:

### CxC

`Operative incomes`
→ `Monto a facturar`

`Transacciones`
→ `Cobro`

### CxP

`Real S&A`
→ `Monto a pagar`

`Transacciones`
→ `Pago`

### Sueldos

`Sueldos 2026`
→ gasto/devengo

`CxP Sueldos`
→ obligación/pago/saldo

---

# Ingreso

Usar `Ingreso` cuando exista una entrada económica real o prevista que corresponda a la naturaleza configurada.

Un ingreso realizado puede representar:

- cobro de cliente;
- grant;
- interés;
- financiamiento;
- otra entrada.

El Tipo `Ingreso` por sí solo no significa ingreso operativo de P&L.

La categoría determina la naturaleza del cash y el devengo puede vivir en otra pestaña.

---

# Egreso

Usar `Egreso` para:

- pagos;
- gastos bancarios;
- obligaciones;
- impuestos;
- servicios;
- otros desembolsos.

Un egreso bancario pagado no determina automáticamente el mes del gasto en P&L.

Ejemplo:

una factura de julio puede pagarse en agosto.

El gasto pertenece al devengo correspondiente y el cash pertenece a agosto.

---

# Transferencias internas

Utilizar cuando el dinero se mueve entre cuentas controladas por Sommos.

Una transferencia interna:

- no es ingreso;
- no es gasto;
- no es CxC;
- no es CxP;
- no afecta P&L;
- sí afecta los saldos bancarios.

Categoría conocida:

`Transferencias internas`

Registrar cuando sea posible:

- cuenta origen;
- cuenta destino;
- referencia;
- detalle.

No convertir una transferencia entre cuentas propias en ingreso o egreso operativo solamente porque un extracto muestre una entrada o salida.

---

# Pagos agrupados

Una transferencia bancaria puede liquidar múltiples facturas.

No asumir relación uno-a-uno.

Cuando exista soporte documental suficiente, puede ser necesario dividir conceptualmente el movimiento por factura para preservar:

- fecha de factura;
- mes de devengo;
- monto de obligación;
- trazabilidad.

## Caso PPO

Existe un caso histórico validado donde un pago bancario consolidado correspondía a varias facturas mensuales.

La solución utilizada fue separar las facturas manteniendo:

- el monto total bancario;
- la misma fecha de pago;
- el detalle individual de cada factura;
- el mes correcto de devengo.

Nunca realizar este split sin respaldo documental y sin comprobar que:

`SUMA partes = movimiento bancario total`

---

# Categorización

La lógica de categorización pertenece principalmente a:

`finanzas-config-categorizacion`

Esta skill debe:

1. consultar `Reglas categorización`;
2. respetar categorías válidas existentes;
3. aplicar reglas activas a movimientos nuevos cuando sea seguro;
4. usar solamente categorías existentes en `Config`;
5. mantener `Por categorizar` cuando exista ambigüedad.

No inventar categorías.

No reemplazar una categoría manual validada simplemente porque una regla automática encuentre otra coincidencia.

---

# Tipo de cambio

## USD

`TC a USD = 1`

`Monto USD = Monto original`

---

# BOB

Usar el tipo de cambio oficial vigente registrado en:

`TC BCB`

Regla:

usar el último TC oficial disponible tal que:

`Fecha TC <= Fecha aplicable`

Nunca utilizar un TC futuro.

Para fines de semana o feriados:

usar el último valor oficial anterior disponible.

Conversión conceptual:

`Monto USD = Monto BOB / TC`

salvo que la estructura viva indique una metodología específica para un movimiento documentado.

---

# PEN / SOL

Mientras la política viva del modelo utilice:

`TC a USD = 0.28`

mantener esa convención.

No modificarla silenciosamente.

Si `Config` o el modelo vivo cambia la política:

seguir la configuración vigente.

---

# Otras monedas

Utilizar TC manual solamente cuando exista:

- fuente;
- respaldo;
- instrucción;
- convención explícita.

No inventar tipos de cambio.

---

# Precisión

No redondear prematuramente valores fuente.

Preservar suficiente precisión en:

- TC;
- Monto USD;
- conciliaciones;
- splits de facturas.

La presentación visual puede redondearse.

Los cálculos internos deben mantener la precisión necesaria para reproducir los estados financieros.

Diferencias de motor Excel vs Google Sheets de fracciones de centavo pueden existir.

No crear ajustes ficticios para eliminarlas.

---

# Importación extractos

`Importación extractos` puede utilizarse como staging/auditoría.

Actualmente puede mantenerse oculta.

Su función puede incluir:

- identificar origen del movimiento;
- estado de importación;
- deduplicación;
- trazabilidad.

No utilizarla como segunda fuente de verdad.

Una vez validado el movimiento:

`Transacciones`

es la capa operativa oficial.

---

# Conciliación bancaria

Después de importar un extracto:

1. confirmar que todos los movimientos estén representados;
2. confirmar que no existan duplicados;
3. revisar Tipo;
4. revisar moneda;
5. revisar TC;
6. revisar Monto USD;
7. revisar categorías;
8. identificar transferencias internas;
9. revisar filas previamente pendientes liquidadas;
10. comparar contra `Bancos`.

La ecuación conceptual:

`Saldo calculado = Saldo inicial + entradas - salidas`

`Diferencia = Saldo real - saldo calculado`

Una cuenta cerrada debe quedar:

`Diferencia ≈ 0`

dentro de la tolerancia definida por el modelo.

Nunca crear un movimiento ficticio para cuadrar el saldo.

---

# Bancos

`Bancos` es la vista de conciliación y saldo por cuenta.

Esta skill puede validar:

- saldo inicial;
- movimientos del mes;
- saldo calculado;
- saldo final del extracto;
- diferencia.

No sobrescribir manualmente una diferencia para mostrar cero.

Investigar la causa.

---

# Mes cerrado

El cierre mensual se gestiona principalmente desde:

`finanzas-cierre-mensual`

Pero esta skill debe respetar los meses ya cerrados.

Actualmente agosto de 2026 es un periodo validado/cerrado dentro del modelo.

No modificar movimientos históricos de un mes cerrado sin:

1. identificar el impacto;
2. explicar la causa;
3. releer Bancos;
4. revisar estados financieros relacionados.

---

# Reversiones

Si el banco muestra:

- cargo real;
- reversión posterior real;

registrar ambos cuando ambos sean necesarios para reproducir el extracto.

No eliminarlos porque el efecto neto sea cero.

---

# Comisiones bancarias

Si aparece una comisión separada:

registrarla como movimiento independiente.

Categoría esperada cuando corresponda:

`Bank fees`

No confundir con:

- interés;
- diferencia de cambio;
- impuesto;
- transferencia.

---

# Intereses

Un interés acreditado por el banco puede corresponder a:

`Bank interest earned`

Un interés financiero pagado/devengado puede corresponder a:

`Financial expense`

No mezclar con Bank fees.

---

# Grants y financiamiento

Entradas como:

- INNOVATECH;
- Startup Perú;
- INCOFIN;
- FIID Guatemala;

pueden corresponder a:

`Other financing cash flow`

No convertir automáticamente estos movimientos en ingreso operativo.

## Startup Perú

Existe un caso validado de aproximadamente:

`USD 934`

correspondiente a septiembre de 2026.

Debe mantenerse como financiamiento/grant según el modelo y no como ingreso operativo ordinario.

---

# Guardrails

- No inventar movimientos.
- No inventar saldos.
- No inventar categorías.
- No inventar banco/cuenta.
- No inventar país.
- No inventar responsable.
- No inventar vencimiento.
- No inventar TC.
- No duplicar operaciones.
- No borrar movimientos históricos para cuadrar Bancos.
- No modificar devengo para hacer coincidir cash.
- No convertir transferencias internas en ingresos/gastos.
- No sobrescribir categorizaciones manuales válidas sin razón.
- No redondear prematuramente.
- No modificar un mes cerrado silenciosamente.
- No depender de posiciones históricas de columnas.

---

# QA obligatorio antes de escribir

Antes de cualquier modificación relevante:

- [ ] Leer encabezados vivos.
- [ ] Identificar banco/cuenta.
- [ ] Revisar periodo del extracto.
- [ ] Buscar duplicados.
- [ ] Buscar obligaciones pendientes relacionadas.
- [ ] Confirmar fecha.
- [ ] Confirmar moneda.
- [ ] Confirmar monto.
- [ ] Confirmar Tipo.
- [ ] Revisar categoría.
- [ ] Revisar TC aplicable.
- [ ] Confirmar si es transferencia interna.

---

# QA obligatorio después de escribir

## Transacciones

Releer las filas modificadas y comprobar:

- Fecha
- Banco/cuenta
- Tipo
- País
- Categoría
- Descripción
- Moneda
- Monto original
- TC
- Monto USD
- Estado pago
- Fecha vencimiento
- Fecha pago/cobro
- Conciliación

## Duplicados

Volver a buscar:

- mismo ID;
- mismo banco;
- misma fecha;
- mismo monto;
- misma referencia.

## Bancos

Si el cambio es bancario:

- revisar saldo calculado;
- saldo del extracto;
- diferencia;
- estado de conciliación.

## Dependencias

Según el movimiento, revisar:

- `CxC Mensual`
- `CxP Mensual`
- `CxP Sueldos`
- `Cash Flow`
- `Balance Sheet`
- `Runway Mensual`
- `Dashboard`

`Operative incomes`, `Real S&A` y `Sueldos 2026` no deben cambiar simplemente por haber registrado cash.

---

# Checks del modelo

Cuando una modificación afecte movimientos relevantes de cash, comprobar que no rompa:

- Cash Flow vs Balance Sheet;
- saldo bancario;
- cierre mensual;
- Dashboard.

Buscar además:

- `#REF!`
- `#VALUE!`
- `#N/A`
- `#DIV/0!`
- `#ERROR!`

---

# Regla de finalización

Nunca reportar una importación, modificación o conciliación como terminada únicamente porque se ejecutó una escritura.

Antes de decir que está lista:

1. releer las filas modificadas;
2. comprobar que las fórmulas calcularon;
3. verificar Bancos;
4. revisar las dependencias relevantes;
5. confirmar que no se introdujeron errores.

---

# Coordinación con otras skills

## `finanzas-config-categorizacion`

Usar para:

- catálogos;
- categorías;
- reglas automáticas;
- prioridades;
- taxonomía.

## `finanzas-cxc-cxp`

Usar para:

- CxC;
- CxP;
- grants pendientes;
- vencimientos;
- saldos;
- CxP Sueldos.

## `finanzas-presupuesto-bancos`

Usar para:

- conciliación consolidada de Bancos;
- presupuesto;
- Budget vs P&L.

## `finanzas-runway-dashboard`

Usar para:

- runway;
- escenarios;
- KPIs;
- dashboard.

## `finanzas-estados-financieros`

Cuando exista, usar para:

- Real P&L;
- Cash Flow;
- Balance Sheet;
- checks de tres estados.

## `finanzas-cierre-mensual`

Cuando exista, usar para:

- orquestar el cierre completo;
- comprobar bancos;
- categorización;
- conciliación;
- estados financieros;
- estado final de cierre.

---

# Referencias

Consultar solamente cuando la tarea lo requiera:

- `references/tc.md`
- `references/importacion-extractos.md`
- `references/transferencias.md`

Si una referencia contradice el Sheet vivo:

prevalece el Google Sheet.

---

# Alcance final

Esta skill debe concentrarse en:

**extracto → movimiento → TC → Transacciones → conciliación**

No debe convertirse en una skill de devengo o estados financieros.

Su función es garantizar que la capa de cash sea:

- completa;
- precisa;
- deduplicada;
- trazable;
- correctamente convertida;
- conciliable.
- `references/tc.md`
- `references/importacion-extractos.md`
- `references/transferencias.md`

Si el contenido de una referencia histórica contradice el Google Sheet actual, prevalece la estructura viva del Sheet.
