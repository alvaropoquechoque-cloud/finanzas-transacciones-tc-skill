---
name: finanzas-transacciones-tc
description: Registra e importa transacciones y aplica la lógica de tipo de cambio a USD del modelo financiero de Sommos.
---

# Finanzas Sommos — Transacciones y TC

## Propósito

Operar el libro de movimientos en `Transacciones` y la lógica de tipo de cambio asociada.

Archivo principal:
- Spreadsheet ID: `1RXy19WZMPQePflFaFeIIHnh09BpJbwOnk6Wumw8bW4E`
- URL: `https://docs.google.com/spreadsheets/d/1RXy19WZMPQePflFaFeIIHnh09BpJbwOnk6Wumw8bW4E/edit`

## Alcance principal

Esta skill opera principalmente:
- `Transacciones`
- `TC BCB`
- `Importación extractos`

También puede consultar:
- `Reglas categorización`
- `Bancos`

## Estructura funcional conocida de Transacciones

Campos principales:
- Fecha
- Tipo
- País
- Categoría
- Descripción
- Moneda
- Monto original
- TC a USD
- Monto USD
- Cuenta / medio
- Conciliación
- Presupuestado
- Estado pago
- Fecha vencimiento
- Responsable
- TC manual
- Cuenta origen
- Cuenta destino
- Detalle transferencia

Antes de cualquier modificación, leer los encabezados actuales del Sheet y no asumir posiciones históricas.

## Tipos de movimiento

Valores principales:
- `Ingreso`
- `Egreso`
- `Transferencia`

Las transferencias internas:
- no son ingresos
- no son egresos económicos
- deben categorizarse como `Transferencias internas`
- no deben formar parte del burn

## Reglas de tipo de cambio

- USD → 1
- SOL → 0.28 mientras esa sea la política vigente
- BOB → tipo de cambio oficial BCB según fecha
- otras monedas → TC manual

Para BOB:
`Monto USD = Monto BOB / TC`

Para otras monedas no USD:
`Monto USD = Monto original × TC`

Cuando la fecha cae en fin de semana o feriado, usar el último TC oficial disponible igual o anterior a la fecha.

## Importación de extractos

Antes de importar:
1. Leer las filas existentes en `Transacciones`.
2. Revisar el staging en `Importación extractos`.
3. Buscar duplicados.
4. Identificar cuenta, moneda, fecha y descripción.
5. Aplicar la categorización disponible.
6. Verificar TC.
7. Registrar solo movimientos reales.
8. Validar conciliación después de la carga.

No inventar movimientos para cuadrar un banco.

## Reglas transversales obligatorias

- El Google Sheet `Finanzas Sommos — Workflow y Control` es la fuente viva.
- `Transacciones` es la fuente de verdad operativa.
- Antes de escribir, leer en vivo encabezados, fórmulas, validaciones y filas relacionadas.
- Nunca asumir posiciones históricas de columnas.
- Nunca duplicar una transacción existente.
- Nunca inventar país, categoría, responsable, fecha de vencimiento, cuenta o medio.
- Si no existe una categoría válida, usar `Por categorizar`.
- `Pendiente` no equivale a dinero realizado ni debe afectar bancos/caja.
- Después de cualquier modificación, verificar las vistas dependientes y buscar errores de fórmula.
- Los snapshots en GitHub documentan contexto; si contradicen el Sheet, prevalece el Sheet.
