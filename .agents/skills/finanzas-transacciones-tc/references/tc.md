# Tipo de cambio

## Reglas vigentes

- USD → 1
- SOL → 0.28 mientras esa sea la política vigente en el Sheet
- BOB → tipo de cambio oficial BCB según la fecha de la transacción
- Otras monedas → TC manual

## Conversión a USD

Para BOB:

`Monto USD = Monto BOB / TC`

Para monedas distintas de USD y BOB:

`Monto USD = Monto original × TC`

## Fechas sin cotización

Cuando la fecha cae en fin de semana o feriado, usar el último tipo de cambio oficial disponible igual o anterior a la fecha de la transacción.

## Reglas de seguridad

- No reemplazar silenciosamente un TC histórico ya conciliado.
- Antes de modificar TC, verificar `TC BCB`, `Transacciones` y el efecto en `Monto USD`.
- Si falta TC para una moneda no soportada automáticamente, usar el campo de TC manual.
