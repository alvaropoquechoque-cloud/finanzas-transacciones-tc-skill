---
name: finanzas-transacciones-tc
description: Registra, importa, normaliza y audita transacciones financieras de Sommos; aplica tipos de cambio a USD y prepara movimientos bancarios para conciliación sin duplicar operaciones.
---

# Finanzas Sommos — Transacciones y TC

## Propósito

Operar la capa transaccional del modelo financiero de Sommos.

Esta skill se encarga principalmente de:

- importar extractos bancarios;
- registrar movimientos en `Transacciones`;
- normalizar ingresos, egresos y transferencias internas;
- prevenir duplicados;
- aplicar tipo de cambio a USD;
- preservar categorizaciones existentes;
- preparar correctamente los movimientos para conciliación bancaria;
- mantener trazabilidad entre extracto, transacción y vistas financieras posteriores.

`Transacciones` es la fuente operativa principal del modelo.

## Archivo principal

Google Sheet:

`Finanzas Sommos — Workflow y Control`

Spreadsheet ID:

`1RXy19WZMPQePflFaFeIIHnh09BpJbwOnk6Wumw8bW4E`

Antes de cualquier escritura, leer en vivo:

- pestañas existentes;
- encabezados actuales;
- fórmulas;
- validaciones;
- filas relacionadas;
- estructura de bancos y cuentas.

Nunca asumir posiciones históricas de columnas.

## Alcance principal

Esta skill opera principalmente:

- `Transacciones`
- `TC BCB`
- `Importación extractos`

Puede consultar:

- `Config`
- `Reglas categorización`
- `Bancos`
- `CxC Mensual`
- `CxP Mensual`
- `Operative incomes`
- `Real S&A`
- `Runway Mensual`
- `Dashboard`

No debe modificar presupuesto, runway o dashboard salvo que sea necesario para reparar una dependencia causada directamente por una modificación transaccional.

---

# Flujo operativo

Usar este flujo:

`Extracto → extracción → normalización → deduplicación → Transacciones → TC → categorización → conciliación bancaria → vistas derivadas`

## 1. Recibir extracto

Preferencia de formatos:

1. CSV
2. XLSX / XLSM
3. PDF con texto estructurado
4. imagen o PDF escaneado como último recurso

Cuando el extracto tenga:

- saldo inicial;
- saldo final;
- totales de créditos;
- totales de débitos;

usar esos datos como controles de conciliación.

No cargar movimientos directamente sin revisar el contenido completo del extracto.

## 2. Extraer movimientos

Para cada movimiento identificar, cuando exista:

- fecha;
- banco o cuenta;
- descripción original;
- contraparte;
- identificador bancario;
- moneda;
- importe;
- signo del movimiento;
- saldo posterior;
- referencia o glosa.

Mantener suficiente información de la descripción bancaria para permitir trazabilidad posterior.

## 3. Normalizar

En `Transacciones`, los importes deben registrarse como valores positivos.

El sentido económico se expresa mediante `Tipo`.

Tipos principales:

- `Ingreso`
- `Egreso`
- `Transferencia interna`

No guardar egresos como números negativos si la estructura vigente usa `Tipo + monto positivo`.

## 4. Deduplicar

Antes de agregar un movimiento, buscar si ya existe.

Prioridad de identificación:

1. identificador bancario nativo;
2. banco + fecha + monto + descripción;
3. banco + fecha + monto + contraparte;
4. revisión manual cuando existan movimientos iguales legítimos.

Nunca duplicar un movimiento solamente porque aparece nuevamente en otro archivo o extracto.

Dos operaciones con mismo monto y misma fecha pueden ser legítimamente distintas si tienen identificadores o contrapartes diferentes.

---

# Transacciones

## Fuente de verdad

`Transacciones` es la fuente operativa central.

Las vistas mensuales y financieras deben derivarse de esta información siempre que corresponda.

No crear una segunda copia manual del mismo movimiento en otra pestaña para hacer cuadrar un reporte.

## Mes y banco

La hoja puede incluir columnas auxiliares como:

- `Mes`
- `Banco / cuenta`

Estas columnas sirven para ordenar y visualizar movimientos por:

