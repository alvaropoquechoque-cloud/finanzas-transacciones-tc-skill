# Transferencias internas

## Regla principal

Las transferencias entre cuentas propias de Sommos:

- Tipo = `Transferencia`
- Categoría = `Transferencias internas`
- no son ingreso
- no son egreso económico
- no forman parte del burn

## Campos

Cuando existan los datos, registrar:
- cuenta origen
- cuenta destino
- detalle transferencia

## Conciliación

Una transferencia puede aparecer como salida en una cuenta y entrada en otra, pero económicamente representa un único movimiento interno.

No registrar ambos lados como gasto e ingreso.

## Ejemplos

- Brex → Brex Card
- Treasury → Primary Checking
- movimientos entre cuentas propias de Sommos

Antes de crear una transferencia, revisar si ya existe el movimiento correspondiente para evitar duplicados.
