# Transferencias internas

## Propósito

Documentar cómo registrar, identificar y conciliar movimientos de dinero entre cuentas propias de Sommos sin tratarlos como ingresos o gastos operativos.

La fuente de verdad es la pestaña `Transacciones` del archivo financiero vigente.

---

## Tipo correcto

Las transferencias entre cuentas propias deben registrarse con:

`Tipo = Transferencia interna`

No usar variantes como:
- Transferencia
- Internal transfer
- Movimiento interno

La lógica financiera y bancaria debe reconocer exactamente `Transferencia interna`.

---

## Categoría

La categoría estándar es:

`Transferencias internas`

Una transferencia interna:

- no es ingreso operativo;
- no es gasto operativo;
- no incrementa revenue;
- no incrementa burn;
- no debe afectar el P&L;
- sí afecta los saldos de las cuentas bancarias involucradas.

---

## Campos relevantes en Transacciones

Para una transferencia interna deben revisarse especialmente:

- `Fecha`
- `Tipo`
- `País`
- `Categoría`
- `Descripción`
- `Moneda`
- `Monto original`
- `TC a USD`
- `Monto USD`
- `Cuenta / medio`
- `Conciliación`
- `Estado pago`
- `Cuenta origen`
- `Cuenta destino`
- `Detalle transferencia`

La estructura real de columnas debe leerse siempre en vivo antes de escribir.

---

## Cuenta origen y cuenta destino

Toda transferencia interna debe indicar su dirección.

Ejemplo:

`Meru → Banco Sol`

Debe registrarse como:

- Cuenta origen: `Meru`
- Cuenta destino: `Banco Sol`

Para la conciliación:

- en `Meru` representa una salida;
- en `Banco Sol` representa una entrada.

Nunca inferir la dirección únicamente por el signo del monto si el extracto o la descripción no son suficientemente claros.

---

## Regla de conciliación bancaria

Una transferencia interna puede mover efectivo entre bancos sin modificar el cash consolidado total de Sommos.

Conceptualmente:

`Salida cuenta origen = - transferencia`

`Entrada cuenta destino = + transferencia`

A nivel consolidado:

`Efecto neto esperado ≈ 0`

Esto puede no ser exactamente cero cuando existen:

- comisiones;
- diferencias de cambio;
- spreads;
- cargos del intermediario.

Esos conceptos deben registrarse por separado.

---

## Transferencias entre monedas

Una transferencia puede salir en una moneda y llegar en otra.

Ejemplo conocido:

`Meru USD → Banco Sol BOB`

En estos casos no exigir que el monto nominal de origen sea igual al monto nominal de destino.

Validar:

1. monto debitado en cuenta origen;
2. monto acreditado en cuenta destino;
3. tipo de cambio aplicado;
4. comisión o diferencia existente;
5. extractos de ambas cuentas.

El movimiento interno sigue siendo una transferencia, mientras que una comisión debe ser un gasto independiente.

---

## Comisiones

Nunca incluir una comisión dentro de `Transferencias internas` solo para hacer cuadrar ambos bancos.

Ejemplo conceptual:

- salen USD 200 de Meru;
- Banco Sol recibe el equivalente a USD 198;
- USD 2 corresponden a comisión.

Registrar:

### Transferencia

`Tipo = Transferencia interna`

por el movimiento principal.

### Comisión

Registrar un movimiento separado como gasto, usando la categoría correspondiente, por ejemplo:

`Bank fees`

o la categoría vigente definida en `Config`.

No inventar una comisión si el extracto no la demuestra.

---

## Intermediarios

Una transferencia entre cuentas propias puede aparecer en el banco bajo el nombre de un intermediario.

Ejemplo conocido:

`RemotePay Solutions`

El nombre del intermediario no convierte automáticamente el movimiento en ingreso de cliente o gasto.

Debe identificarse la realidad económica del movimiento:

`cuenta propia → intermediario → cuenta propia`

Si ese es el caso, sigue siendo una transferencia interna.

---

## Una fila o dos filas

La prioridad es evitar duplicar flujo financiero.