`Mes → Banco / cuenta → Fecha`

No deben alterar el significado económico de la transacción.

Si son fórmulas o campos auxiliares, preservar su lógica existente.

## Fecha

Para movimientos bancarios realizados:

`Fecha = fecha efectiva del extracto`

Para obligaciones o cuentas pendientes que todavía no pasaron por banco:

- conservar la fecha documental correspondiente;
- utilizar `Fecha vencimiento` para controlar cuándo debería cobrarse o pagarse.

No cambiar una fecha histórica solo para mover un flujo a otro mes.

## Fecha pago / cobro

Cuando exista una obligación previamente registrada como `Pendiente` y posteriormente se pague o cobre:

- actualizar la transacción original;
- registrar `Fecha pago / cobro`;
- cambiar el estado a `Pagado/Cobrado`;
- no crear una segunda transacción para liquidarla, salvo que la estructura vigente requiera explícitamente movimientos separados.

La fecha original del documento debe conservarse.

---

# Tipos de movimiento

## Ingreso

Usar cuando existe una entrada económica real que constituye ingreso o financiamiento según su categoría.

Un ingreso `Pendiente` representa una cuenta por cobrar.

Un ingreso `Pagado/Cobrado` representa un movimiento realizado.

## Egreso

Usar para pagos, gastos y obligaciones.

Un egreso `Pendiente` representa una cuenta por pagar.

Un egreso `Pagado/Cobrado` representa un gasto o salida realizada.

## Transferencia interna

Usar cuando el dinero se mueve entre cuentas controladas por Sommos.

Ejemplos:

- Brex → Meru
- Meru → BancoSol
- cuenta de grant → cuenta principal
- treasury → checking

Una transferencia interna:

- no es ingreso;
- no es gasto;
- no debe incrementar ingresos operativos;
- no debe incrementar burn.

Debe registrarse con suficiente información para identificar:

- `Cuenta origen`
- `Cuenta destino`
- detalle de la transferencia

Para conciliación bancaria:

- la cuenta origen recibe una salida;
- la cuenta destino recibe una entrada.

Nunca dejar una transferencia sin dirección cuando esa ausencia pueda generar diferencias bancarias.

---

# Categorización

Consultar `Reglas categorización` y `Config`.

Si una transacción ya tiene una categoría válida asignada por el usuario:

**preservarla.**

No reemplazar una categorización manual válida simplemente porque una regla automática devolvería otra cosa.

Para movimientos nuevos:

1. intentar regla activa aplicable;
2. respetar Tipo y condiciones de la regla;
3. validar que la categoría exista en `Config`;
4. si no existe una coincidencia suficientemente clara, usar `Por categorizar`.

Nunca inventar una categoría.

La revisión humana prevalece sobre una inferencia automática.

---

# Tipo de cambio

## USD

`TC a USD = 1`

`Monto USD = Monto original`

## SOL

Mientras la política vigente del Sheet sea:

`TC a USD = 0.28`

entonces:

`Monto USD = Monto SOL × 0.28`

No modificar esta política sin instrucción explícita.

## BOB

Usar el TCO oficial registrado en `TC BCB` correspondiente a la fecha de la transacción.

Si no existe publicación para esa fecha:

usar el último TCO oficial disponible con:

`fecha TC <= fecha transacción`

Nunca usar un TC futuro.

Conversión:

`Monto USD = Monto BOB / TC`

## Otras monedas

Usar `TC manual (otras)` únicamente cuando la hoja lo requiera.

No inventar tipos de cambio.

---

# Estados de pago

Estados principales:

- `Pendiente`
- `Pagado/Cobrado`

Reglas:

`Ingreso + Pendiente → CxC`

`Egreso + Pendiente → CxP`

`Ingreso + Pagado/Cobrado → ingreso realizado`

`Egreso + Pagado/Cobrado → egreso realizado`

Las vistas actuales de cuentas son:

- `CxC Mensual`
- `CxP Mensual`

Las antiguas pestañas `CxC` y `CxP` ya no deben recrearse.

---

# Vistas derivadas

Después de registrar o actualizar transacciones, revisar que las vistas relacionadas respondan correctamente.

