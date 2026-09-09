# Finanzas Sommos — Transferencias internas

## Propósito

Definir cómo registrar y conciliar movimientos entre cuentas propias de Sommos sin tratarlos como ingresos, gastos, CxC o CxP.

La fuente operativa es:

`Transacciones`

La conciliación se valida en:

`Bancos`

---

# Clasificación

Una transferencia entre cuentas controladas por Sommos debe usar:

`Tipo = Transferencia interna`

y:

`Categoría = Transferencias internas`

Utilizar siempre los valores exactos del Sheet vivo.

---

# Impacto financiero

Una transferencia interna:

| Componente | Impacto |
|---|---|
| Banco origen | Disminuye |
| Banco destino | Aumenta |
| Cash consolidado | Sin cambio |
| Ingresos P&L | No |
| Gastos P&L | No |
| CxC | No |
| CxP | No |
| Burn operativo | No |

Pueden existir efectos separados por:

- comisión;
- diferencia cambiaria;
- spread.

Esos conceptos no deben esconderse dentro de la transferencia.

---

# Dirección

Toda transferencia debe poder interpretarse como:

`Cuenta origen → Cuenta destino`

Ejemplo:

`Meru → Banco Sol`

Para Bancos:

- Meru = salida
- Banco Sol = entrada

No inferir dirección sin suficiente evidencia.

---

# Campos relevantes

Revisar cuando existan:

- Fecha
- Banco/cuenta
- Tipo
- Categoría
- Descripción
- Moneda
- Monto original
- TC
- Monto USD
- Estado pago
- Conciliación
- Cuenta origen
- Cuenta destino
- Detalle transferencia
- Fecha pago/cobro

Leer los encabezados vivos antes de escribir.

---

# Una fila económica

Cuando la arquitectura vigente permite que una misma fila tenga:

- Cuenta origen;
- Cuenta destino;

preferir una sola transacción económica para representar el traslado.

La lógica de Bancos puede interpretar esa fila como:

- salida en origen;
- entrada en destino.

No duplicar automáticamente la transferencia porque aparezca en ambos extractos.

---

# Excepción: arquitectura bancaria

Si el modelo vivo requiere dos registros separados para representar ambos lados:

seguir la estructura vigente.

Antes de crear la segunda fila:

- comprobar que no duplique el cash consolidado;
- verificar referencias;
- revisar la lógica de `Bancos`.

No asumir una metodología sin leer el modelo actual.

---

# Transferencias entre monedas

Una transferencia puede salir en una moneda y llegar en otra.

Ejemplo:

`Meru USD → Banco Sol BOB`

No exigir igualdad nominal.

Validar:

1. monto debitado;
2. monto acreditado;
3. TC;
4. comisión;
5. spread/diferencia;
6. extractos de ambos lados.

La naturaleza principal sigue siendo transferencia interna.

---

# Comisiones

Si existe una comisión:

registrarla separadamente cuando esté demostrada.

No incluirla dentro de:

`Transferencias internas`

solo para conseguir que ambas cuentas cuadren.

Categoría habitual:

`Bank fees`

cuando corresponda.

---

# Diferencias cambiarias

Si existe una diferencia económica real por conversión:

puede corresponder a:

`Exchange rate differences`

según la lógica contable vigente.

No crear una diferencia cambiaria únicamente porque origen y destino tengan monedas distintas.

---

# Intermediarios

Una transferencia puede aparecer bajo el nombre de:

- RemotePay;
- procesador;
- banco corresponsal;
- billetera;
- intermediario.

El nombre mostrado por el banco no determina la naturaleza económica.

Si el flujo real es:

`Cuenta propia → intermediario → cuenta propia`

sigue siendo una transferencia interna.

---

# Estado de pago

Una transferencia realizada debe tener el estado correspondiente a cash realizado según `Config`.

Una transferencia pendiente:

no debe afectar Bancos como movimiento realizado.

No marcar como realizada sin evidencia.

---

# Deduplicación

Antes de registrar revisar:

- fecha;
- monto;
- moneda;
- cuenta origen;
- cuenta destino;
- referencia;
- descripción;
- contraparte.

La aparición del movimiento en ambos extractos no significa necesariamente que existan dos movimientos económicos.

---

# Conciliación

Después de registrar una transferencia:

comprobar:

- banco origen;
- banco destino;
- dirección;
- estado;
- monto;
- TC;
- posibles comisiones;
- posible diferencia cambiaria;
- duplicados.

A nivel consolidado:

`efecto de transferencia ≈ 0`

salvo costos asociados registrados por separado.

---

# Casos conocidos

## Meru → Banco Sol

Puede existir:

- salida USD en Meru;
- entrada BOB en Banco Sol;
- intermediario en la descripción.

No reconocer la entrada en Banco Sol como revenue.

---

# Brex

Movimientos ACH desde Brex pueden ser:

- transferencia entre cuentas propias;
- pago a proveedor;
- otro movimiento.

No clasificar todos los ACH como transferencia interna.

Determinar primero quién recibe económicamente el dinero.

---

# Guardrails

No usar Transferencias internas para:

- ocultar un gasto;
- ocultar un ingreso;
- eliminar diferencias bancarias;
- compensar una comisión;
- cuadrar Cash Flow.

La clasificación debe representar el movimiento real.

---

# QA

Después de modificar una transferencia revisar:

- `Transacciones`
- `Bancos`
- Cash consolidado
- `Cash Flow`
- `Balance Sheet`

Confirmar que no afectó:

- revenue;
- gastos;
- CxC;
- CxP.

---

# Regla final

Una transferencia interna cambia:

**dónde está la caja**

pero no cambia:

**cuánta caja consolidada tiene Sommos**

salvo costos reales asociados que deben registrarse separadamente.
