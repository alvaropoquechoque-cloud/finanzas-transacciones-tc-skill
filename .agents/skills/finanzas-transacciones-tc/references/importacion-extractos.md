# Finanzas Sommos — Importación de extractos bancarios

## Propósito

Definir cómo convertir extractos bancarios y estados de cuenta en movimientos confiables dentro de:

`Transacciones`

La importación debe garantizar:

- completitud;
- trazabilidad;
- deduplicación;
- TC correcto;
- correcta identificación de pagos/cobros;
- transferencias internas correctas;
- conciliación bancaria.

---

# Fuente de verdad

`Transacciones` es la fuente de verdad para:

- movimientos bancarios;
- cash realizado;
- pagos;
- cobros;
- banco/cuenta;
- fecha efectiva;
- moneda;
- monto;
- conciliación.

Pero no es la fuente única del devengo.

El modelo separa:

`Operative incomes`
→ devengo de ingresos

`Real S&A`
→ devengo de gastos

`Sueldos 2026`
→ devengo de nómina

`Transacciones`
→ cash

Por lo tanto:

un extracto confirma que hubo movimiento de dinero.

No determina automáticamente cuándo debe reconocerse el ingreso o gasto en P&L.

---

# Flujo estándar

`Extracto`
→ identificación de cuenta y periodo
→ extracción
→ normalización
→ búsqueda de obligaciones existentes
→ deduplicación
→ registro/actualización en Transacciones
→ TC
→ categorización
→ conciliación
→ Bancos
→ validación de dependencias

---

# Antes de importar

Identificar:

- banco;
- cuenta;
- moneda;
- periodo;
- fecha inicial;
- fecha final;
- si el extracto es completo o parcial;
- saldo inicial;
- saldo final;
- créditos;
- débitos.

Si falta información material:

no declarar el periodo como conciliado.

---

# Extractos conocidos

El workflow puede recibir extractos de cuentas como:

- Banco Sol
- Brex
- Brex Card
- Brex Checking
- Brex Treasury
- Meru
- otras cuentas activas definidas en `Bancos`

La lista viva en el modelo prevalece.

No crear un nuevo nombre de cuenta antes de comprobar si ya existe.

---

# Una fila por movimiento bancario

Como regla general:

cada movimiento independiente del extracto debe poder rastrearse individualmente.

No agrupar movimientos porque:

- sean del mismo proveedor;
- tengan misma fecha;
- compartan categoría;
- parezcan parte del mismo concepto.

La trazabilidad bancaria tiene prioridad.

---

# Duplicados

Antes de insertar:

buscar coincidencias por:

- ID bancario;
- banco/cuenta;
- fecha;
- moneda;
- monto;
- descripción;
- referencia;
- contraparte.

Dos movimientos con mismo monto y fecha pueden ser legítimamente distintos.

No eliminar uno sin evidencia.

---

# Obligaciones existentes

Antes de crear una fila nueva por un pago/cobro:

buscar si ya existe una obligación pendiente relacionada.

Comparar:

- descripción;
- proveedor/cliente;
- moneda;
- monto;
- fecha documental;
- vencimiento;
- proyecto;
- soporte.

Cuando el extracto confirme claramente el pago/cobro de una fila existente:

actualizar esa fila con:

- Banco/cuenta;
- Estado pago;
- Fecha pago/cobro;
- Conciliación;
- referencia bancaria.

No crear automáticamente una segunda fila.

---

# Devengo vs cash

## Cobro de cliente

El devengo puede venir de:

`Operative incomes`

El extracto confirma:

`Cobro`

No mover el ingreso del P&L al mes del cobro.

---

# Pago a proveedor

El devengo puede venir de:

`Real S&A`

El extracto confirma:

`Pago`

No mover el gasto al mes bancario.

---

# Pago de sueldo

El devengo viene de:

`Sueldos 2026`

El pago se refleja en:

`CxP Sueldos`

y/o `Transacciones` según la arquitectura vigente.

No reconocer nuevamente el gasto al pagar.

---

# Fecha

Para un movimiento bancario nuevo:

usar la fecha efectiva del extracto.

Cuando liquida una obligación existente:

- conservar la fecha documental;
- conservar vencimiento;
- registrar `Fecha pago / cobro` con la fecha del banco.

No reemplazar la fecha de factura por la fecha bancaria.

---

# Moneda

Preservar:

- moneda original;
- monto original.

Después aplicar:

- USD → 1
- BOB → TC BCB aplicable
- PEN/SOL → política vigente
- otras → TC manual documentado

No reemplazar el monto original por USD.

---

# Categorización

La importación debe consultar:

- `Config`
- `Reglas categorización`

Pero puede finalizar con:

`Por categorizar`

si no existe suficiente evidencia.

Es mejor una transacción pendiente de clasificación que una categoría incorrecta.

No utilizar categorización para cuadrar bancos.

---

# Transferencias internas

Si el dinero se mueve entre cuentas propias:

Tipo:

`Transferencia interna`

Categoría:

`Transferencias internas`

Completar cuando sea posible:

- Cuenta origen
- Cuenta destino
- detalle

No reconocer como ingreso/gasto.

---

# Intermediarios

Nombres como procesadores, billeteras o intermediarios no identifican necesariamente a la contraparte económica.

Separar:

- contraparte bancaria;
- cliente/proveedor económico;
- cuenta origen;
- cuenta destino;
- concepto real.

Ejemplo conocido:

un movimiento mostrado por un intermediario puede ser en realidad una transferencia entre cuentas propias.

---

# Comisiones

Si el extracto muestra una comisión separada:

registrarla separadamente.

No reducir artificialmente el importe principal.

Categoría habitual:

`Bank fees`

cuando corresponda.

No inventar comisiones para explicar diferencias.

---

# Reversiones

Si existen:

- cargo;
- reversión;

y ambos son movimientos bancarios reales:

registrar ambos si son necesarios para reproducir el saldo.

No netearlos simplemente porque el resultado consolidado sea cero.

---

# Pagos agrupados

Un pago bancario puede cubrir varias facturas.

Cuando exista soporte documental:

puede ser necesario dividir la obligación por factura para preservar el devengo correcto.

Caso conocido:

PPO.

Regla crítica:

`SUMA de componentes = total del movimiento bancario`

La fecha de pago puede ser común.

Las fechas de factura/devengo pueden ser distintas.

---

# Pagos parciales

No marcar toda una obligación como pagada cuando solo hubo un pago parcial.

Determinar:

- obligación inicial;
- cash recibido/pagado;
- saldo restante.

No inventar splits sin respaldo.

---

# Importación extractos

La pestaña:

`Importación extractos`

puede utilizarse como staging/auditoría técnica.

Puede permanecer oculta.

No es una segunda fuente contable.

El movimiento validado debe terminar reflejado correctamente en:

`Transacciones`

---

# Conciliación con Bancos

Después de importar:

validar por cuenta y periodo:

`Saldo calculado = saldo inicial + entradas - salidas`

y:

`Diferencia = saldo real - saldo calculado`

La diferencia esperada debe ser:

`≈ 0`

dentro de la tolerancia documentada.

Nunca insertar un movimiento ficticio para obtener cero.

---

# Orden de investigación de diferencias

Si Bancos no concilia, revisar:

1. movimiento faltante;
2. duplicado;
3. transferencia interna mal identificada;
4. cuenta incorrecta;
5. fecha incorrecta;
6. moneda/monto incorrectos;
7. comisión faltante;
8. reversión;
9. TC;
10. extracto parcial.

No empezar cambiando categorías o creando ajustes.

---

# Mes cerrado

Antes de modificar transacciones de un mes ya cerrado:

- identificar el impacto;
- revisar Bancos;
- revisar CxC/CxP;
- revisar Cash Flow;
- revisar Balance Sheet.

Actualmente agosto de 2026 es un mes cerrado y validado dentro del workflow.

No modificarlo silenciosamente.

---

# Validación posterior

Después de una importación revisar como mínimo:

`Transacciones`

`Bancos`

y, cuando corresponda:

- `CxC Mensual`
- `CxP Mensual`
- `CxP Sueldos`
- `Cash Flow`
- `Balance Sheet`
- `Runway Mensual`
- `Dashboard`

No esperar que `Operative incomes`, `Real S&A` o `Sueldos 2026` cambien únicamente porque entró un movimiento bancario.

---

# Estado de cierre

Una importación no se considera terminada solamente porque las filas fueron cargadas.

Debe verificarse:

- no duplicados;
- TC correcto;
- categorización válida o pendiente explícita;
- transferencias identificadas;
- saldo bancario conciliado;
- filas leídas nuevamente;
- ausencia de errores.

---

# Regla final

El extracto es evidencia de:

**cash**

No es automáticamente evidencia del:

**periodo contable de devengo**

Mantener siempre esa separación.