Según el movimiento pueden verse afectadas:

- `CxC Mensual`
- `CxP Mensual`
- `Operative incomes`
- `Real S&A`
- `Bancos`
- `Presupuesto`
- `Runway Mensual`
- `Dashboard`

No escribir manualmente en una vista derivada para corregir un problema que pertenece a `Transacciones`.

Corregir siempre la fuente correcta.

---

# Conciliación posterior a un extracto

Después de importar un extracto:

1. confirmar que todos los movimientos necesarios estén en `Transacciones`;
2. confirmar que no existan duplicados;
3. validar Tipo;
4. validar categorización;
5. validar moneda y TC;
6. revisar transferencias internas y su dirección;
7. comparar ingresos y egresos por banco;
8. comparar saldo calculado con saldo real del extracto;
9. investigar cualquier diferencia.

La regla conceptual es:

`Saldo calculado = Saldo inicial + Entradas - Salidas`

Luego:

`Diferencia = Saldo final banco - Saldo calculado`

La diferencia esperada de una cuenta conciliada es:

`0`

o estar dentro de la tolerancia definida por el modelo.

Nunca crear un movimiento ficticio para lograr diferencia cero.

---

# Casos especiales de extractos

## Reversiones

Si un banco muestra:

- cargo;
- posterior reversión;

registrar ambos movimientos cuando sean movimientos bancarios reales distintos y necesarios para reproducir el saldo.

No eliminar ambos simplemente porque el efecto neto sea cero.

## Comisiones

Cuando exista una comisión bancaria real:

- registrarla como movimiento independiente si aparece separada;
- usar la categoría bancaria correspondiente definida en `Config`.

Si una transferencia enviada y una recibida difieren por una comisión, no asumir automáticamente que la diferencia desapareció.

Investigar primero el extracto.

## Extractos parciales

Si el archivo no cubre todo el mes:

- no declarar el banco completamente conciliado;
- identificar el período cubierto;
- señalar que faltan movimientos o cierre completo.

## Pagos agrupados

Una transferencia bancaria puede liquidar varias facturas.

No forzar una relación uno-a-uno cuando el documento demuestra un pago consolidado.

Mantener trazabilidad suficiente para vincular el pago con las obligaciones correspondientes.

---

# Guardrails

- No inventar movimientos.
- No inventar saldos.
- No inventar categorías.
- No inventar bancos.
- No inventar país.
- No inventar responsable.
- No inventar fecha de vencimiento.
- No inventar tipo de cambio.
- No duplicar operaciones.
- No borrar movimientos históricos para cuadrar bancos.
- No convertir transferencias internas en ingresos o gastos.
- No cambiar categorizaciones manuales válidas sin instrucción.
- No contar `Pendiente` como realizado.
- No sobrescribir fórmulas o validaciones innecesariamente.
- No depender de posiciones históricas de columnas.

Si existe incertidumbre material, conservar el dato original y señalar qué requiere confirmación.

---

# QA obligatorio

Después de cualquier modificación relevante:

## Transacciones

- revisar encabezados actuales;
- verificar filas escritas;
- confirmar fechas;
- confirmar banco;
- confirmar moneda;
- confirmar Tipo;
- confirmar categoría;
- confirmar estado;
- confirmar TC;
- confirmar Monto USD;
- buscar duplicados.

## Fórmulas

Buscar:

- `#REF!`
- `#VALUE!`
- `#N/A`
- `#ERROR!`

## Bancos

Cuando se haya importado un extracto:

- verificar saldo inicial;
- verificar entradas;
- verificar salidas;
- verificar saldo final;
- verificar diferencia;
- confirmar conciliación.

## Vistas dependientes

Comprobar las vistas afectadas, especialmente:

- `CxC Mensual`
- `CxP Mensual`
- `Operative incomes`
- `Real S&A`
- `Runway Mensual`
- `Dashboard`

---

# Referencias

Consultar solamente las referencias necesarias para cada tarea:

- `references/tc.md`
- `references/importacion-extractos.md`
- `references/transferencias.md`

Si el contenido de una referencia histórica contradice el Google Sheet actual, prevalece la estructura viva del Sheet.
