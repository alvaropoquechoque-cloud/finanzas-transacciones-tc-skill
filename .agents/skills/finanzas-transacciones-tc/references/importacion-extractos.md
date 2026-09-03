# Importación de extractos

## Flujo

1. Leer el extracto o staging en `Importación extractos`.
2. Leer las filas actuales de `Transacciones`.
3. Buscar duplicados antes de registrar movimientos.
4. Identificar:
   - fecha
   - descripción
   - monto
   - moneda
   - cuenta / medio
5. Aplicar categorización disponible.
6. Aplicar o validar TC.
7. Registrar únicamente movimientos reales.
8. Verificar conciliación después de la carga.

## Duplicados

No duplicar:
- movimientos ya existentes
- cargos ya conciliados
- transferencias internas
- pagos de tarjeta registrados por ambos lados como gasto económico

## Bancos

Nunca inventar un movimiento para hacer cuadrar un banco.

Si existe diferencia de conciliación:
- investigar el extracto
- revisar fechas
- revisar TC
- revisar transferencias
- revisar movimientos faltantes o duplicados

No corregir artificialmente el saldo.

## Referencia BancoSol julio 2026

Snapshot histórico:
- 36 movimientos
- saldo inicial Bs 3.202,06
- ingresos Bs 43.617,92
- egresos Bs 11.888,82
- saldo final Bs 34.931,16
- cierre conciliado

Meru y Scotiabank no tuvieron movimientos registrados en julio.

Este snapshot es solo referencia histórica; siempre prevalece el Sheet en vivo.