### Cuando existe una sola fila

Puede mantenerse una sola transacción con:

- Cuenta origen
- Cuenta destino
- monto
- detalle de transferencia

La lógica de `Bancos` debe interpretar esa misma fila como:

- salida para el origen;
- entrada para el destino.

Esta es la estructura preferida cuando el modelo ya soporta transferencias de doble efecto.

### Cuando los extractos muestran movimientos distintos

No crear dos transacciones económicas si ambas representan el mismo traslado de fondos, salvo que la arquitectura del Sheet lo requiera expresamente.

Antes de añadir una segunda fila, buscar duplicados y revisar la lógica de conciliación.

---

## Estado de pago

Una transferencia que realmente ocurrió debe estar en estado realizado según las opciones vigentes del Sheet.

Los movimientos pendientes no deben afectar los saldos bancarios.

Antes de registrar una transferencia como realizada, debe existir evidencia en el extracto o confirmación equivalente.

---

## Reglas para importar extractos

Cuando se carga un extracto:

1. buscar primero si el movimiento ya existe;
2. identificar si corresponde a dinero entre cuentas propias;
3. revisar descripción, contraparte y banco;
4. determinar cuenta origen;
5. determinar cuenta destino;
6. asignar `Tipo = Transferencia interna`;
7. asignar `Categoría = Transferencias internas`;
8. separar cualquier comisión;
9. conciliar ambas cuentas;
10. verificar que el movimiento no haya afectado ingresos o burn.

---

## Detección de duplicados

Antes de registrar una transferencia revisar coincidencias por:

- fecha;
- monto;
- moneda;
- cuenta origen;
- cuenta destino;
- descripción;
- contraparte;
- referencia del extracto.

Una transferencia puede aparecer en los extractos de ambas cuentas.

Eso no significa automáticamente que deban existir dos registros económicos en `Transacciones`.

---

## Diferencias de conciliación

Si un banco no concilia después de registrar una transferencia, revisar:

1. si `Tipo` es exactamente `Transferencia interna`;
2. si existe `Cuenta origen`;
3. si existe `Cuenta destino`;
4. si la dirección está invertida;
5. si existe una comisión;
6. si existe diferencia cambiaria;
7. si falta la contraparte de la transferencia;
8. si el movimiento está duplicado;
9. si la fecha corresponde al extracto;
10. si el movimiento está marcado como realizado.

Nunca inventar movimientos para llevar la diferencia a cero.

---

## Impacto financiero

| Componente | Transferencia interna |
|---|---|
| Banco origen | Disminuye |
| Banco destino | Aumenta |
| Cash consolidado | Sin cambio, salvo comisión/FX |
| Ingresos | No |
| Gastos | No |
| Burn | No |
| CxC | No |
| CxP | No |
| P&L | No |
| Flujo entre cuentas | Sí |

---

## Ejemplos operativos conocidos

### Meru → Banco Sol

Puede existir una salida en USD desde Meru y una entrada en BOB a Banco Sol mediante un intermediario.

Registrar la transferencia según su realidad económica y separar cualquier comisión.

### Movimientos ACH desde Brex

Si un ACH representa dinero enviado desde Brex hacia otra cuenta propia, debe incluir correctamente `Cuenta origen` y `Cuenta destino`.

Si el receptor no es una cuenta propia, no asumir que es transferencia interna: revisar la naturaleza del pago antes de categorizar.

---

## Regla de seguridad

No cambiar una categoría únicamente para conseguir que el banco concilie.

Primero determinar qué ocurrió económicamente.

La conciliación bancaria debe ser consecuencia de registros correctos, no el objetivo de una reclasificación artificial.

---

## Fuente viva

Los ejemplos de este archivo sirven como referencia.

Si existe contradicción entre este documento y:

`Finanzas Sommos — Workflow y Control`

prevalecen siempre:

1. los extractos bancarios;
2. la estructura vigente de `Transacciones`;
3. las reglas activas en `Config` y `Reglas categorización`;
4. la información actual del Google Sheet.
